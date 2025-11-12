# PhishTrek - GoPhish Deployment Architecture

## Offensive Campaign Generation

```mermaid
sequenceDiagram
    actor RedTeam as Red Team Lead<br/>(Authenticated User)
    participant Web as Frontend<br/>(React UI)
    participant API as Backend API<br/>(FastAPI)
    participant DB as PostgreSQL<br/>(Campaign Metadata)
    participant Queue as Job Queue<br/>(Celery/Redis)
    participant Cache as Cache Layer<br/>(Redis)
    participant GoPhish as GoPhish API<br/>(Admin Port 3333)
    participant SMTPRelay as SMTP Relay<br/>(SendGrid/Mailgun)
    participant EmailServers as Target Email<br/>Servers

    Note over RedTeam,EmailServers: 🎯 PHASE 2: OFFENSIVE CAMPAIGN GENERATION (GoPhish)

    Note over RedTeam,EmailServers: ⏱️ STEP 1: CAMPAIGN CONFIGURATION

    RedTeam->>Web: 1. Click "Create Campaign"
    Web->>RedTeam: Display campaign creation form

    RedTeam->>Web: 2. Fill campaign details:<br/>├─ Campaign name: "Acme HR Q4 Test"<br/>├─ Target persona: "Finance Manager"<br/>├─ Target domain: "acme.com"<br/>├─ Recipients: [50 email addresses]<br/>├─ LLM model: "GPT-4"<br/>├─ Email variants: 30<br/>├─ Attack type: "Credentials"<br/>└─ Launch time: "2025-11-20 14:00:00 UTC"

    Web->>API: POST /campaigns/initiate {<br/>  name: "Acme HR Q4 Test",<br/>  persona: "Finance Manager",<br/>  domain: "acme.com",<br/>  recipients: [...50 emails...],<br/>  llm_model: "GPT-4",<br/>  variant_count: 30,<br/>  attack_type: "credentials",<br/>  launch_datetime: "2025-11-20T14:00:00Z"<br/>}

    API->>DB: INSERT campaigns {<br/>  user_id,<br/>  name: "Acme HR Q4 Test",<br/>  status: "pending_gophish_setup",<br/>  created_at: NOW(),<br/>  gophish_campaign_ids: NULL<br/>}
    DB-->>API: campaign_id = 1001

    API-->>Web: {status: "setup_pending", campaign_id: 1001}
    Web-->>RedTeam: ✅ "Campaign 1001 created. Setting up GoPhish resources..."

    Note over RedTeam,EmailServers: ⏱️ STEP 2: CREATE GOPHISH GROUP (Recipients)

    Note over API,GoPhish: Create recipient list in GoPhish

    API->>GoPhish: POST /api/groups {<br/>  name: "Acme_Finance_Managers_Q4",<br/>  targets: [<br/>    {<br/>      email: "john.smith@acme.com",<br/>      first_name: "John",<br/>      last_name: "Smith",<br/>      position: "Finance Manager"<br/>    },<br/>    {<br/>      email: "jane.doe@acme.com",<br/>      first_name: "Jane",<br/>      last_name: "Doe",<br/>      position: "Financial Analyst"<br/>    },<br/>    ... (48 more targets)<br/>  ]<br/>}

    alt ✅ GROUP CREATED (201 Created)
        GoPhish-->>API: {<br/>  id: 101,<br/>  name: "Acme_Finance_Managers_Q4",<br/>  target_count: 50,<br/>  created_date: "2025-11-20T13:55:00Z"<br/>}

        API->>DB: INSERT gophish_groups {<br/>  campaign_id: 1001,<br/>  gophish_group_id: 101,<br/>  name: "Acme_Finance_Managers_Q4",<br/>  target_count: 50,<br/>  created_at: NOW()<br/>}
        DB-->>API: ✅ Stored

        Note over API: Group creation successful. Proceed to templates.

    else ❌ ERROR (400 Bad Request)
        GoPhish-->>API: {error: "Invalid email format", targets: [invalid_emails]}

        API-->>Web: {status: "error", message: "Invalid email in target list. Review recipients."}
        Web-->>RedTeam: ❌ "Error creating group. Check recipient email addresses."
        Note over RedTeam: User must fix invalid emails and retry

    else ⚠️ DUPLICATE GROUP
        GoPhish-->>API: {error: "Group already exists"}

        API->>API: Retrieve existing group ID
        API->>DB: UPDATE campaigns SET gophish_group_id = existing_id
        Note over API: Use existing group instead

    end

    Note over RedTeam,EmailServers: ⏱️ STEP 3: CREATE EMAIL TEMPLATES (30 variants)

    Note over API,GoPhish: Create 30 email template variations

    par CREATE TEMPLATES IN PARALLEL
        loop For each email variant (1-30)
            API->>GoPhish: POST /api/templates {<br/>  name: "Variant_001_FinanceAudit",<br/>  subject: "Urgent: Q3 Financial Audit Required",<br/>  text: "Dear {{.FirstName}},\n\nPlease verify your financial audit access...",<br/>  html: "<html><body><h2>Financial Audit Notice</h2>\n<p>Dear {{.FirstName}} {{.LastName}},</p>\n<p><a href='{{.URL}}'>Click here to verify</a></p>\n</body></html>",<br/>  attachments: []<br/>}

            alt ✅ TEMPLATE CREATED (201)
                GoPhish-->>API: {<br/>  id: 201,<br/>  name: "Variant_001_FinanceAudit",<br/>  subject: "Urgent: Q3 Financial Audit Required",<br/>  created_date: "2025-11-20T13:56:00Z"<br/>}

                API->>DB: INSERT gophish_templates {<br/>  campaign_id: 1001,<br/>  gophish_template_id: 201,<br/>  variant_number: 1,<br/>  subject: "Urgent: Q3 Financial Audit Required",<br/>  created_at: NOW()<br/>}
                DB-->>API: ✅ Stored

            else ❌ ERROR
                GoPhish-->>API: {error: "Template too large (> 2MB)"}
                API->>API: Log error and skip variant
                API-->>Web: Event: "Template variant 15 exceeded size limit. Skipped."

            end
        end
    end

    API->>DB: INSERT campaign_stats {<br/>  campaign_id: 1001,<br/>  templates_created: 30,<br/>  templates_created_at: NOW()<br/>}
    DB-->>API: ✅ All 30 templates logged

    API-->>Web: Event: "✅ 30 email templates created successfully"
    Web-->>RedTeam: ✅ "Email variants ready (30/30)"

    Note over RedTeam,EmailServers: ⏱️ STEP 4: CREATE LANDING PAGE

    Note over API,GoPhish: Clone target website or create custom form

    API->>API: Generate landing page HTML<br/>Options:<br/>1. Custom form (phishing form)<br/>2. Cloned target website<br/>3. Hybrid (real content + form overlay)

    API->>API: Decision: Create custom form<br/>html = """<html><head><title>Acme Financial Portal</title></head><br/><body><br/><form method='POST'><br/>  <input type='email' name='email' placeholder='Email' required><br/>  <input type='password' name='password' placeholder='Password' required><br/>  <input type='submit' value='Login'><br/></form><br/></body></html>"""

    API->>GoPhish: POST /api/landing-pages {<br/>  name: "Acme_Finance_Portal_Clone",<br/>  html: "...",<br/>  capture_credentials: true,<br/>  capture_passwords: true<br/>}

    alt ✅ LANDING PAGE CREATED (201)
        GoPhish-->>API: {<br/>  id: 301,<br/>  name: "Acme_Finance_Portal_Clone",<br/>  created_date: "2025-11-20T13:57:00Z"<br/>}

        API->>DB: INSERT gophish_landing_pages {<br/>  campaign_id: 1001,<br/>  gophish_page_id: 301,<br/>  name: "Acme_Finance_Portal_Clone",<br/>  capture_credentials: true,<br/>  capture_passwords: true,<br/>  created_at: NOW()<br/>}
        DB-->>API: ✅ Stored

    else ❌ ERROR (Invalid HTML)
        GoPhish-->>API: {error: "Invalid HTML structure"}

        API->>API: Validate and fix HTML
        API->>GoPhish: POST /api/landing-pages {corrected_html}
        GoPhish-->>API: {id: 301}

    end

    API-->>Web: Event: "✅ Landing page created"
    Web-->>RedTeam: ✅ "Landing page configured (Acme Finance Portal)"

    Note over RedTeam,EmailServers: ⏱️ STEP 5: CREATE SENDING PROFILE (SMTP)

    Note over API,GoPhish: Configure SMTP relay for email sending

    API->>API: Retrieve SMTP credentials<br/>From secrets manager:<br/>- Provider: SendGrid<br/>- Host: smtp.sendgrid.net:587<br/>- Username: apikey<br/>- Password: SG.xxxxxxxxxxxxxx (encrypted)

    API->>GoPhish: POST /api/sending-profiles {<br/>  name: "SendGrid_Acme_Q4",<br/>  from: "noreply@acme-verify.example.com",<br/>  host: "smtp.sendgrid.net:587",<br/>  username: "apikey",<br/>  password: "SG.xxxxxxxxxxxxxx",<br/>  ignore_cert_errors: false<br/>}

    alt ✅ SENDING PROFILE CREATED (201)
        GoPhish-->>API: {<br/>  id: 401,<br/>  name: "SendGrid_Acme_Q4",<br/>  from: "noreply@acme-verify.example.com",<br/>  created_date: "2025-11-20T13:58:00Z"<br/>}

        API->>DB: INSERT gophish_sending_profiles {<br/>  campaign_id: 1001,<br/>  gophish_profile_id: 401,<br/>  name: "SendGrid_Acme_Q4",<br/>  from_address: "noreply@acme-verify.example.com",<br/>  smtp_host: "smtp.sendgrid.net:587",<br/>  created_at: NOW()<br/>}
        DB-->>API: ✅ Stored

        Note over API: SMTP profile ready. Ready to launch campaigns.

    else ❌ SMTP CONNECTION ERROR (400)
        GoPhish-->>API: {error: "SMTP connection failed. Check credentials."}

        API->>API: Log error and alert admin
        API-->>Web: {status: "error", message: "SMTP connection failed. Check sending profile configuration."}
        Web-->>RedTeam: ❌ "SMTP configuration error. Contact administrator."
        Note over RedTeam: Cannot proceed until SMTP is fixed

    end

    API-->>Web: Event: "✅ Sending profile configured"
    Web-->>RedTeam: ✅ "SMTP relay configured (SendGrid)"

    Note over RedTeam,EmailServers: ⏱️ STEP 6: CREATE & LAUNCH CAMPAIGNS (30 variant campaigns)

    Note over API,GoPhish: Launch 30 separate GoPhish campaigns (one per template variant)

    par LAUNCH 30 CAMPAIGNS (Parallel Execution)
        loop For each variant (1-30)
            API->>GoPhish: POST /api/campaigns {<br/>  name: "Acme_Finance_Variant_001",<br/>  template: {<br/>    id: 201  <br/>  },<br/>  landing_page: {<br/>    id: 301<br/>  },<br/>  sending_profile: {<br/>    id: 401<br/>  },<br/>  groups: [<br/>    {id: 101}<br/>  ],<br/>  url: "https://acme-verify-login.example.com",<br/>  launch_date: "2025-11-20T14:00:00Z"<br/>}

            alt ✅ CAMPAIGN CREATED & LAUNCHED (201)
                GoPhish-->>API: {<br/>  id: 5001,<br/>  name: "Acme_Finance_Variant_001",<br/>  created_date: "2025-11-20T13:59:00Z",<br/>  launch_date: "2025-11-20T14:00:00Z",<br/>  status: "In Progress",<br/>  stats: {<br/>    send_count: 0,<br/>    opened_count: 0,<br/>    clicked_count: 0<br/>  }<br/>}

                API->>DB: INSERT gophish_campaigns {<br/>  campaign_id: 1001,<br/>  gophish_campaign_id: 5001,<br/>  variant_number: 1,<br/>  name: "Acme_Finance_Variant_001",<br/>  status: "in_progress",<br/>  launched_at: NOW()<br/>}
                DB-->>API: ✅ Stored

                Note over API: Campaign 5001 launched. SMTP will send emails at scheduled time.

            else ❌ CAMPAIGN CREATION ERROR (400)
                GoPhish-->>API: {error: "Invalid campaign parameters"}

                API->>API: Log error, retry with corrected parameters
                API->>GoPhish: POST /api/campaigns {corrected_params}
                GoPhish-->>API: {id: 5001}

            else ⚠️ DUPLICATE CAMPAIGN NAME
                GoPhish-->>API: {error: "Campaign name already exists"}

                API->>API: Append timestamp to name: "Acme_Finance_Variant_001_ts1234"
                API->>GoPhish: POST /api/campaigns {updated_name}
                GoPhish-->>API: {id: 5001}

            end
        end
    end

    API->>DB: BULK INSERT gophish_campaigns {<br/>  campaign_ids: [5001-5030],<br/>  count: 30,<br/>  all_launched_at: NOW()<br/>}
    DB-->>API: ✅ All 30 campaigns logged

    API->>Cache: SET campaign:1001:gophish_ids {<br/>  gophish_ids: [5001, 5002, ..., 5030],<br/>  launched_at: NOW(),<br/>  total_variants: 30,<br/>  ttl: 86400 (24 hours)<br/>}
    Cache-->>API: ✅ Cached for fast polling

    API-->>Web: {<br/>  status: "launched",<br/>  campaign_id: 1001,<br/>  gophish_campaign_ids: [5001-5030],<br/>  variants_launched: 30,<br/>  recipients_total: 50,<br/>  emails_queued: 1500,<br/>  message: "Campaign launched! Emails sending now..."<br/>}

    Web-->>RedTeam: ✅ CAMPAIGN LAUNCHED SUCCESSFULLY<br/><br/>📊 Summary:<br/>├─ Campaign ID: 1001<br/>├─ Variants: 30 email templates<br/>├─ Recipients: 50 finance managers<br/>├─ Total emails: 1500 (30 × 50)<br/>├─ Status: IN PROGRESS<br/>├─ Launch time: 2025-11-20T14:00:00Z<br/>└─ Monitoring: Real-time results<br/><br/>[View Results] [Pause Campaign] [Block Campaign]

    Note over RedTeam,EmailServers: 📧 STEP 7: GOPHISH SENDS EMAILS (SMTP Execution)

    Note over GoPhish,EmailServers: GoPhish executes at launch_date and sends emails via SMTP

    GoPhish->>GoPhish: Poll database for scheduled campaigns<br/>SELECT * FROM campaigns WHERE launch_date <= NOW()

    GoPhish->>GoPhish: For each campaign variant (5001-5030):<br/>1. Retrieve template (ID: 201-230)<br/>2. Retrieve group targets (ID: 101 - 50 recipients)<br/>3. For each recipient in group:<br/>   - Personalize email ({{.FirstName}}, {{.LastName}})<br/>   - Add unique tracking pixel<br/>   - Add unique URL with tracking code<br/>   - Generate message-id<br/>   - Queue email for sending

    loop Send emails in batches (50 at a time per variant)
        GoPhish->>SMTPRelay: SMTP: MAIL FROM: noreply@acme-verify.example.com<br/>RCPT TO: john.smith@acme.com<br/>DATA: (email with headers + body + tracking pixel)

        alt ✅ EMAIL ACCEPTED (250 OK)
            SMTPRelay-->>GoPhish: 250 OK - Message queued

            GoPhish->>GoPhish: Record event: {<br/>  campaign_id: 5001,<br/>  email: john.smith@acme.com,<br/>  event_type: "sent",<br/>  timestamp: NOW(),<br/>  message_id: "<abc123@acme-verify.example.com>"<br/>}

        else ⚠️ RECIPIENT BOUNCE (550 User unknown)
            SMTPRelay-->>GoPhish: 550 User not found

            GoPhish->>GoPhish: Record event: {<br/>  campaign_id: 5001,<br/>  email: invalid@acme.com,<br/>  event_type: "bounce",<br/>  bounce_type: "permanent",<br/>  bounce_reason: "User unknown",<br/>  timestamp: NOW()<br/>}

        else ⚠️ RATE LIMITED BY RELAY (421 Service unavailable)
            SMTPRelay-->>GoPhish: 421 Too many connections

            GoPhish->>GoPhish: Backoff and retry later<br/>Queue email for retry in 5 minutes

        end
    end

    Note over GoPhish,EmailServers: All 1500 emails sent (or queued for retry)

    par EmailServers receive emails
        EmailServers->>EmailServers: Email arrives in recipient mailbox<br/>├─ From: noreply@acme-verify.example.com<br/>├─ Subject: "Urgent: Q3 Financial Audit Required"<br/>├─ Body: HTML with landing page link<br/>├─ Tracking pixel: <img src='https://acme-verify.../open?r=abc123' /><br/>└─ CTA Button: <a href='https://acme-verify.../click?r=abc123'>Click here</a>

    end

    Note over RedTeam,EmailServers: ⏱️ STEP 8: POLL GOPHISH RESULTS (Real-time Monitoring)

    Note over API,GoPhish: Begin real-time result polling

    API->>Queue: ENQUEUE campaign_polling_job {<br/>  campaign_id: 1001,<br/>  gophish_campaign_ids: [5001-5030],<br/>  poll_interval: 10,<br/>  max_duration: 7200 (2 hours)<br/>}
    Queue-->>API: ✅ Job queued

    loop Every 10 seconds (for 2 hours or until campaign ends)
        Queue->>GoPhish: GET /api/campaigns/5001/results<br/>(batch query all 30 campaigns)

        alt ✅ RESULTS AVAILABLE (200 OK)
            GoPhish-->>Queue: {<br/>  id: 5001,<br/>  name: "Acme_Finance_Variant_001",<br/>  status: "In Progress",<br/>  stats: {<br/>    send_count: 50,<br/>    delivered_count: 49,<br/>    bounce_count: 1,<br/>    opened_count: 32,<br/>    clicked_count: 18,<br/>    submitted_count: 5<br/>  },<br/>  results: [<br/>    {<br/>      id: 1,<br/>      email: "john.smith@acme.com",<br/>      first_name: "John",<br/>      last_name: "Smith",<br/>      status: "Clicked",<br/>      ip: "192.168.1.100",<br/>      latitude: 48.8566,<br/>      longitude: 2.3522,<br/>      user_agent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64)",<br/>      timestamp: "2025-11-20T14:05:15Z"<br/>    },<br/>    {<br/>      id: 2,<br/>      email: "jane.doe@acme.com",<br/>      first_name: "Jane",<br/>      last_name: "Doe",<br/>      status: "Opened",<br/>      ip: "192.168.1.101",<br/>      latitude: 48.8600,<br/>      longitude: 2.3550,<br/>      user_agent: "Mozilla/5.0 (Macintosh; Intel Mac OS X)",<br/>      timestamp: "2025-11-20T14:04:30Z"<br/>    },<br/>    {<br/>      id: 3,<br/>      email: "bob.jones@acme.com",<br/>      first_name: "Bob",<br/>      status: "Submitted",<br/>      payload: {<br/>        email: "bob.jones@acme.com",<br/>        password: "***CAPTURED***"<br/>      },<br/>      ip: "192.168.1.102",<br/>      timestamp: "2025-11-20T14:06:45Z"<br/>    },<br/>    ... (47 more results)<br/>  ]<br/>}

            Queue->>Queue: PARSE AND NORMALIZE RESULTS<br/>For each result:<br/>1. Deduplicate (same email, same timestamp)<br/>2. Categorize event: sent | delivered | opened | clicked | submitted<br/>3. Extract metadata: IP, geolocation, user agent<br/>4. Validate timestamps<br/>5. Calculate event_type for each

            Queue->>DB: BULK INSERT campaign_events {<br/>  campaign_id: 1001,<br/>  variant: 1,<br/>  events: [<br/>    {<br/>      user_email: "john.smith@acme.com",<br/>      event_type: "clicked",<br/>      timestamp: "2025-11-20T14:05:15Z",<br/>      ip_address: "192.168.1.100",<br/>      latitude: 48.8566,<br/>      longitude: 2.3522,<br/>      user_agent: "Mozilla/5.0...",<br/>      ingestion_timestamp: NOW()<br/>    },<br/>    {<br/>      user_email: "jane.doe@acme.com",<br/>      event_type: "opened",<br/>      timestamp: "2025-11-20T14:04:30Z",<br/>      ...<br/>    },<br/>    {<br/>      user_email: "bob.jones@acme.com",<br/>      event_type: "submitted",<br/>      payload: {credentials_captured: true},<br/>      timestamp: "2025-11-20T14:06:45Z",<br/>      ...<br/>    }<br/>  ]<br/>}
            DB-->>Queue: ✅ Events stored (50 per variant)

            Queue->>Queue: UPDATE CAMPAIGN SUMMARY<br/>sent_count: 1500 (30 × 50)<br/>opened_count: 960 (accumulated from all variants)<br/>clicked_count: 540<br/>submitted_count: 135<br/>bounce_count: 15<br/>last_polled: NOW()

            Queue->>Cache: UPDATE campaign:1001:results {<br/>  sent: 1500,<br/>  delivered: 1485,<br/>  opened: 960,<br/>  clicked: 540,<br/>  submitted: 135,<br/>  bounce: 15,<br/>  last_updated: NOW(),<br/>  ttl: 300 (5 min refresh)<br/>}
            Cache-->>Queue: ✅ Updated for dashboard

            Note over Queue: Results stored and cached for real-time dashboard updates

        else ❌ GOPHISH API ERROR (500)
            GoPhish-->>Queue: {error: "Internal Server Error"}

            Queue->>Queue: Log error and retry<br/>Backoff: 30 seconds<br/>Retry count: 3

            Queue->>Queue: If retries exhausted:<br/>Alert admin and continue with cached data

        else ⏳ CAMPAIGN NOT FOUND (404)
            GoPhish-->>Queue: {error: "Campaign not found"}

            Queue->>DB: UPDATE campaigns SET status='error', error_message='Campaign not found in GoPhish'
            Queue->>API: Send alert to user

        end
    end

    Note over RedTeam,EmailServers: 📊 STEP 9: REAL-TIME DASHBOARD UPDATE

    Queue->>API: Publish event: "Campaign results updated"

    API->>Cache: GET campaign:1001:results
    Cache-->>API: {sent: 1500, opened: 960, clicked: 540, ...}

    API-->>Web: WebSocket: campaign_results_updated {<br/>  campaign_id: 1001,<br/>  sent: 1500,<br/>  delivered: 1485,<br/>  opened: 960,<br/>  clicked: 540,<br/>  submitted: 135,<br/>  open_rate: 0.646,<br/>  click_rate: 0.364,<br/>  submission_rate: 0.091<br/>}

    Web->>Web: UPDATE DASHBOARD IN REAL-TIME<br/>├─ Campaign card:<br/>│  ├─ Sent: 1500<br/>│  ├─ Opened: 960 (64.6%)<br/>│  ├─ Clicked: 540 (36.4%)<br/>│  ├─ Submitted: 135 (9.1%)<br/>│  └─ Last update: 2 seconds ago<br/>├─ Engagement funnel:<br/>│  sent→delivered→opened→clicked→submitted<br/>├─ Real-time metrics update every 10 seconds<br/>└─ Green indicator: "Campaign ACTIVE"

    Web-->>RedTeam: ✅ Real-time dashboard updated<br/><br/>📊 Live Campaign Metrics:<br/>├─ Sent: 1,500<br/>├─ Delivered: 1,485 (99%)<br/>├─ Opened: 960 (64.6%)<br/>├─ Clicked: 540 (36.4%) ⚠️<br/>├─ Submitted: 135 (9.1%) 🔴<br/>└─ Last update: 10 seconds ago

    Note over RedTeam,EmailServers: ⏱️ STEP 10: CAMPAIGN COMPLETION & FINALIZATION

    loop Until campaign ends or 2+ hours pass
        Queue->>GoPhish: GET /api/campaigns/5001/results
        Note over Queue: Continue polling...
    end

    Queue->>Queue: CAMPAIGN COMPLETION DETECTION<br/>Criteria:<br/>1. Launch time + 120 min has passed, OR<br/>2. No new events for 30 minutes, OR<br/>3. User manually stops campaign

    Queue->>GoPhish: GET /api/campaigns/5001<br/>(check campaign status)

    GoPhish-->>Queue: {<br/>  id: 5001,<br/>  status: "Completed",<br/>  stats: {<br/>    send_count: 50,<br/>    opened_count: 32,<br/>    clicked_count: 18,<br/>    submitted_count: 5<br/>  }<br/>}

    Queue->>DB: FINALIZE CAMPAIGN RESULTS<br/>UPDATE campaigns SET {<br/>  status: "completed",<br/>  completed_at: NOW(),<br/>  final_sent: 1500,<br/>  final_opened: 960,<br/>  final_clicked: 540,<br/>  final_submitted: 135,<br/>  final_bounce: 15<br/>}

    Queue->>DB: CALCULATE FINAL METRICS<br/>INSERT campaign_summary {<br/>  campaign_id: 1001,<br/>  total_sent: 1500,<br/>  total_delivered: 1485,<br/>  total_opened: 960,<br/>  total_clicked: 540,<br/>  total_submitted: 135,<br/>  delivery_rate: 0.99,<br/>  open_rate: 0.646,<br/>  click_rate: 0.364,<br/>  submission_rate: 0.091,<br/>  calculated_at: NOW()<br/>}
    DB-->>Queue: ✅ Finalized

    Queue->>API: Event: campaign_completed {campaign_id: 1001}

    API-->>Web: Event: Campaign 1001 completed

    Web-->>RedTeam: ✅ CAMPAIGN COMPLETED<br/><br/>📊 Final Results:<br/>├─ Sent: 1,500<br/>├─ Delivered: 1,485 (99%)<br/>├─ Opened: 960 (64.6%)<br/>├─ Clicked: 540 (36.4%)<br/>├─ Submitted: 135 (9.1%)<br/>├─ Bounced: 15 (1%)<br/>└─ Status: COMPLETED<br/><br/>[View Analysis] [Generate Rules] [Export Report] [Start New Campaign]

    Note over RedTeam,EmailServers: ⏱️ STEP 11: CAMPAIGN ARCHIVAL & CLEANUP

    Queue->>GoPhish: DELETE /api/campaigns/5001<br/>(Variant 1 - cleanup old campaign)
    Queue->>GoPhish: DELETE /api/campaigns/5002<br/>(Variant 2)<br/>... (delete all 30 variants)

    GoPhish-->>Queue: ✅ Campaigns deleted from GoPhish

    Queue->>DB: UPDATE gophish_campaigns SET {<br/>  status: "archived",<br/>  deleted_from_gophish_at: NOW()<br/>}<br/>WHERE campaign_id = 1001

    Queue->>Queue: ARCHIVE IN LONG-TERM STORAGE<br/>├─ Compress campaign_events table data<br/>├─ Move to cold storage (if using tiered storage)<br/>├─ Retain in PostgreSQL for 12 months minimum<br/>└─ Create backup snapshot

    Queue->>Cache: DELETE campaign:1001:*<br/>(Clean up Redis cache)

    Queue->>API: Event: campaign_archived {campaign_id: 1001}

    API-->>Web: Event: Campaign archived successfully

    Web-->>RedTeam: ✅ Campaign archived<br/>Data retained for 12 months for historical analysis<br/><br/>[View Archived Campaign] [Download Final Report]

    Note over RedTeam,EmailServers: ✅ GOPHISH INTEGRATION COMPLETE

    rect rgb(0, 150, 0)
        Note over RedTeam,EmailServers: ✅ GOPHISH CAMPAIGN LIFECYCLE COMPLETE
        Note over RedTeam,EmailServers: • Group Created ✅ (50 recipients)
        Note over RedTeam,EmailServers: • Templates Created ✅ (30 variants)
        Note over RedTeam,EmailServers: • Landing Page Created ✅ (with credential capture)
        Note over RedTeam,EmailServers: • SMTP Profile Configured ✅ (SendGrid)
        Note over RedTeam,EmailServers: • 30 Campaigns Launched ✅ (1500 emails sent)
        Note over RedTeam,EmailServers: • Real-time Polling ✅ (10-second intervals)
        Note over RedTeam,EmailServers: • Results Aggregated ✅ (960 opens, 540 clicks, 135 submissions)
        Note over RedTeam,EmailServers: • Campaign Completed ✅ (after 2 hours)
        Note over RedTeam,EmailServers: • Cleanup & Archive ✅ (GoPhish deleted, data retained)
        Note over RedTeam,EmailServers: • Ready for Analysis ✅ (Detection, Rule Generation, Dashboard)
    end
```