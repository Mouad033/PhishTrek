# PhishTrek -  Real-Time Metrics & Analytics Platform

## KPI Aggregation & Real-Time Dashboard Display

```mermaid
sequenceDiagram
    actor SOCAnalyst as SOC Analyst/<br/>CISO
    participant Web as Frontend Dashboard<br/>(React + Tailwind)
    participant API as Backend API<br/>(FastAPI)
    participant DB as PostgreSQL<br/>(Campaign Data)
    participant Cache as Cache Layer<br/>(Redis)
    participant Queue as Job Queue<br/>(Celery)
    participant MetricsEngine as Metrics Engine<br/>(Aggregation)
    participant AlertSys as Alert System<br/>(Threshold Engine)
    participant WebSocket as WebSocket Server<br/>(Real-time Push)
    participant Export as Export Module<br/>(PDF/PPT/CSV)

    Note over SOCAnalyst,Export: 📊 PHASE 9️⃣: KPI DASHBOARD & METRICS AGGREGATION

    Note over SOCAnalyst,Export: ⏱️ PREREQUISITES (from prior phases)

    Note over SOCAnalyst: ✅ Campaign 1001 completed<br/>✅ 1500 emails sent (960 opened, 540 clicked, 135 submitted)<br/>✅ VirusTotal scan: CLEAN verdict<br/>✅ Detection analysis: 74% detected, 390 false negatives<br/>✅ 4 Sigma rules generated and approved<br/>Next: Aggregate all metrics into KPI dashboard

    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════
    Note over SOCAnalyst,Export: ⏱️ STEP 1: RETRIEVE CAMPAIGN DATA FOR KPI CALCULATION
    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════

    SOCAnalyst->>Web: 1. Open PhishTrek Dashboard
    Web->>Web: Load dashboard UI (React components)

    SOCAnalyst->>Web: 2. Navigate to Campaign 1001 overview
    Web->>API: GET /campaigns/1001/dashboard?include=full_metrics

    API->>Cache: GET campaign:1001:dashboard_data

    alt ✅ CACHE HIT (< 300 seconds old)
        Cache-->>API: {<br/>  campaign_id: 1001,<br/>  kpis: {...cached KPIs...},<br/>  cached_at: "2025-11-20T14:45:00Z",<br/>  ttl_remaining: 180<br/>}

        API-->>Web: {<br/>  source: "cache",<br/>  campaign_data: {...},<br/>  response_time: "25ms"<br/>}

        Web->>Web: Display dashboard from cache
        Note over Web: Fast response from cache

    else ❌ CACHE MISS (Expired or first request)
        Cache-->>API: nil

        Note over API: Proceed to full KPI calculation

    end

    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════
    Note over SOCAnalyst,Export: ⏱️ STEP 2: AGGREGATE ENGAGEMENT METRICS
    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════

    par Calculate Engagement KPIs

        Queue->>DB: SELECT COUNT(*) FILTER (WHERE event_type = 'sent')<br/>FROM campaign_events WHERE campaign_id = 1001

        DB-->>Queue: sent_count = 1500

        Queue->>Queue: SENT METRIC<br/>sent_count: 1500<br/>unique_recipients: 950

        par Calculate Delivery Rate

            Queue->>DB: SELECT COUNT(*) FILTER (WHERE event_type = 'delivered')<br/>FROM campaign_events WHERE campaign_id = 1001

            DB-->>Queue: delivered_count = 1485

            Queue->>DB: SELECT COUNT(*) FILTER (WHERE event_type = 'bounce')<br/>FROM campaign_events WHERE campaign_id = 1001

            DB-->>Queue: bounce_count = 15

            Queue->>Queue: DELIVERY METRICS<br/>├─ delivered_count: 1485<br/>├─ bounce_count: 15<br/>├─ delivery_rate: (1485 / 1500) × 100 = 99%<br/>└─ bounce_rate: (15 / 1500) × 100 = 1%

        and Calculate Open Rate

            Queue->>DB: SELECT COUNT(*) FILTER (WHERE event_type = 'opened')<br/>FROM campaign_events WHERE campaign_id = 1001

            DB-->>Queue: opened_count = 960

            Queue->>DB: SELECT COUNT(DISTINCT user_email) FILTER (WHERE event_type = 'opened')<br/>FROM campaign_events WHERE campaign_id = 1001

            DB-->>Queue: unique_openers = 845

            Queue->>Queue: OPEN METRICS<br/>├─ opened_count: 960<br/>├─ open_rate: (960 / 1485) × 100 = 64.6%<br/>├─ unique_openers: 845<br/>├─ unique_open_rate: (845 / 950) × 100 = 88.9%<br/>└─ opens_per_user: 960 / 845 ≈ 1.14 (avg)

        and Calculate Click Rate (Phish-Prone %)

            Queue->>DB: SELECT COUNT(*) FILTER (WHERE event_type = 'clicked')<br/>FROM campaign_events WHERE campaign_id = 1001

            DB-->>Queue: clicked_count = 540

            Queue->>DB: SELECT COUNT(DISTINCT user_email) FILTER (WHERE event_type = 'clicked')<br/>FROM campaign_events WHERE campaign_id = 1001

            DB-->>Queue: unique_clickers = 485

            Queue->>Queue: CLICK METRICS<br/>├─ clicked_count: 540<br/>├─ click_rate: (540 / 1485) × 100 = 36.4%<br/>├─ unique_clickers: 485<br/>├─ phish_prone_percentage: (485 / 950) × 100 = 51%<br/>├─ click_rate_on_opened: (540 / 960) × 100 = 56.3%<br/>└─ clicks_per_user: 540 / 485 ≈ 1.11 (avg)

        and Calculate Submission Rate

            Queue->>DB: SELECT COUNT(*) FILTER (WHERE event_type = 'submitted')<br/>FROM campaign_events WHERE campaign_id = 1001

            DB-->>Queue: submitted_count = 135

            Queue->>DB: SELECT COUNT(DISTINCT user_email) FILTER (WHERE event_type = 'submitted')<br/>FROM campaign_events WHERE campaign_id = 1001

            DB-->>Queue: unique_submitted = 128

            Queue->>Queue: SUBMISSION METRICS<br/>├─ submitted_count: 135<br/>├─ submission_rate: (135 / 1485) × 100 = 9.1%<br/>├─ unique_submitted: 128<br/>├─ credential_compromise_rate: (128 / 950) × 100 = 13.5%<br/>└─ resubmission_rate: (135 / 128) - 1 ≈ 5.5%

        and Calculate Report Rate

            Queue->>DB: SELECT COUNT(*) FILTER (WHERE event_type = 'reported')<br/>FROM campaign_events WHERE campaign_id = 1001

            DB-->>Queue: reported_count = 18

            Queue->>DB: SELECT COUNT(DISTINCT user_email) FILTER (WHERE event_type = 'reported')<br/>FROM campaign_events WHERE campaign_id = 1001

            DB-->>Queue: unique_reporters = 17

            Queue->>Queue: REPORT METRICS<br/>├─ reported_count: 18<br/>├─ report_rate: (18 / 1485) × 100 = 1.2%<br/>├─ unique_reporters: 17<br/>├─ unique_report_rate: (17 / 950) × 100 = 1.8%<br/>└─ awareness_indicator: LOW (only 1.8% reported)

    end

    Queue->>MetricsEngine: INSERT engagement_metrics {<br/>  campaign_id: 1001,<br/>  sent: 1500,<br/>  delivered: 1485,<br/>  bounced: 15,<br/>  opened: 960,<br/>  clicked: 540,<br/>  submitted: 135,<br/>  reported: 18,<br/>  delivery_rate: 0.99,<br/>  open_rate: 0.646,<br/>  click_rate: 0.364,<br/>  submission_rate: 0.091,<br/>  report_rate: 0.012,<br/>  phish_prone_percent: 0.51,<br/>  unique_openers: 845,<br/>  unique_clickers: 485,<br/>  unique_submitted: 128,<br/>  unique_reporters: 17<br/>}
    MetricsEngine-->>Queue: ✅ Stored

    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════
    Note over SOCAnalyst,Export: ⏱️ STEP 3: AGGREGATE DEFENSE DETECTION METRICS
    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════

    par Calculate Detection Metrics

        par VirusTotal Detection

            Queue->>DB: SELECT COUNT(DISTINCT url) FILTER (WHERE vt_verdict != 'CLEAN')<br/>FROM vt_analyses WHERE campaign_id = 1001

            DB-->>Queue: vt_flagged_urls = 8

            Queue->>DB: SELECT COUNT(DISTINCT url) FROM vt_analyses WHERE campaign_id = 1001

            DB-->>Queue: total_urls = 30

            Queue->>Queue: VIRUSTOTAL METRICS<br/>├─ urls_tested: 30<br/>├─ urls_flagged: 8<br/>├─ vt_detection_rate: (8 / 30) × 100 = 26.7%<br/>├─ vt_verdict_breakdown: {CLEAN: 22, SUSPICIOUS: 7, MALICIOUS: 1}<br/>└─ vt_risk_scores: [0.002, 0.008, ..., 0.18]

        and SIEM Detection Rate

            Queue->>DB: SELECT COUNT(*) FILTER (WHERE email_detected_by_siem = true)<br/>FROM campaign_events WHERE campaign_id = 1001

            DB-->>Queue: siem_detected_count = 820

            Queue->>Queue: SIEM DETECTION METRICS<br/>├─ emails_detected_by_siem: 820<br/>├─ siem_detection_rate: (820 / 1485) × 100 = 55.2%<br/>├─ rules_triggered: [rule_domain_spoof, rule_urgency_url, ...]<br/>├─ unique_emails_detected: 820 (no overlap counted)<br/>└─ detection_confidence: avg 0.87

        and False Negatives Analysis

            Queue->>DB: SELECT COUNT(*) FILTER (WHERE event_type = 'clicked')<br/>FROM campaign_events WHERE campaign_id = 1001<br/>AND email_detected_by_siem = false

            DB-->>Queue: fn_clicked_count = 540

            Queue->>DB: SELECT COUNT(*) FILTER (WHERE event_type = 'submitted')<br/>FROM campaign_events WHERE campaign_id = 1001<br/>AND email_detected_by_siem = false

            DB-->>Queue: fn_submitted_count = 135

            Queue->>Queue: FALSE NEGATIVE METRICS<br/>├─ fn_clicked: 540 (emails clicked despite defenses)<br/>├─ fn_submitted: 135 (credentials submitted despite defenses)<br/>├─ fn_total: 540 (clicked is superset of submitted)<br/>├─ fn_rate_on_sent: (540 / 1500) × 100 = 36%<br/>├─ fn_rate_on_clicked: (540 / 540) × 100 = 100%<br/>└─ severity: CRITICAL (defenses missed clickers)

        and Overall Detection Rate

            Queue->>Queue: OVERALL DETECTION CALCULATION<br/>├─ Items tested: 1500 (emails sent)<br/>├─ Items detected by any defense: 1110<br/>│  ├─ SIEM detected: 820<br/>│  ├─ VT detected (landing page flagged): 300*<br/>│  └─ Overlap (counted once): 10<br/>├─ detection_rate: (1110 / 1500) × 100 = 74%<br/>├─ false_negative_count: 390<br/>└─ coverage_gap: 26%

        and Detection by Variant

            Queue->>DB: SELECT variant_number, COUNT(*) as variant_sent,<br/>SUM(CASE WHEN detected = true THEN 1 ELSE 0 END) as variant_detected<br/>FROM campaign_events GROUP BY variant_number

            DB-->>Queue: [<br/>  {variant: 1, sent: 50, detected: 37, rate: 74%},<br/>  {variant: 2, sent: 50, detected: 38, rate: 76%},<br/>  ...<br/>  {variant: 30, sent: 50, detected: 36, rate: 72%}<br/>]

            Queue->>Queue: VARIANT DETECTION ANALYSIS<br/>├─ Best variant: #2 (76% detection)<br/>├─ Worst variant: #15 (68% detection)<br/>├─ Avg detection by variant: 74%<br/>├─ Std dev: 2.1%<br/>└─ Variance: Low (consistent across variants)

    end

    Queue->>MetricsEngine: INSERT defense_metrics {<br/>  campaign_id: 1001,<br/>  vt_detection_rate: 0.267,<br/>  vt_urls_tested: 30,<br/>  vt_urls_flagged: 8,<br/>  siem_detection_rate: 0.552,<br/>  siem_emails_detected: 820,<br/>  overall_detection_rate: 0.74,<br/>  false_negative_count: 390,<br/>  false_negative_rate: 0.26,<br/>  detection_by_variant: [...],<br/>  avg_detection_confidence: 0.87<br/>}
    MetricsEngine-->>Queue: ✅ Stored

    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════
    Note over SOCAnalyst,Export: ⏱️ STEP 4: CALCULATE SIGMA RULE METRICS
    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════

    par Calculate Rule Metrics

        Queue->>DB: SELECT COUNT(*) FROM sigma_rules<br/>WHERE campaign_id = 1001

        DB-->>Queue: rules_generated = 4

        Queue->>DB: SELECT COUNT(*) FROM sigma_rules<br/>WHERE campaign_id = 1001 AND approval_status = 'approved'

        DB-->>Queue: rules_approved = 4

        Queue->>Queue: RULE GENERATION METRICS<br/>├─ rules_generated: 4<br/>├─ rules_validated: 4<br/>├─ rules_approved: 4<br/>├─ approval_rate: (4 / 4) × 100 = 100%<br/>└─ deployment_pending: 4

        par Rule Performance Analysis (30-day post-deployment)

            Queue->>DB: SELECT rule_id,<br/>SUM(CASE WHEN alert_type = 'tp' THEN 1 ELSE 0 END) as tp_count,<br/>SUM(CASE WHEN alert_type = 'fp' THEN 1 ELSE 0 END) as fp_count<br/>FROM rule_alerts WHERE rule_id IN (1001_1, 1001_2, 1001_3, 1001_4)<br/>AND alert_timestamp >= NOW() - interval '30 days'

            DB-->>Queue: [<br/>  {rule_id: 1001_1, tp: 4, fp: 1, total: 5},<br/>  {rule_id: 1001_2, tp: 11, fp: 1, total: 12},<br/>  {rule_id: 1001_3, tp: 7, fp: 1, total: 8},<br/>  {rule_id: 1001_4, tp: 2, fp: 1, total: 3}<br/>]

            Queue->>Queue: RULE PERFORMANCE (30-day history)<br/>├─ Rule 1 (Domain Spoof): TP: 4 | FP: 1 | TP Rate: 80% | FP Rate: 20% | Status: Needs Tuning<br/>├─ Rule 2 (Urgency+URL): TP: 11 | FP: 1 | TP Rate: 91.7% | FP Rate: 8.3% | Status: Good<br/>├─ Rule 3 (Headers): TP: 7 | FP: 1 | TP Rate: 87.5% | FP Rate: 12.5% | Status: Good<br/>└─ Rule 4 (AI-Content): TP: 2 | FP: 1 | TP Rate: 66.7% | FP Rate: 33.3% | Status: Needs Tuning<br/>Overall Average:<br/>├─ Avg TP Rate: 81.5%<br/>├─ Avg FP Rate: 13.5%<br/>└─ Avg Accuracy: 81.5%

        and Coverage Analysis

            Queue->>Queue: FALSE NEGATIVE COVERAGE BY RULES<br/>├─ Rule 1 coverage: 145 FN detected<br/>├─ Rule 2 coverage: 120 FN detected<br/>├─ Rule 3 coverage: 85 FN detected<br/>├─ Rule 4 coverage: 40 FN detected<br/>├─ Total FN covered: 390 (100% - if all rules deployed)<br/>├─ Overlapping detections: ~10 FN<br/>└─ Net unique coverage: 380 FN

        end

    end

    Queue->>MetricsEngine: INSERT rule_metrics {<br/>  campaign_id: 1001,<br/>  rules_generated: 4,<br/>  rules_approved: 4,<br/>  rules_deployed: 0,<br/>  deployment_pending: 4,<br/>  rule_performance: [<br/>    {rule_id: 1001_1, tp_rate: 0.80, fp_rate: 0.20},<br/>    {rule_id: 1001_2, tp_rate: 0.917, fp_rate: 0.083},<br/>    {rule_id: 1001_3, tp_rate: 0.875, fp_rate: 0.125},<br/>    {rule_id: 1001_4, tp_rate: 0.667, fp_rate: 0.333}<br/>  ],<br/>  avg_tp_rate: 0.815,<br/>  avg_fp_rate: 0.135,<br/>  total_fn_coverage: 390<br/>}
    MetricsEngine-->>Queue: ✅ Stored

    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════
    Note over SOCAnalyst,Export: ⏱️ STEP 5: CALCULATE USER RISK METRICS
    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════

    par Calculate User Risk Metrics

        Queue->>DB: SELECT user_email,<br/>SUM(CASE WHEN event_type = 'clicked' THEN 1 ELSE 0 END) as click_count,<br/>SUM(CASE WHEN event_type = 'submitted' THEN 1 ELSE 0 END) as submit_count,<br/>SUM(CASE WHEN event_type = 'reported' THEN 1 ELSE 0 END) as report_count,<br/>SUM(1) as email_count<br/>FROM campaign_events<br/>WHERE campaign_id = 1001<br/>GROUP BY user_email<br/>ORDER BY click_count DESC<br/>LIMIT 10

        DB-->>Queue: [<br/>  {email: "john.doe@acme.com", clicks: 2, submits: 1, reports: 0, total: 2, click_rate: 1.0},<br/>  {email: "jane.smith@acme.com", clicks: 1, submits: 1, reports: 0, total: 1, click_rate: 1.0},<br/>  {email: "bob.jones@acme.com", clicks: 1, submits: 0, reports: 0, total: 1, click_rate: 1.0},<br/>  {email: "alice.clark@acme.com", clicks: 1, submits: 0, reports: 0, total: 1, click_rate: 1.0},<br/>  {email: "charlie.brown@acme.com", clicks: 3, submits: 1, reports: 0, total: 5, click_rate: 0.6},<br/>  ... (5 more)<br/>]

        Queue->>Queue: TOP 10 RISKY USERS<br/>1. john.doe@acme.com: 100% click rate, 50% submit rate 🔴<br/>2. jane.smith@acme.com: 100% click rate, 100% submit rate 🔴<br/>3. bob.jones@acme.com: 100% click rate 🔴<br/>4. alice.clark@acme.com: 100% click rate 🔴<br/>5. charlie.brown@acme.com: 60% click rate, 20% submit rate 🟠<br/>... and 5 more

        and Calculate Department Risk

            Queue->>DB: SELECT u.department,<br/>COUNT(DISTINCT ce.user_email) as total_users,<br/>SUM(CASE WHEN ce.event_type = 'clicked' THEN 1 ELSE 0 END) / COUNT(DISTINCT ce.user_email) as dept_click_rate<br/>FROM campaign_events ce<br/>JOIN users u ON ce.user_email = u.email<br/>WHERE ce.campaign_id = 1001<br/>GROUP BY u.department<br/>ORDER BY dept_click_rate DESC

            DB-->>Queue: [<br/>  {dept: "Finance", users: 31, clicks: 18, rate: 0.58},<br/>  {dept: "HR", users: 25, clicks: 13, rate: 0.52},<br/>  {dept: "Sales", users: 40, clicks: 19, rate: 0.48},<br/>  {dept: "IT", users: 22, clicks: 5, rate: 0.23},<br/>  {dept: "Executive", users: 12, clicks: 2, rate: 0.17}<br/>]

            Queue->>Queue: TOP RISKY DEPARTMENTS (Phish-Prone %)<br/>1. Finance: 58% 🔴 (18/31 users clicked)<br/>2. HR: 52% 🟠 (13/25 users clicked)<br/>3. Sales: 48% 🟡 (19/40 users clicked)<br/>4. IT: 23% 🟢 (5/22 users clicked)<br/>5. Executive: 17% 🟢 (2/12 users clicked)

        and Calculate Time-to-Click Metrics

            Queue->>DB: SELECT PERCENTILE_CONT(0.5) WITHIN GROUP<br/>(ORDER BY EXTRACT(EPOCH FROM timestamp - sent_at)) as median_ttc<br/>FROM campaign_events<br/>WHERE campaign_id = 1001 AND event_type = 'clicked'

            DB-->>Queue: median_ttc_seconds = 135

            Queue->>Queue: TIME-TO-INTERACTION METRICS<br/>├─ Median time-to-first-click: 2min 15sec (135 seconds)<br/>├─ Median time-to-first-submission: 4min 30sec (270 seconds)<br/>├─ Mean time-to-report: 12min 45sec (765 seconds)<br/>├─ Click velocity (clicks in first 5 min): 65%<br/>└─ Submission velocity (submissions in first 10 min): 82%

    end

    Queue->>MetricsEngine: INSERT user_risk_metrics {<br/>  campaign_id: 1001,<br/>  top_risky_users: [<br/>    {email: "john.doe@acme.com", click_rate: 1.0, risk_level: "critical"},<br/>    {email: "jane.smith@acme.com", click_rate: 1.0, risk_level: "critical"},<br/>    ...<br/>  ],<br/>  top_risky_departments: [<br/>    {dept: "Finance", click_rate: 0.58, risk_level: "high"},<br/>    {dept: "HR", click_rate: 0.52, risk_level: "high"},<br/>    ...<br/>  ],<br/>  median_ttc_seconds: 135,<br/>  median_ttc_readable: "2min 15sec"<br/>}
    MetricsEngine-->>Queue: ✅ Stored

    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════
    Note over SOCAnalyst,Export: ⏱️ STEP 6: CALCULATE PLATFORM OPERATIONAL METRICS
    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════

    par Calculate Operational Metrics

        Queue->>DB: SELECT<br/>AVG(api_latency_ms) as avg_latency,<br/>MAX(api_latency_ms) as max_latency,<br/>COUNT(*) FILTER (WHERE error = true) as error_count,<br/>COUNT(*) as total_calls<br/>FROM api_logs<br/>WHERE campaign_id = 1001

        DB-->>Queue: {<br/>  gophish_avg_latency: 125,<br/>  vt_avg_latency: 450,<br/>  siem_avg_latency: 280,<br/>  total_api_calls: 600,<br/>  error_count: 2<br/>}

        Queue->>Queue: API PERFORMANCE METRICS<br/>├─ GoPhish API avg latency: 125 ms ✅<br/>├─ VirusTotal API avg latency: 450 ms ✅<br/>├─ SIEM API avg latency: 280 ms ✅<br/>├─ API error rate: (2 / 600) × 100 = 0.3% ✅<br/>└─ Overall uptime: 99.7%

        and Calculate Campaign Execution Timing

            Queue->>DB: SELECT<br/>campaign_created_at,<br/>gophish_launched_at,<br/>campaign_completed_at<br/>FROM campaigns WHERE campaign_id = 1001

            DB-->>Queue: {<br/>  created: "2025-11-20T12:00:00Z",<br/>  launched: "2025-11-20T14:00:00Z",<br/>  completed: "2025-11-20T16:00:00Z"<br/>}

            Queue->>Queue: CAMPAIGN EXECUTION TIMING<br/>├─ LLM generation time: 2.3 minutes<br/>├─ GoPhish creation time: 0.5 minutes<br/>├─ VirusTotal scanning time: 2 minutes (35 URLs)<br/>├─ Rule generation time: 3.1 minutes<br/>├─ Total setup time: 6.4 minutes<br/>├─ Campaign duration: 120 minutes (sending)<br/>└─ Total end-to-end: 126.4 minutes

        and Calculate Resource Utilization

            Queue->>DB: SELECT<br/>queue_length,<br/>avg_job_duration_ms,<br/>cache_hit_rate,<br/>db_query_avg_time_ms<br/>FROM system_metrics<br/>WHERE timestamp >= NOW() - interval '1 hour'

            DB-->>Queue: {<br/>  queue_length: 5,<br/>  avg_job_duration: 2500,<br/>  cache_hit_rate: 0.82,<br/>  db_query_avg_time: 45<br/>}

            Queue->>Queue: RESOURCE METRICS<br/>├─ Job queue length: 5 jobs (healthy)<br/>├─ Avg job duration: 2.5 seconds<br/>├─ Cache hit rate: 82% (excellent)<br/>├─ DB query avg time: 45 ms<br/>├─ LLM tokens used: 12,500 / 100,000 (monthly quota)<br/>└─ Storage used: 2.3 GB / 10 GB allocated

    end

    Queue->>MetricsEngine: INSERT operational_metrics {<br/>  campaign_id: 1001,<br/>  api_latency_avg: {gophish: 125, vt: 450, siem: 280},<br/>  api_error_rate: 0.003,<br/>  uptime_percent: 0.997,<br/>  campaign_setup_time_minutes: 6.4,<br/>  campaign_execution_time_minutes: 120,<br/>  queue_length: 5,<br/>  cache_hit_rate: 0.82,<br/>  db_query_avg_ms: 45<br/>}
    MetricsEngine-->>Queue: ✅ Stored

    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════
    Note over SOCAnalyst,Export: ⏱️ STEP 7: THRESHOLD-BASED ALERTING
    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════

    Queue->>AlertSys: TRIGGER THRESHOLD EVALUATION

    AlertSys->>AlertSys: LOAD THRESHOLD CONFIGURATION<br/>From organization settings (Acme Corp):<br/>├─ Alert 1: click_rate > 15% → HIGH_RISK<br/>├─ Alert 2: vt_detection_rate >= 10% → BLOCK<br/>├─ Alert 3: report_rate < 1% AND click_rate > 5% → TRAINING_REQUIRED<br/>├─ Alert 4: siem_detection_rate < 50% → DEFENSE_WEAKNESS<br/>├─ Alert 5: fn_rate > 30% → COVERAGE_GAP<br/>└─ Alert 6: phish_prone_percent > 40% → HIGH_ORGANIZATIONAL_RISK

    AlertSys->>AlertSys: EVALUATE ALERT 1: Click Rate > 15%<br/>Threshold: 0.15<br/>Actual: 0.364 (36.4%)<br/>Result: 0.364 > 0.15 ✅ TRIGGERED<br/>Severity: HIGH<br/>Action: Send alert to SOC team

    AlertSys->>AlertSys: EVALUATE ALERT 2: VT Detection Rate >= 10%<br/>Threshold: 0.10<br/>Actual: 0.267 (26.7%)<br/>Result: 0.267 >= 0.10 ✅ TRIGGERED<br/>Severity: CRITICAL<br/>Action: Block campaign + alert admin

    AlertSys->>AlertSys: EVALUATE ALERT 3: Report Rate < 1% AND Click Rate > 5%<br/>Threshold: report_rate < 0.01 AND click_rate > 0.05<br/>Actual: 0.012 (1.2%) AND 0.364 (36.4%)<br/>Result: (0.012 > 0.01) = false, condition NOT met ❌ NOT TRIGGERED<br/>Note: Report rate is actually ABOVE 1%, so no training alert needed

    AlertSys->>AlertSys: EVALUATE ALERT 4: SIEM Detection Rate < 50%<br/>Threshold: 0.50<br/>Actual: 0.552 (55.2%)<br/>Result: 0.552 >= 0.50 ❌ NOT TRIGGERED<br/>Action: None (detection rate is above threshold)

    AlertSys->>AlertSys: EVALUATE ALERT 5: False Negative Rate > 30%<br/>Threshold: 0.30<br/>Actual: 0.26 (26%)<br/>Result: 0.26 <= 0.30 ❌ NOT TRIGGERED<br/>Note: False negative rate is below threshold, acceptable

    AlertSys->>AlertSys: EVALUATE ALERT 6: Phish-Prone % > 40%<br/>Threshold: 0.40<br/>Actual: 0.51 (51%)<br/>Result: 0.51 > 0.40 ✅ TRIGGERED<br/>Severity: HIGH<br/>Action: Flag for immediate training deployment

    AlertSys-->>AlertSys: ALERT SUMMARY<br/>┌─────────────────────────────────────┐<br/>│ 🔴 CRITICAL: 1 alert                │<br/>│   - VT Detection: 26.7% >= 10%      │<br/>├─────────────────────────────────────┤<br/>│ 🟡 HIGH: 2 alerts                   │<br/>│   - Click Rate: 36.4% > 15%         │<br/>│   - Phish-Prone: 51% > 40%          │<br/>├─────────────────────────────────────┤<br/>│ ✅ OK: 3 alerts (not triggered)     │<br/>└─────────────────────────────────────┘

    AlertSys->>DB: INSERT threshold_alerts {<br/>  campaign_id: 1001,<br/>  alerts: [<br/>    {type: "HIGH_CLICK_RATE", severity: "high", threshold: 0.15, actual: 0.364, status: "triggered"},<br/>    {type: "VT_DETECTION", severity: "critical", threshold: 0.10, actual: 0.267, status: "triggered"},<br/>    {type: "HIGH_PHISH_PRONE", severity: "high", threshold: 0.40, actual: 0.51, status: "triggered"},<br/>    {type: "TRAINING_REQUIRED", severity: "high", threshold: "0.01 AND 0.05", status: "not_triggered"},<br/>    {type: "DEFENSE_WEAKNESS", severity: "medium", threshold: 0.50, actual: 0.552, status: "not_triggered"},<br/>    {type: "COVERAGE_GAP", severity: "medium", threshold: 0.30, actual: 0.26, status: "not_triggered"}<br/>  ],<br/>  total_alerts_triggered: 3,<br/>  alert_evaluation_timestamp: NOW()<br/>}
    DB-->>AlertSys: ✅ Stored

    AlertSys->>AlertSys: SEND NOTIFICATIONS<br/>Email to: soc-team@acme.com<br/>Slack: #phistrek-alerts<br/>Subject: "[PHISTREK ALERT] Campaign 1001: 3 thresholds triggered"<br/>Body:<br/>  🔴 CRITICAL: VirusTotal detected 26.7% of URLs<br/>  🟡 HIGH: Click rate 36.4% exceeds target 15%<br/>  🟡 HIGH: 51% of users phish-prone (target: < 40%)<br/>  ACTION: Review campaign impact and recommend immediate training deployment

    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════
    Note over SOCAnalyst,Export: ⏱️ STEP 8: CACHE & WEBSOCKET PUSH
    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════

    Queue->>Cache: SET campaign:1001:dashboard_data {<br/>  campaign_id: 1001,<br/>  engagement_metrics: {<br/>    sent: 1500, delivered: 1485, opened: 960, clicked: 540, submitted: 135,<br/>    open_rate: 0.646, click_rate: 0.364, submission_rate: 0.091, report_rate: 0.012,<br/>    phish_prone_percent: 0.51<br/>  },<br/>  defense_metrics: {<br/>    vt_detection_rate: 0.267, siem_detection_rate: 0.552, overall_detection_rate: 0.74,<br/>    false_negative_count: 390, false_negative_rate: 0.26<br/>  },<br/>  rule_metrics: {<br/>    rules_generated: 4, rules_approved: 4, avg_tp_rate: 0.815, avg_fp_rate: 0.135<br/>  },<br/>  user_risk_metrics: {<br/>    top_risky_users: [...], top_risky_departments: [...],<br/>    phish_prone_trend: [...30-day data...}<br/>  },<br/>  operational_metrics: {<br/>    api_latency_avg: {...}, uptime: 0.997, cache_hit_rate: 0.82<br/>  },<br/>  alerts: {<br/>    triggered: 3, critical: 1, high: 2<br/>  },<br/>  cached_at: NOW(),<br/>  ttl: 300<br/>}

    Cache-->>Queue: ✅ Dashboard data cached (5 min TTL)

    Queue->>WebSocket: PUBLISH dashboard_update {<br/>  campaign_id: 1001,<br/>  event_type: "metrics_updated",<br/>  timestamp: NOW(),<br/>  data: {...complete KPI data...},<br/>  alerts_triggered: 3<br/>}

    WebSocket-->>Web: Broadcast to all connected clients:<br/>topic: "campaign/1001/dashboard"

    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════
    Note over SOCAnalyst,Export: ⏱️ STEP 9: REAL-TIME DASHBOARD DISPLAY & RENDERING
    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════

    Web->>Web: Receive WebSocket event: metrics_updated

    Web->>Web: UPDATE REACT STATE<br/>├─ engagement_metrics<br/>├─ defense_metrics<br/>├─ rule_metrics<br/>├─ user_risk_metrics<br/>├─ alerts<br/>└─ operational_metrics

    Web->>Web: RE-RENDER DASHBOARD COMPONENTS

    Web-->>SOCAnalyst: ✅ PHISTREK CAMPAIGN DASHBOARD<br/>═══════════════════════════════════════════════════════<br/><br/>📋 EXECUTIVE SUMMARY (Top Cards)<br/>┌──────────────────────────────────────────────────────┐<br/>│                 CAMPAIGN 1001 OVERVIEW                │<br/>├──────────────────────────────────────────────────────┤<br/>│                                                       │<br/>│  Campaign: Acme HR Q4 Test        Status: ✅ Complete│<br/>│  Launch: 2025-11-20 14:00:00     Duration: 120 min   │<br/>│                                                       │<br/>│  ┌──────────┬──────────┬──────────┬──────────┐       │<br/>│  │ SENT     │ DELIVERED│ OPENED   │ CLICKED  │       │<br/>│  │ 1500     │ 1485(99%)│ 960(65%) │ 540(36%) │       │<br/>│  └──────────┴──────────┴──────────┴──────────┘       │<br/>│                                                       │<br/>│  ┌──────────┬──────────┬──────────┬──────────┐       │<br/>│  │ SUBMITTED│ REPORTED │ VT DET   │ SIEM DET │       │<br/>│  │ 135(9%)  │ 18(1.2%) │ 26.7%    │ 55.2%    │       │<br/>│  └──────────┴──────────┴──────────┴──────────┘       │<br/>│                                                       │<br/>│  ⚠️ PHISH-PRONE: 51% | ❌ FN RATE: 26%              │<br/>│  📝 RULES GENERATED: 4 | 🚀 READY FOR DEPLOY        │<br/>│                                                       │<br/>└──────────────────────────────────────────────────────┘<br/><br/>🔴 ACTIVE ALERTS<br/>┌──────────────────────────────────────────────────────┐<br/>│ 🔴 CRITICAL: VT Detection 26.7% >= 10% threshold    │<br/>│    Recommendation: Review landing page safety        │<br/>│                                                       │<br/>│ 🟡 HIGH: Click Rate 36.4% > 15% threshold           │<br/>│    Recommendation: Deploy security awareness training│<br/>│                                                       │<br/>│ 🟡 HIGH: Phish-Prone 51% > 40% target               │<br/>│    Recommendation: Focus on Finance (58%) & HR (52%) │<br/>└──────────────────────────────────────────────────────┘<br/><br/>📊 ENGAGEMENT FUNNEL<br/>┌──────────────────────────────────────────────────────┐<br/>│                                                       │<br/>│  SENT (1500)                                          │<br/>│    │                                                  │<br/>│    ├── 99% DELIVERY ──→ DELIVERED (1485)             │<br/>│         │                                           │<br/>│         ├── 65% OPEN ──→ OPENED (960)               │<br/>│              │                                      │<br/>│              ├── 56% CLICK ──→ CLICKED (540)        │<br/>│                   │                                 │<br/>│                   └── 25% SUBMIT ──→ SUBMITTED (135)│<br/>│                                                       │<br/>└──────────────────────────────────────────────────────┘<br/><br/>🎯 DETECTION PERFORMANCE<br/>┌──────────────────────────────────────────────────────┐<br/>│ Detection Rate Breakdown:                            │<br/>│                                                       │<br/>│ ▓▓▓▓▓▓▓▓▓▓░░░░░░░░░░  74% Overall Detected          │<br/>│                    (1110 / 1500)                     │<br/>│                                                       │<br/>│ SIEM Detection:     ▓▓▓▓▓▓░░░░░  55.2% (820 emails)│<br/>│ VT Detection:       ▓▓░░░░░░░░░░░  26.7% (8/30 URLs)│<br/>│ Combined Detection: ▓▓▓▓▓▓▓░░░░░░░░  74% (1110 total)│<br/>│                                                       │<br/>│ False Negatives:    ▓▓░░░░░░░░░  26% (390 emails)   │<br/>│                                                       │<br/>└──────────────────────────────────────────────────────┘<br/><br/>👥 USER RISK HEATMAP<br/>┌──────────────────────────────────────────────────────┐<br/>│ Top Risky Users:                 Top Risky Depts:     │<br/>│                                                       │<br/>│ 1. john.doe@acme.com    100% 🔴  Finance: 58% 🔴    │<br/>│ 2. jane.smith@acme.com  100% 🔴  HR: 52% 🟠         │<br/>│ 3. bob.jones@acme.com   100% 🔴  Sales: 48% 🟡      │<br/>│ 4. alice.clark@acme.com  80% 🟠  IT: 23% 🟢         │<br/>│ 5. charlie.brown@acme   60% 🟡  Exec: 17% 🟢        │<br/>│                                                       │<br/>│ Time to Click: 2min 15sec (median)                   │<br/>│ Time to Submit: 4min 30sec (median)                  │<br/>│                                                       │<br/>└──────────────────────────────────────────────────────┘<br/><br/>📝 SIGMA RULES GENERATED<br/>┌──────────────────────────────────────────────────────┐<br/>│ Rule 1: Domain Spoofing                               │<br/>│  Status: ✅ Approved | Confidence: 95% | Coverage: 145│<br/>│  FP Risk: 5%                                         │<br/>│                                                       │<br/>│ Rule 2: Urgency + URL Shorteners                    │<br/>│  Status: ✅ Approved | Confidence: 92% | Coverage: 120│<br/>│  FP Risk: 8%                                         │<br/>│                                                       │<br/>│ Rule 3: Header Manipulation                          │<br/>│  Status: ✅ Approved | Confidence: 88% | Coverage: 85 │<br/>│  FP Risk: 12%                                        │<br/>│                                                       │<br/>│ Rule 4: AI-Generated Content                         │<br/>│  Status: ✅ Approved | Confidence: 78% | Coverage: 40 │<br/>│  FP Risk: 15%                                        │<br/>│                                                       │<br/>│ 🚀 [Deploy to SIEM] [Download YAML] [View Details]   │<br/>│                                                       │<br/>└──────────────────────────────────────────────────────┘<br/><br/>📈 30-DAY TREND ANALYSIS<br/>┌──────────────────────────────────────────────────────┐<br/>│ Phish-Prone % Trend (30d):                           │<br/>│ 30% │                                                │<br/>│ 25% │    ╱╲        ╱                                  │<br/>│ 20% │   ╱  ╲      ╱╲                                 │<br/>│ 15% │  ╱    ╲    ╱  ╲        ╱╲                       │<br/>│ 10% │ ╱      ╲  ╱    ╲      ╱  ╲                      │<br/>│      │╱────────╲╱──────╲────╱────╲── 26% (Today)    │<br/>│      └────────────────────────────────               │<br/>│      Oct 20   Oct 25  Nov 10  Nov 20                 │<br/>│ Trend: Fluctuating, avg 25% → current 26% (↑ 1%)   │<br/>│ Recommendation: Continue monitoring after training   │<br/>│                                                       │<br/>└──────────────────────────────────────────────────────┘<br/><br/>🔧 OPERATIONAL METRICS<br/>┌──────────────────────────────────────────────────────┐<br/>│ API Performance:       System Health:                 │<br/>│ ├─ GoPhish: 125ms      ├─ Uptime: 99.7%             │<br/>│ ├─ VT: 450ms           ├─ Queue: Healthy (5 jobs)   │<br/>│ ├─ SIEM: 280ms         ├─ Cache: 82% hit rate       │<br/>│ └─ Error Rate: 0.3%    └─ DB Query: 45ms avg        │<br/>│                                                       │<br/>│ Setup Time: 6.4 min | Campaign Time: 120 min        │<br/>│                                                       │<br/>└──────────────────────────────────────────────────────┘<br/><br/>💾 ACTIONS<br/>┌──────────────────────────────────────────────────────┐<br/>│ [Block Campaign] [Deploy Rules] [Assign Training]    │<br/>│ [Export Report] [View Details] [Share Results]       │<br/>└──────────────────────────────────────────────────────┘

    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════
    Note over SOCAnalyst,Export: ⏱️ STEP 10: INTERACTIVE DASHBOARD ACTIONS
    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════

    par User Action 1: Block Campaign

        SOCAnalyst->>Web: Click "Block Campaign" button
        Web->>Web: Show confirmation modal

        SOCAnalyst->>Web: Confirm block with reason: "High click rate and VT detection"

        Web->>API: POST /campaigns/1001/block {<br/>  reason: "High click rate (36.4%) and VT detection (26.7%)",<br/>  blocked_by: SOC_ANALYST_ID<br/>}

        API->>GoPhish: DELETE /api/campaigns/5001..5030<br/>(Delete all 30 campaign variants from GoPhish)

        GoPhish-->>API: ✅ All campaigns deleted

        API->>DB: UPDATE campaigns SET {<br/>  status: "blocked",<br/>  blocked_at: NOW(),<br/>  block_reason: "User initiated - high risk metrics"<br/>}
        DB-->>API: ✅ Updated

        API-->>Web: {status: "blocked", message: "Campaign 1001 blocked"}
        Web-->>SOCAnalyst: ✅ "Campaign blocked successfully"

    and User Action 2: Deploy Rules to SIEM

        SOCAnalyst->>Web: Click "Deploy to SIEM" button
        Web->>Web: Show SIEM connection dialog

        SOCAnalyst->>Web: Select: Splunk endpoint<br/>Authentication: Token provided

        Web->>API: POST /campaigns/1001/rules/deploy {<br/>  target: "splunk",<br/>  endpoint: "...",<br/>  rules: [1001_1, 1001_2, 1001_3, 1001_4]<br/>}

        API->>API: Create 4 Splunk saved searches via REST
        API-->>Web: {status: "deployed", rules_deployed: 4}

        Web-->>SOCAnalyst: ✅ "4 rules deployed to Splunk"

    and User Action 3: Assign Training

        SOCAnalyst->>Web: Click "Assign Training" button
        Web->>Web: Show training assignment form

        SOCAnalyst->>Web: Select: {<br/>  target: "Department",<br/>  department: "Finance",<br/>  module: "Phishing Awareness 101",<br/>  deadline: "2025-11-27"<br/>}

        Web->>API: POST /training/assign {<br/>  target_type: "department",<br/>  target_id: "finance",<br/>  training_module_id: "phishing-101",<br/>  deadline: "2025-11-27",<br/>  notify: true<br/>}

        API->>DB: INSERT training_assignments {...}
        API-->>Web: {status: "assigned", users_notified: 31}

        Web-->>SOCAnalyst: ✅ "Training assigned to 31 Finance staff"

    and User Action 4: Export Report

        SOCAnalyst->>Web: Click "Export Report"
        Web->>Web: Show export options

        SOCAnalyst->>Web: Select format: "PDF"<br/>Include: All sections

        Web->>API: POST /reports/generate {<br/>  campaign_id: 1001,<br/>  format: "pdf",<br/>  sections: ["summary", "metrics", "alerts", "rules", "users"]<br/>}

        API->>Export: Generate comprehensive PDF
        Export->>Export: Build multi-page report<br/>Page 1: Executive Summary<br/>Pages 2-3: KPI metrics + charts<br/>Pages 4-5: Alert analysis<br/>Pages 6-7: Rules overview<br/>Pages 8-9: User risk analysis

        Export-->>API: phistrek_report_1001.pdf

        API-->>Web: {<br/>  file_url: "https://...",<br/>  file_size: "2.3 MB"<br/>}

        Web-->>SOCAnalyst: ✅ "Report ready"<br/>📥 [Download PDF]

    end

    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════
    Note over SOCAnalyst,Export: ⏱️ STEP 11: 30-DAY TREND ANALYSIS & HISTORICAL TRACKING
    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════

    SOCAnalyst->>Web: Click "Trends" tab
    Web->>API: GET /dashboard/trends?time_range=30d&metrics=click_rate,phish_prone,detection

    API->>Cache: GET dashboard:trends:30d

    alt Cache Hit
        Cache-->>API: {trend_data: [...]}
    else Cache Miss
        API->>DB: SELECT DATE_TRUNC('day', created_at) as day,<br/>AVG(click_rate) as avg_click,<br/>AVG(phish_prone_percent) as avg_phish_prone,<br/>AVG(overall_detection_rate) as avg_detection<br/>FROM campaigns<br/>WHERE created_at >= NOW() - interval '30 days'<br/>GROUP BY day ORDER BY day

        DB-->>API: [<br/>  {day: "2025-10-21", click_rate: 0.22, phish_prone: 0.18, detection: 0.68},<br/>  {day: "2025-10-22", click_rate: 0.25, phish_prone: 0.20, detection: 0.71},<br/>  ...<br/>  {day: "2025-11-20", click_rate: 0.28, phish_prone: 0.26, detection: 0.74}<br/>]

    end

    API-->>Web: Trend data

    Web-->>SOCAnalyst: 📈 30-DAY TREND ANALYSIS<br/><br/>Click Rate Trend (↑ +6% over 30 days)<br/>▁▂▂▃▃▄▄▅▅▆▆▇▇██████ 28% (today)<br/>Oct 21                Nov 20<br/>Baseline: 22% → Current: 28%<br/>Change: +6% (negative trend - needs improvement)<br/><br/>Phish-Prone % Trend (↑ +8% over 30 days)<br/>▁▂▂▃▃▃▄▅▅▆▆▇███████ 26% (today)<br/>Trend: Increasing (requires training intervention)<br/><br/>Detection Rate Trend (↑ +6% improvement)<br/>████████████████░░░ 74% (today)<br/>Baseline: 68% → Current: 74%<br/>Change: +6% (positive trend - rules helping)<br/><br/>Recommendations:<br/>├─ Continue Sigma rule deployment<br/>├─ Increase training frequency<br/>├─ Focus on Finance & HR departments<br/>└─ Retest after training in 14 days

    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════
    Note over SOCAnalyst,Export: ⏱️ STEP 12: CONTINUOUS IMPROVEMENT CYCLE
    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════

    Queue->>Queue: NIGHTLY AGGREGATION JOB (Background)

    Queue->>DB: AGGREGATE 30-DAY ROLLING METRICS<br/>For each day in last 30 days:<br/>1. Sum all campaign events<br/>2. Calculate daily KPIs<br/>3. Compute rolling averages<br/>4. Identify trend direction<br/>5. Check threshold violations<br/>6. Generate recommendations

    Queue->>DB: INSERT daily_metrics {<br/>  date: "2025-11-20",<br/>  click_rate_avg: 0.28,<br/>  detection_rate_avg: 0.74,<br/>  phish_prone_avg: 0.26,<br/>  top_risky_dept: "Finance",<br/>  rules_deployed_today: 4,<br/>  alerts_triggered_today: 3,<br/>  training_assigned_today: 31,<br/>  improvement_vs_baseline: "+6%"<br/>}
    DB-->>Queue: ✅ Stored

    Queue->>Queue: CALCULATE IMPROVEMENT METRICS<br/>30-day avg click rate: 25%<br/>Baseline (90 days ago): 32%<br/>Improvement: (32% - 25%) / 32% ≈ 22% reduction ✅<br/><br/>Detection rate improvement:<br/>Before Sigma rules: 68%<br/>After Sigma rules: 74%<br/>Rules impact: +6% detection ✅<br/><br/>User awareness improvement:<br/>Training completed: 8 users<br/>Post-training click rate: 15% (vs 28% pre-training)<br/>Training effectiveness: 46% reduction ✅

    Queue->>DB: INSERT improvement_metrics {<br/>  date: "2025-11-20",<br/>  click_rate_improvement_30d: 0.22,<br/>  detection_improvement_with_rules: 0.06,<br/>  training_effectiveness: 0.46,<br/>  campaigns_blocked: 1,<br/>  rules_deployed: 4,<br/>  users_trained: 31<br/>}

    Queue->>API: Publish monthly report to dashboard<br/>├─ Key metrics<br/>├─ Trend analysis<br/>├─ Recommendations<br/>└─ Action items for next month

    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════
    Note over SOCAnalyst,Export: ✅ KPI DASHBOARD CYCLE COMPLETE
    Note over SOCAnalyst,Export: ═══════════════════════════════════════════════════════

    rect rgb(0, 150, 0)
        Note over SOCAnalyst,Export: ✅ KPI DASHBOARD & METRICS AGGREGATION COMPLETE
        Note over SOCAnalyst,Export: • Data Retrieval ✅ (1500 events, 30 variants)
        Note over SOCAnalyst,Export: • Engagement Metrics ✅ (Sent, Open, Click, Submit, Report rates)
        Note over SOCAnalyst,Export: • Defense Metrics ✅ (VT: 26.7%, SIEM: 55.2%, Overall: 74%)
        Note over SOCAnalyst,Export: • Rule Metrics ✅ (4 rules, 88.3% avg confidence)
        Note over SOCAnalyst,Export: • User Risk Metrics ✅ (Top risky users & departments)
        Note over SOCAnalyst,Export: • Operational Metrics ✅ (API latency, uptime, cache)
        Note over SOCAnalyst,Export: • Threshold Alerting ✅ (3 alerts triggered)
        Note over SOCAnalyst,Export: • Real-Time Display ✅ (WebSocket updates, interactive dashboard)
        Note over SOCAnalyst,Export: • User Actions ✅ (Block, Deploy, Train, Export)
        Note over SOCAnalyst,Export: • Trend Analysis ✅ (30-day rolling metrics)
        Note over SOCAnalyst,Export: • Continuous Improvement ✅ (Nightly aggregations, improvement tracking)
        Note over SOCAnalyst,Export: 📊 TOTAL METRICS: 40+ KPIs | ALERTS: 3 | COVERAGE: 74% DETECTION
    end
```