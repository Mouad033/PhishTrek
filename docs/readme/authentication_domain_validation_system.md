# PhishTrek - Authentication & Domain Validation System

## Complete Authentication Flow with Email Verification & 2FA

```mermaid
sequenceDiagram
    actor User
    participant Web as Frontend<br/>(React)
    participant API as Backend<br/>(FastAPI)
    participant DB as PostgreSQL
    participant Email as Email Service<br/>(SendGrid/SMTP)
    participant BlockList as Blocklist<br/>(Local Cache)
    participant Clearbit as Clearbit API<br/>(Company enrichment)
    participant WHOIS as WHOIS API<br/>(Domain info)
    participant OpenCorp as OpenCorporates<br/>(Company registry)
    participant DNS as DNS Resolver<br/>(MX/SPF/DKIM)
    participant SSL as SSL Checker<br/>(TLS cert)
    participant Queue as Job Queue<br/>(Background)

    Note over User,SSL: 📋 PHASE 1: SIGNUP INITIAL & BLOCKLIST CHECK

    User->>Web: 1. Click "Sign Up"
    Web->>User: Display signup form
    User->>Web: 2. Enter email@acme.com + password + company_name
    Web->>API: POST /auth/signup {email, password, company_name}

    Note over API,BlockList: ⚡ STEP 1: IMMEDIATE FILTERING (< 100ms)

    API->>API: Extract domain from email<br/>domain = "acme.com"
    API->>BlockList: Is acme.com in PUBLIC_PROVIDERS list?<br/>(gmail.com, outlook.com, yahoo.com, etc.)
    BlockList-->>API: ❌ NO match

    API->>BlockList: Is acme.com in DISPOSABLE list?<br/>(temp-mail.com, 10minutemail.com, etc.)
    BlockList-->>API: ❌ NO match

    alt ❌ DOMAIN BLOCKED
        API-->>Web: {"error": "Domain not allowed. Use company email."}
        Web-->>User: ❌ "Gmail/Outlook not permitted. Use your company email."
    else ✅ DOMAIN ALLOWED
        Note over API,SSL: ✅ Domain passed initial checks

        Note over API,Email: 📧 STEP 2: EMAIL VERIFICATION TOKEN

        API->>API: Generate verification_token = UUID<br/>Generate confirmation_code = "524891"
        API->>DB: INSERT VerificationToken {email, token, code, expires_at=NOW+24h, domain}
        DB-->>API: ✅ Token stored

        API->>Email: SEND VERIFICATION EMAIL<br/>To: user@acme.com<br/>Subject: "Verify your PhishTrek account"<br/>Body: "Code: 524891" + link
        Email-->>API: ✅ Email queued

        API->>DB: INSERT User (UNVERIFIED) {email, hashed_password, company_name, domain, is_verified=false, status=pending_email_verification}
        DB-->>API: ✅ User created (unverified)

        API-->>Web: {status: "pending_verification", message: "Check your email for code", expires_in: "24h"}
        Web-->>User: ✅ "Check your email for verification code"

        Note over User,SSL: 📧 USER RECEIVES EMAIL & VERIFIES

        User->>Email: Open inbox
        Email-->>User: "Verify PhishTrek account - Code: 524891"

        par Option A: Click Link
            User->>Web: Click verification link
            Web->>API: GET /auth/verify?token=abc123xyz
        or Option B: Enter Code
            User->>Web: Enter code: 524891
            Web->>API: POST /auth/verify-code {code: "524891"}
        end

        Note over API,BlockList: 🔗 STEP 3: EMAIL VERIFICATION VALIDATION

        API->>DB: Validate token/code: Token exists? Not expired? Hash matches?
        DB-->>API: ✅ VALID

        API->>DB: UPDATE User {is_verified=true, verified_at=NOW(), status=pending_domain_validation}
        DB-->>API: ✅ Updated

        API-->>Web: {status: "email_verified", message: "Email verified! We're validating your domain..."}
        Web-->>User: ✅ "Email verified! Account created (validation in progress)"

        Note over API,SSL: 🔄 PHASE 2: ASYNCHRONOUS DOMAIN ENRICHMENT (BACKGROUND)

        Note over API,Queue: ⏱️ NON-BLOCKING: Jobs queued in background<br/>User can already login during validation

        API->>Queue: ENQUEUE job_validate_domain {user_id, domain: "acme.com", company_name: "Acme Corp"}
        Queue-->>API: ✅ Job queued

        par CHECK 1: Clearbit Company Enrichment

            Queue->>Clearbit: GET /api/v1/companies/find?domain=acme.com
            Clearbit-->>Queue: {name: "Acme Corporation", domain: "acme.com", employees: 1250, founded: 2010}
            Queue->>Queue: Clearbit match found<br/>score += 40

        and CHECK 2: WHOIS Domain Info

            Queue->>WHOIS: WHOIS lookup domain=acme.com
            WHOIS-->>Queue: {registrant_name: "Acme Corp", creation_date: "2010-03-15", privacy_enabled: false}
            Queue->>Queue: Domain age: 14 years (✅ +10)<br/>Privacy disabled (✅ +5)<br/>score += 15

        and CHECK 3: OpenCorporates Registry

            Queue->>OpenCorp: Search company="Acme Corp" jurisdiction="US"
            OpenCorp-->>Queue: {company: "Acme Corporation", incorporation_date: "2010", status: "active"}
            Queue->>Queue: OpenCorporates match found<br/>score += 25

        and CHECK 4: DNS Records

            Queue->>DNS: MX lookup acme.com
            DNS-->>Queue: {mx: ["mail.acme.com"], priority: 10}
            Queue->>Queue: MX exists (✅ +15)

            Queue->>DNS: SPF lookup acme.com
            DNS-->>Queue: "v=spf1 include:_spf.google.com ~all"
            Queue->>Queue: SPF found (✅ +10)

            Queue->>DNS: DMARC lookup acme.com
            DNS-->>Queue: "v=DMARC1; p=reject"
            Queue->>Queue: DMARC found (✅ +10)<br/>score += 35

        and CHECK 5: TLS/SSL Certificate

            Queue->>SSL: GET https://acme.com<br/>Check TLS cert
            SSL-->>Queue: {valid: true, issuer: "Let's Encrypt", org: "Acme Corporation", expiry: "2026-03-15"}
            Queue->>Queue: TLS cert valid (✅ +10)<br/>Org matches (✅ +5)<br/>score += 15

        end

        Note over API,Queue: 📊 PHASE 3: FINAL SCORING & DECISION

        Queue->>Queue: FINAL SCORE CALCULATION:<br/>Clearbit: +40 | WHOIS: +15 | OpenCorporates: +25<br/>DNS: +35 | TLS: +15 | TOTAL: 130 points

        alt SCORE >= 100: TRUSTED PROFESSIONAL DOMAIN

            Queue->>DB: UPDATE User {status: "active", domain_status: "trusted", domain_score: 130}
            DB-->>Queue: ✅ Updated
            Queue->>Email: SEND SUCCESS EMAIL<br/>Subject: "Welcome to PhishTrek!"
            Email-->>Queue: ✅ Sent

        else SCORE 60-99: CONDITIONAL APPROVAL

            Queue->>DB: UPDATE User {status: "active_conditional", domain_status: "verified_partial", limitations: ["no_bulk_campaigns"]}
            DB-->>Queue: ✅ Updated
            Queue->>Email: SEND CONDITIONAL EMAIL<br/>Subject: "PhishTrek - Some features restricted"
            Email-->>Queue: ✅ Sent

        else SCORE < 60: MANUAL REVIEW NEEDED

            Queue->>DB: UPDATE User {status: "pending_manual_review", domain_status: "requires_validation"}
            DB-->>Queue: ✅ Updated
            Queue->>Email: SEND MANUAL REVIEW EMAIL<br/>Subject: "PhishTrek - Please verify your domain"
            Email-->>Queue: ✅ Sent
            Note over Queue: Manual verification options:<br/>1) Upload company cert<br/>2) Add SSO admin<br/>3) Add DNS TXT record

        end

        Note over User,SSL: 🔐 PHASE 4: LOGIN & 2FA EMAIL

        User->>Web: Click "Login"
        Web->>User: Display login form
        User->>Web: Enter email + password
        Web->>API: POST /auth/login {email, password}

        API->>DB: SELECT User WHERE email = "user@acme.com"
        DB-->>API: User found

        API->>API: Verify password hash

        alt PASSWORD CORRECT

            API->>API: Generate OTP = "738291"<br/>Hash OTP + store with 10min expiry
            API->>DB: INSERT LoginOTP {user_id, otp_hash, session_id, expires_at=NOW+10min}
            DB-->>API: ✅ Stored

            API->>Email: SEND OTP EMAIL<br/>Subject: "Your PhishTrek login code"<br/>Body: "Code: 738291 (expires in 10 min)"
            Email-->>API: ✅ Sent

            API-->>Web: {status: "otp_sent", session_id: "session_abc123", message: "Enter code sent to email"}
            Web-->>User: 📧 "Enter OTP sent to user@acme.com"

            User->>Email: Open inbox
            Email-->>User: "Your PhishTrek login code: 738291"
            User->>Web: Enter OTP: 738291

            Web->>API: POST /auth/verify-otp {session_id, otp}

            API->>DB: SELECT LoginOTP WHERE session_id AND verified=false
            DB-->>API: OTP record found

            alt OTP NOT EXPIRED

                API->>API: Verify OTP: hash("738291") == stored_otp_hash?

                alt OTP HASH MATCHES

                    API->>DB: UPDATE LoginOTP {verified=true}
                    DB-->>API: ✅ OTP marked verified

                    API->>API: Create JWT token {user_id, email, domain_status, exp=NOW+7days}
                    API-->>Web: {access_token: "eyJhbGc...", token_type: "bearer", user_status: "active"}
                    Web-->>User: ✅ LOGGED IN SUCCESSFULLY<br/>Redirect to /dashboard

                else OTP HASH DOES NOT MATCH
                    API-->>Web: {error: "Invalid OTP"}
                    Web-->>User: ❌ "Invalid code. Please try again."
                end

            else OTP EXPIRED
                API-->>Web: {error: "OTP expired. Please request new code."}
                Web-->>User: ❌ "Code expired. Please login again."
            end

        else PASSWORD INCORRECT
            API-->>Web: {error: "Invalid credentials"}
            Web-->>User: ❌ "Invalid email or password"
        end

        Note over User,SSL: ✅ AUTHENTICATION COMPLETE
```