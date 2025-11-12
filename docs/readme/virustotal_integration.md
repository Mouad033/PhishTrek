# PhishTrek - Virus Total Deployment Architecture

## VirusTotal Pre-Flight Check

```mermaid
sequenceDiagram
    actor Analyst as Security Analyst<br/>(Red Team Lead)
    participant Web as Frontend<br/>(React UI)
    participant API as Backend API<br/>(FastAPI)
    participant DB as PostgreSQL<br/>(Campaign Metadata)
    participant Cache as Cache Layer<br/>(Redis)
    participant Queue as Job Queue<br/>(Celery/Redis)
    participant VT as VirusTotal API<br/>(v3 endpoints)
    participant GoPhish as GoPhish API<br/>(Deferred)
    participant Dashboard as Dashboard<br/>(KPI Display)

    Note over Analyst,Dashboard: 🔍 PHASE 3️⃣: VIRUSTOTAL DEFENSIVE ANALYSIS (Pre-GoPhish Gate)

    Note over Analyst,Dashboard: ⏱️ STEP 0: PREREQUISITES (from prior phases)

    Note over API: ✅ Campaign 1001 created with LLM-generated variants<br/>✅ Landing page URL ready: https://acme-hr-verify.example.com/login<br/>✅ 30 email templates prepared<br/>✅ 50 recipients validated<br/>Next: VT scanning BEFORE GoPhish launch

    Note over Analyst,Dashboard: ═══════════════════════════════════════════════════════
    Note over Analyst,Dashboard: ⏱️ STEP 1: CHECK REDIS CACHE (Fast Path)
    Note over Analyst,Dashboard: ═══════════════════════════════════════════════════════

    API->>API: Generate URL hash for cache lookup<br/>url: "https://acme-hr-verify.example.com/login"<br/>url_hash: SHA256(url) = "abc123def456..."

    API->>Cache: GET vt:url:abc123def456...

    alt ✅ CACHE HIT (URL previously scanned)
        Cache-->>API: {<br/>  analysis_id: "cached_analysis_xyz789",<br/>  verdict: "CLEAN",<br/>  risk_score: 0.002,<br/>  malicious_count: 0,<br/>  suspicious_count: 0,<br/>  undetected_count: 77,<br/>  cached_at: "2025-11-19T10:30:00Z",<br/>  expires_at: "2025-11-22T10:30:00Z",<br/>  ttl_remaining: 86400<br/>}

        API->>DB: INSERT vt_scan_log {<br/>  campaign_id: 1001,<br/>  url: "https://acme-hr-verify.example.com/login",<br/>  scan_method: "cache_hit",<br/>  verdict: "CLEAN",<br/>  risk_score: 0.002,<br/>  cached: true,<br/>  cached_since: "2025-11-19T10:30:00Z"<br/>}
        DB-->>API: ✅ Logged

        API-->>Web: {<br/>  status: "vt_approved",<br/>  campaign_id: 1001,<br/>  verdict: "CLEAN",<br/>  risk_score: 0.002,<br/>  source: "cache",<br/>  scan_time: "< 100ms",<br/>  message: "URL previously verified CLEAN. Proceeding with campaign..."<br/>}

        Web-->>Analyst: ✅ "URL verified CLEAN (cached result)<br/>Proceeding to GoPhish setup..."

        Note over API: Jump to DECISION GATE (CLEAN verdict → proceed to GoPhish)

    else ❌ CACHE MISS (URL not previously scanned or cache expired)
        Cache-->>API: nil

        Note over API: Proceed to Step 2 (URL submission to VT)

    end

    Note over Analyst,Dashboard: ═══════════════════════════════════════════════════════
    Note over Analyst,Dashboard: ⏱️ STEP 2: SUBMIT URL TO VIRUSTOTAL API
    Note over Analyst,Dashboard: ═══════════════════════════════════════════════════════

    Note over API,VT: Prepare and validate URL before submission

    API->>API: VALIDATE URL<br/>├─ Check URL format (RFC 3986)<br/>├─ Check length (max 2048 chars)<br/>├─ Check scheme (http/https only)<br/>├─ Verify no sensitive parameters<br/>├─ Check against internal blocklist<br/>└─ Validate domain ownership (DNS TXT record)

    API->>API: Result: ✅ URL is VALID and SAFE to submit

    API->>API: Retrieve VT API credentials from Secrets Manager<br/>API_KEY: "***encrypted_key_from_vault***"<br/>Rate limit config: 4 requests/minute (free tier)

    API->>Queue: CHECK RATE LIMIT QUOTA<br/>Current usage this minute: 2/4 calls<br/>Available: 2 calls remaining<br/>Proceed: YES ✅

    Note over API,VT: Submit URL to VirusTotal v3 API

    API->>VT: POST /api/v3/urls {<br/>  url: "https://acme-hr-verify.example.com/login"<br/>}<br/>Headers: {<br/>  x-apikey: "***VT_API_KEY***",<br/>  Content-Type: "application/x-www-form-urlencoded"<br/>}

    alt ✅ SUBMISSION SUCCESS (200 OK)
        VT-->>API: {<br/>  data: {<br/>    type: "url",<br/>    id: "dGhlIGJpZ2dlc3QgbGllIGluIHRoZSB1bml2ZXJzZQ==",<br/>    attributes: {<br/>      status: "queued",<br/>      last_submission_date: 1700472000,<br/>      last_analysis_date: null,<br/>      times_submitted: 1<br/>    }<br/>  },<br/>  links: {<br/>    self: "https://www.virustotal.com/api/v3/urls/dGhlIGJpZ2dlc3QgbGllIGluIHRoZSB1bml2ZXJzZQ=="<br/>  }<br/>}

        API->>DB: INSERT vt_analyses {<br/>  campaign_id: 1001,<br/>  url: "https://acme-hr-verify.example.com/login",<br/>  url_hash: "abc123def456...",<br/>  vt_analysis_id: "dGhlIGJpZ2dlc3QgbGllIGluIHRoZSB1bml2ZXJzZQ==",<br/>  status: "queued",<br/>  submission_status: "success",<br/>  submitted_at: NOW(),<br/>  times_submitted: 1,<br/>  last_check: NOW()<br/>}
        DB-->>API: ✅ Stored (vt_scan_id: 500)

        API->>DB: INSERT campaign_vt_scans {<br/>  campaign_id: 1001,<br/>  vt_scan_id: 500,<br/>  status: "pending",<br/>  polling_started: false<br/>}
        DB-->>API: ✅ Linked to campaign

        API->>Queue: ENQUEUE poll_vt_analysis_job {<br/>  vt_analysis_id: "dGhlIGJpZ2dlc3QgbGllIGluIHRoZSB1bml2ZXJzZQ==",<br/>  campaign_id: 1001,<br/>  url: "https://acme-hr-verify.example.com/login",<br/>  poll_interval: 10,<br/>  max_polls: 60,<br/>  max_duration: 600,<br/>  backoff_strategy: "exponential"<br/>}
        Queue-->>API: ✅ Polling job queued (job_id: job_vt_12345)

        API->>Cache: SET vt:url:abc123def456... {<br/>  analysis_id: "dGhlIGJpZ2dlc3QgbGllIGluIHRoZSB1bml2ZXJzZQ==",<br/>  status: "queued",<br/>  submitted_at: NOW(),<br/>  ttl: 3600 (1 hour)<br/>}
        Cache-->>API: ✅ Cached for fast re-submission detection

        API->>API: Increment rate limit counter<br/>Calls used this minute: 3/4

        API-->>Web: {<br/>  status: "vt_scanning",<br/>  campaign_id: 1001,<br/>  analysis_id: "dGhlIGJpZ2dlc3QgbGllIGluIHRoZSB1bml2ZXJzZQ==",<br/>  message: "Landing page URL submitted to VirusTotal. Scanning...",<br/>  estimated_time: "30-120 seconds",<br/>  poll_job_id: "job_vt_12345"<br/>}

        Web-->>Analyst: ⏳ SCANNING IN PROGRESS<br/>URL: https://acme-hr-verify.example.com/login<br/>Status: Queued for analysis<br/>Estimated time: 30-120 seconds<br/>Progress: [████░░░░░░░░░░░░░░] 20% - Waiting for VT analysis...

    else ⚠️ RATE LIMIT EXCEEDED (429 Too Many Requests)
        VT-->>API: {<br/>  error: {<br/>    code: "NotFoundError",<br/>    message: "You are not allowed to perform this operation"<br/>  },<br/>  data: {status: "rate_limited"}<br/>}

        API->>API: LOG RATE LIMIT EVENT<br/>Rate limit exceeded at: NOW()<br/>Remaining quota: 0/4<br/>Reset time: (in 60 seconds)

        API->>Queue: ENQUEUE retry_vt_submission {<br/>  campaign_id: 1001,<br/>  url: "https://acme-hr-verify.example.com/login",<br/>  retry_count: 1,<br/>  backoff_delay: 65,<br/>  max_retries: 3<br/>}
        Queue-->>API: ✅ Retry scheduled for 65 seconds later

        API->>API: Fall back to local heuristics for now
        API->>API: RUN LOCAL HEURISTICS ANALYSIS<br/>├─ Domain blocklist check: ✅ NOT in blocklist<br/>├─ Domain age check: ✅ 14 years old (legitimate)<br/>├─ SPF/DKIM/DMARC check: ✅ All present<br/>├─ URL structure check: ✅ No suspicious patterns<br/>└─ Local verdict: "CLEAN" (with note: VT pending)

        API-->>Web: {<br/>  status: "rate_limited",<br/>  campaign_id: 1001,<br/>  message: "VT rate limited. Using local analysis. Will retry VT in 65 seconds.",<br/>  local_verdict: "CLEAN",<br/>  local_confidence: 0.95,<br/>  warning: "VT scan pending - using local checks as fallback"<br/>}

        Web-->>Analyst: ⚠️ VIRUSTOTAL RATE LIMITED<br/>Using local security checks (confidence: 95%)<br/>Local verdict: CLEAN<br/>VT scan will retry automatically in 65 seconds...<br/>Campaign can proceed, but VT verification pending

    else ❌ AUTHENTICATION ERROR (401 Unauthorized)
        VT-->>API: {<br/>  error: {<br/>    code: "AuthenticationRequiredError",<br/>    message: "Invalid API key"<br/>  }<br/>}

        API->>API: LOG ERROR<br/>Error: Invalid VT API key<br/>Severity: CRITICAL<br/>Action required: Rotate API key

        API->>API: Alert admin<br/>Alert: VT API key invalid or expired<br/>Contact: devops-team@phistrek.internal

        API->>API: Fall back to local heuristics
        API-->>Web: {<br/>  status: "vt_error",<br/>  campaign_id: 1001,<br/>  error_type: "auth_failed",<br/>  message: "VT API authentication failed. Using local analysis.",<br/>  local_verdict: "CLEAN",<br/>  local_confidence: 0.90,<br/>  warning: "Admin notified. VT verification unavailable."<br/>}

        Web-->>Analyst: ❌ VIRUSTOTAL AUTHENTICATION ERROR<br/>Using local security checks only<br/>⚠️ Admin notification sent<br/>Campaign can proceed at reduced confidence level

    else ❌ SERVER ERROR (5xx)
        VT-->>API: {<br/>  error: {<br/>    code: "InternalServerError",<br/>    message: "Internal Server Error"<br/>  }<br/>}

        API->>Queue: ENQUEUE retry_vt_submission {<br/>  backoff_delay: 30,<br/>  max_retries: 3,<br/>  jitter: true<br/>}

        API->>API: Fall back to local heuristics
        API-->>Web: {<br/>  status: "vt_error",<br/>  message: "VT temporarily unavailable. Using local analysis. Will retry.",<br/>  local_verdict: "CLEAN"<br/>}

        Web-->>Analyst: ⚠️ VIRUSTOTAL TEMPORARILY UNAVAILABLE<br/>Using local checks | Will retry automatically

    else ❌ INVALID URL (400 Bad Request)
        VT-->>API: {<br/>  error: {<br/>    code: "BadRequest",<br/>    message: "Invalid URL format"<br/>  }<br/>}

        API->>DB: UPDATE vt_analyses SET {<br/>  status: "failed",<br/>  error: "Invalid URL format",<br/>  needs_investigation: true<br/>}

        API-->>Web: {<br/>  status: "error",<br/>  message: "URL format invalid. Please review landing page configuration."<br/>}

        Web-->>Analyst: ❌ INVALID URL FORMAT<br/>Please verify landing page URL configuration

    end

    Note over Analyst,Dashboard: ═══════════════════════════════════════════════════════
    Note over Analyst,Dashboard: ⏱️ STEP 3: ASYNCHRONOUS POLLING (Background Job)
    Note over Analyst,Dashboard: ═══════════════════════════════════════════════════════

    Note over Queue,VT: Background Job: poll_vt_analysis_job (Non-blocking to user)

    loop Every 10 seconds (max 60 polls = 10 minutes total)
        Queue->>Queue: CHECK ELAPSED TIME<br/>Started: NOW() - 30 seconds ago<br/>Elapsed: 30 seconds<br/>Max duration: 600 seconds<br/>Status: ✅ Continue polling

        Queue->>VT: GET /api/v3/analyses/dGhlIGJpZ2dlc3QgbGllIGluIHRoZSB1bml2ZXJzZQ==<br/>Headers: {x-apikey: "***VT_API_KEY***"}

        alt ✅ ANALYSIS COMPLETED (200 OK - status: completed)
            VT-->>Queue: {<br/>  data: {<br/>    type: "analysis",<br/>    id: "dGhlIGJpZ2dlc3QgbGllIGluIHRoZSB1bml2ZXJzZQ==",<br/>    attributes: {<br/>      status: "completed",<br/>      last_analysis_stats: {<br/>        malicious: 2,<br/>        suspicious: 1,<br/>        undetected: 74,<br/>        harmless: 0,<br/>        timeout: 0<br/>      },<br/>      last_analysis_results: {<br/>        "Kaspersky": {<br/>          category: "phishing",<br/>          engine_name: "Kaspersky",<br/>          result: "phishing",<br/>          engine_update: "20251120"<br/>        },<br/>        "McAfee": {<br/>          category: "suspicious",<br/>          engine_name: "McAfee",<br/>          result: "potentially_unwanted",<br/>          engine_update: "20251120"<br/>        },<br/>        "Sophos": {<br/>          category: "undetected",<br/>          engine_name: "Sophos",<br/>          result: null,<br/>          engine_update: "20251120"<br/>        },<br/>        ... (74 more vendors - all undetected)<br/>      },<br/>      last_analysis_date: 1700472045,<br/>      categories: {<br/>        "Kaspersky": "phishing",<br/>        "McAfee": "suspicious"<br/>      }<br/>    }<br/>  }<br/>}

            Queue->>Queue: PARSE ANALYSIS RESULTS<br/>├─ Total vendors: 77<br/>├─ Malicious: 2 (Kaspersky, McAfee)<br/>├─ Suspicious: 1 (McAfee flagged as potentially_unwanted)<br/>├─ Undetected: 74<br/>├─ Harmless: 0<br/>└─ Overall: Low threat

            Queue->>Queue: CALCULATE RISK SCORE<br/>Formula: (malicious × 1.0 + suspicious × 0.5) / total_vendors<br/>Calculation: (2 × 1.0 + 1 × 0.5) / 77 = 2.5 / 77 ≈ 0.0325<br/>Risk Score: 0.0325 (3.25%)

            Queue->>Queue: DETERMINE VERDICT<br/>Thresholds:<br/>├─ MALICIOUS: risk_score >= 0.15 (15%)<br/>├─ SUSPICIOUS: 0.07 <= risk_score < 0.15<br/>├─ CAUTION: 0.01 <= risk_score < 0.07<br/>└─ CLEAN: risk_score < 0.01<br/><br/>Result: risk_score = 0.0325<br/>Verdict: ✅ CLEAN (exceeds CLEAN threshold)

            Queue->>Queue: EXTRACT VENDOR DETAILS<br/>Detections:<br/>├─ Kaspersky: "phishing"<br/>├─ McAfee: "potentially_unwanted"<br/>└─ (75 other vendors: undetected)<br/><br/>Top vendors by detection: Kaspersky (phishing)

            Queue->>DB: UPDATE vt_analyses {<br/>  campaign_id: 1001,<br/>  status: "completed",<br/>  polling_completed: true,<br/>  total_vendors: 77,<br/>  malicious_count: 2,<br/>  suspicious_count: 1,<br/>  undetected_count: 74,<br/>  harmless_count: 0,<br/>  risk_score: 0.0325,<br/>  verdict: "CLEAN",<br/>  analysis_date: "2025-11-20T14:02:45Z",<br/>  vendor_results: {...complete JSON...},<br/>  top_detections: ["Kaspersky: phishing", "McAfee: suspicious"],<br/>  completed_at: NOW(),<br/>  ttl_expires_at: NOW() + 259200  (3 days in seconds)<br/>}
            DB-->>Queue: ✅ Updated (vt_scan_id: 500)

            Queue->>Cache: UPDATE vt:url:abc123def456... {<br/>  analysis_id: "dGhlIGJpZ2dlc3QgbGllIGluIHRoZSB1bml2ZXJzZQ==",<br/>  status: "completed",<br/>  verdict: "CLEAN",<br/>  risk_score: 0.0325,<br/>  malicious: 2,<br/>  suspicious: 1,<br/>  undetected: 74,<br/>  completed_at: NOW(),<br/>  ttl: 259200 (3 days)<br/>}
            Cache-->>Queue: ✅ Cached for 3 days

            Queue->>Queue: BREAK POLLING LOOP<br/>Reason: Analysis complete, verdict determined

            Queue->>API: POST /internal/webhooks/vt_analysis_complete {<br/>  campaign_id: 1001,<br/>  verdict: "CLEAN",<br/>  risk_score: 0.0325<br/>}

            API->>API: TRIGGER DECISION GATE<br/>Proceed to next phase (GoPhish setup)

            Note over Queue: Analysis complete - exit polling loop

        else ⏳ ANALYSIS IN PROGRESS (status: in_progress)
            VT-->>Queue: {<br/>  data: {<br/>    attributes: {<br/>      status: "in_progress",<br/>      last_analysis_stats: {<br/>        malicious: null,<br/>        suspicious: null,<br/>        undetected: null,<br/>        harmless: null<br/>      }<br/>    }<br/>  }<br/>}

            Queue->>Queue: Analysis still processing<br/>Elapsed time: 30 seconds<br/>Estimated remaining: 30-90 seconds<br/>Continue polling: YES

            Queue->>Queue: Exponential backoff strategy<br/>Next poll delay: 10 seconds<br/>Total polls so far: 3

            Note over Queue: Still processing - will retry in 10s

        else ⏳ ANALYSIS QUEUED (status: queued)
            VT-->>Queue: {<br/>  data: {<br/>    attributes: {<br/>      status: "queued",<br/>      position_in_queue: 42<br/>    }<br/>  }<br/>}

            Queue->>Queue: Analysis queued at position 42<br/>Estimated wait: 60-120 seconds<br/>Continue polling

            Note over Queue: Waiting in queue - will retry

        else ⚠️ ANALYSIS TIMEOUT
            VT-->>Queue: {error: "Analysis timeout after 10 minutes"}

            Queue->>DB: UPDATE vt_analyses SET {<br/>  status: "timeout",<br/>  error: "Analysis did not complete within 10 minutes",<br/>  final_verdict: "UNKNOWN"<br/>}

            Queue->>Queue: FALLBACK: Use local heuristics<br/>Local verdict: CLEAN (from initial check)

            Queue->>API: POST /internal/webhooks/vt_timeout {<br/>  campaign_id: 1001,<br/>  final_verdict: "UNKNOWN",<br/>  fallback_verdict: "CLEAN"<br/>}

            Note over Queue: Timeout - use fallback verdict

        else ❌ ANALYSIS ERROR (400/401/429/5xx)
            VT-->>Queue: {error: "API Error"}

            Queue->>Queue: LOG ERROR<br/>Error type: API error<br/>Retry count: 2/3<br/>Next retry in: 30 seconds

            Queue->>DB: UPDATE vt_analyses SET {<br/>  status: "error",<br/>  error_count: 2,<br/>  last_error: "API error on poll attempt 3",<br/>  last_error_at: NOW()<br/>}

            Queue->>Queue: EXPONENTIAL BACKOFF<br/>Backoff: 30 seconds<br/>Jitter: ±5 seconds

            Queue->>Queue: Check max retries<br/>Retries: 2/3<br/>Status: Continue polling

            Note over Queue: Error encountered - will retry

        end
    end

    par POLLING COMPLETE: Update Dashboard
        API->>Dashboard: POST /events/vt_analysis_complete {<br/>  campaign_id: 1001,<br/>  verdict: "CLEAN",<br/>  risk_score: 0.0325<br/>}

        Dashboard->>Web: WebSocket: vt_status_update {<br/>  campaign_id: 1001,<br/>  status: "vt_analysis_complete",<br/>  verdict: "CLEAN",<br/>  message: "VirusTotal analysis complete"<br/>}

        Web-->>Analyst: ✅ VIRUSTOTAL ANALYSIS COMPLETE<br/>Verdict: CLEAN ✅<br/>Risk Score: 3.25%<br/>Vendors detecting: 2/77<br/>Top detection: Kaspersky (phishing)<br/>Status: APPROVED FOR GOPHISH
    end

    Note over Analyst,Dashboard: ═══════════════════════════════════════════════════════
    Note over Analyst,Dashboard: ⏱️ STEP 4: DECISION GATE (VT + Local Heuristics)
    Note over Analyst,Dashboard: ═══════════════════════════════════════════════════════

    Queue->>Queue: EVALUATE VT VERDICT & RISK SCORE<br/><br/>Verdict: CLEAN<br/>Risk Score: 0.0325 (3.25%)<br/>Thresholds:<br/>├─ MALICIOUS (block): >= 0.15 (15%)<br/>├─ SUSPICIOUS (warn): 0.07-0.15 (7-15%)<br/>├─ CAUTION (monitor): 0.01-0.07 (1-7%)<br/>└─ CLEAN (approve): < 0.01 (< 1%)<br/><br/>Decision: CLEAN - Proceed to GoPhish

    alt 🔴 VERDICT = MALICIOUS (risk_score >= 0.15)

        API->>DB: UPDATE campaigns SET {<br/>  status: "blocked_vt_malicious",<br/>  vt_verdict: "MALICIOUS",<br/>  vt_risk_score: 0.18,<br/>  vt_malicious_vendors: 12,<br/>  block_reason: "URL flagged as MALICIOUS by 12 vendors",<br/>  blocked_at: NOW(),<br/>  action_required: "Review domain or use different URL"<br/>}
        DB-->>API: ✅ Updated

        API->>API: Block campaign from GoPhish launch
        API->>API: Alert administrator<br/>Subject: "Campaign 1001 blocked - MALICIOUS URL detected"<br/>Risk: HIGH

        API-->>Web: {<br/>  status: "blocked",<br/>  campaign_id: 1001,<br/>  reason: "MALICIOUS_URL",<br/>  severity: "CRITICAL",<br/>  details: {<br/>    vt_verdict: "MALICIOUS",<br/>    vendors_flagging: 12,<br/>    risk_score: 0.18,<br/>    top_detections: ["Kaspersky: Trojan", "McAfee: Malware", "Sophos: Phishing"],<br/>    recommendation: "Change domain. URL is flagged as malicious."<br/>  },<br/>  actions: ["change_domain", "cancel_campaign"]<br/>}

        Web-->>Analyst: 🔴 CAMPAIGN BLOCKED - MALICIOUS URL<br/><br/>VirusTotal Verdict: MALICIOUS<br/>Risk Score: 18%<br/>Vendors detecting: 12/77<br/><br/>Top Detections:<br/>├─ Kaspersky: Trojan<br/>├─ McAfee: Malware<br/>└─ Sophos: Phishing<br/><br/>❌ Campaign cannot proceed<br/>Action Required: Change domain or cancel campaign

        Dashboard->>Dashboard: RED ALERT: Campaign blocked

    else 🟡 VERDICT = SUSPICIOUS (0.07 <= risk_score < 0.15)

        API->>DB: UPDATE campaigns SET {<br/>  status: "pending_vt_approval",<br/>  vt_verdict: "SUSPICIOUS",<br/>  vt_risk_score: 0.084,<br/>  vt_suspicious_vendors: 7,<br/>  requires_user_approval: true,<br/>  approval_required_reason: "VT flagged URL as suspicious by 7 vendors"<br/>}
        DB-->>API: ✅ Updated

        API-->>Web: {<br/>  status: "warning",<br/>  campaign_id: 1001,<br/>  reason: "SUSPICIOUS_URL",<br/>  severity: "HIGH",<br/>  details: {<br/>    vt_verdict: "SUSPICIOUS",<br/>    vendors_flagging: 7,<br/>    risk_score: 0.084,<br/>    top_detections: ["Kaspersky: phishing (5)", "McAfee: suspicious (2)"],<br/>    recommendation: "7/77 vendors flag this URL. Review before proceeding."<br/>  },<br/>  user_actions: {<br/>    approve: "Proceed with campaign (acknowledge risk)",<br/>    change_domain: "Change URL and re-scan",<br/>    cancel: "Cancel campaign"<br/>  }<br/>}

        Web-->>Analyst: 🟡 WARNING: SUSPICIOUS URL DETECTED<br/><br/>VirusTotal Verdict: SUSPICIOUS<br/>Risk Score: 8.4%<br/>Vendors detecting: 7/77<br/><br/>Top Detections:<br/>├─ Kaspersky: Phishing (5 vendors)<br/>└─ McAfee: Suspicious (2 vendors)<br/><br/>⚠️ User review required<br/><br/>[Proceed Anyway] [Change Domain] [Cancel]

        Dashboard->>Dashboard: YELLOW ALERT: Suspicious URL - user approval needed

        par User chooses: PROCEED ANYWAY
            Analyst->>Web: Click "Proceed Anyway"
            Web->>API: POST /campaigns/1001/approve-vt-warning {<br/>  approval_reason: "known_false_positive_from_kaspersky"<br/>}

            API->>DB: UPDATE campaigns SET {<br/>  vt_user_approved: true,<br/>  vt_approval_timestamp: NOW(),<br/>  vt_approval_reason: "known_false_positive_from_kaspersky",<br/>  status: "vt_approved"<br/>}
            DB-->>API: ✅ Updated

            API-->>Web: {status: "approved", message: "Campaign approved. Proceeding to GoPhish setup..."}
            Web-->>Analyst: ✅ "Proceeding with campaign..."

            Note over API: Continue to GoPhish phase

        or User chooses: CHANGE DOMAIN
            Analyst->>Web: Click "Change Domain"
            Web->>Analyst: Display domain change form

            Analyst->>Web: Enter new domain: "acme-finance-check.com"
            Web->>API: PATCH /campaigns/1001/landing-page {domain: "acme-finance-check.com"}

            API->>API: Generate new landing page URL<br/>old_url: "https://acme-hr-verify.example.com/login"<br/>new_url: "https://acme-finance-check.com/login"

            API->>Cache: DELETE vt:url:abc123def456...<br/>(Clear old URL from cache)

            API->>VT: POST /api/v3/urls {url: "https://acme-finance-check.com/login"}

            Note over API: Restart VT scanning with new domain

            Web-->>Analyst: ⏳ "Re-scanning new domain with VirusTotal..."

        or User chooses: CANCEL
            Analyst->>Web: Click "Cancel Campaign"
            Web->>API: POST /campaigns/1001/cancel

            API->>DB: UPDATE campaigns SET {<br/>  status: "cancelled",<br/>  cancelled_at: NOW(),<br/>  cancellation_reason: "User cancelled due to VT SUSPICIOUS verdict"<br/>}
            DB-->>API: ✅ Updated

            API-->>Web: {status: "cancelled"}
            Web-->>Analyst: ❌ "Campaign cancelled"

            Dashboard->>Dashboard: Campaign cancelled - not proceeding to GoPhish

        end

    else ✅ VERDICT = CLEAN or CAUTION (risk_score < 0.07)

        API->>DB: UPDATE campaigns SET {<br/>  status: "vt_approved",<br/>  vt_verdict: "CLEAN",<br/>  vt_risk_score: 0.0325,<br/>  vt_malicious_vendors: 2,<br/>  approved_for_gophish: true,<br/>  vt_approval_timestamp: NOW(),<br/>  vt_approval_method: "automatic"<br/>}
        DB-->>API: ✅ Updated

        API->>Cache: SET campaign:1001:vt_approved {<br/>  verdict: "CLEAN",<br/>  risk_score: 0.0325,<br/>  approved_at: NOW(),<br/>  ttl: 3600<br/>}

        API-->>Web: {<br/>  status: "approved",<br/>  campaign_id: 1001,<br/>  vt_verdict: "CLEAN",<br/>  risk_score: 0.0325,<br/>  message: "URL verified CLEAN by VirusTotal. Ready for GoPhish campaign launch."<br/>}

        Web-->>Analyst: ✅ URL APPROVED<br/><br/>VirusTotal Verdict: CLEAN ✅<br/>Risk Score: 3.25%<br/>Vendors detecting: 2/77<br/><br/>Campaign approved and ready for GoPhish setup<br/><br/>[Proceed to GoPhish] [View VT Report]

        Dashboard->>Dashboard: GREEN APPROVAL: URL verified CLEAN

        API->>API: Trigger next phase<br/>Phase: GoPhish Campaign Setup

    end

    Note over Analyst,Dashboard: ═══════════════════════════════════════════════════════
    Note over Analyst,Dashboard: 📊 STEP 5: VIRUSTOTAL ENRICHMENT & LOGGING
    Note over Analyst,Dashboard: ═══════════════════════════════════════════════════════

    par Enrich Campaign with VT Intel

        API->>DB: INSERT campaign_vt_enrichment {<br/>  campaign_id: 1001,<br/>  vt_analysis_id: "dGhlIGJpZ2dlc3QgbGllIGluIHRoZSB1bml2ZXJzZQ==",<br/>  vt_verdict: "CLEAN",<br/>  vt_risk_score: 0.0325,<br/>  vt_url: "https://acme-hr-verify.example.com/login",<br/>  scanning_time_seconds: 45,<br/>  total_vendors: 77,<br/>  malicious_vendors: 2,<br/>  suspicious_vendors: 1,<br/>  undetected_vendors: 74,<br/>  top_detections: ["Kaspersky: phishing", "McAfee: suspicious"],<br/>  scan_completion_rate: 1.0,<br/>  enrichment_timestamp: NOW()<br/>}
        DB-->>API: ✅ Stored

    and Log VT Transaction

        API->>DB: INSERT vt_transaction_log {<br/>  timestamp: NOW(),<br/>  campaign_id: 1001,<br/>  operation: "url_submission_and_polling",<br/>  url: "https://acme-hr-verify.example.com/login",<br/>  analysis_id: "dGhlIGJpZ2dlc3QgbGllIGluIHRoZSB1bml2ZXJzZQ==",<br/>  api_calls_made: 7,<br/>  total_duration_seconds: 45,<br/>  cache_status: "miss",<br/>  verdict: "CLEAN",<br/>  risk_score: 0.0325,<br/>  user_id: 42<br/>}
        DB-->>API: ✅ Logged

    and Generate VT Report

        API->>API: Generate VT Analysis Report<br/>include: {<br/>  url,<br/>  verdict,<br/>  risk_score,<br/>  vendor_statistics,<br/>  top_detections,<br/>  scanning_time,<br/>  recommendation<br/>}

        API->>Cache: SET campaign:1001:vt_report {report_data, ttl: 86400}

    end

    Note over Analyst,Dashboard: ═══════════════════════════════════════════════════════
    Note over Analyst,Dashboard: ✅ VIRUSTOTAL PHASE COMPLETE - DECISION MADE
    Note over Analyst,Dashboard: ═══════════════════════════════════════════════════════

    rect rgb(0, 150, 0)
        Note over Analyst,Dashboard: ✅ VIRUSTOTAL INTEGRATION COMPLETE
        Note over Analyst,Dashboard: • URL Cache Check ✅ (miss)
        Note over Analyst,Dashboard: • URL Submission ✅ (queued)
        Note over Analyst,Dashboard: • Analysis Polling ✅ (10s intervals × 5 polls = 45 seconds)
        Note over Analyst,Dashboard: • Risk Score Calculation ✅ (0.0325 = 3.25%)
        Note over Analyst,Dashboard: • Verdict Determination ✅ (CLEAN)
        Note over Analyst,Dashboard: • Decision Gate ✅ (APPROVED for GoPhish)
        Note over Analyst,Dashboard: • Campaign Enrichment ✅ (VT intel stored)
        Note over Analyst,Dashboard: • Transaction Logging ✅ (7 API calls logged)
        Note over Analyst,Dashboard: • Ready for GoPhish ✅ (Next phase: Campaign execution)
    end

    Note over Analyst,Dashboard: 🎯 STATUS: APPROVED FOR GOPHISH LAUNCH
    Note over Analyst,Dashboard: Campaign 1001 cleared for execution
    Note over Analyst,Dashboard: Proceeding to Phase 4: GoPhish Campaign Setup
```