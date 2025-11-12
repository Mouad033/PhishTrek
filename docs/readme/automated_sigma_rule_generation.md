# PhishTrek - Counter-Measure Generation Module

## Sigma Rule Generation & Automation

```mermaid
sequenceDiagram
    actor Analyst as Security Analyst<br/>(Blue Team)
    participant Web as Frontend<br/>(React Dashboard)
    participant API as Backend API<br/>(FastAPI)
    participant DB as PostgreSQL<br/>(Campaign Data)
    participant Cache as Cache Layer<br/>(Redis)
    participant Queue as Job Queue<br/>(Celery)
    participant Detector as Detection<br/>Pipeline (ML)
    participant LogAnalyzer as Log Analyzer<br/>(Pattern Extraction)
    participant LLM as LLM Service<br/>(GPT-4/Claude)
    participant RuleGen as Rule Generator<br/>(Sigma Template Engine)
    participant RuleVal as Rule Validator<br/>(Sigma Parser + Logic)
    participant RuleConv as Rule Converter<br/>(Uncoder.IO)
    participant SigmaRepo as Sigma Repository<br/>(Templates)
    participant Export as Export Module<br/>(YAML/PDF/SPL/KQL)

    Note over Analyst,Export: 📝 PHASE 8️⃣: AUTOMATED SIGMA RULE GENERATION

    Note over Analyst,Export: ⏱️ PREREQUISITES (from prior phases)

    Note over Analyst: ✅ Campaign 1001 completed<br/>✅ 1500 emails sent via GoPhish<br/>✅ Results: 960 opened, 540 clicked, 135 submitted<br/>✅ VirusTotal scan completed (CLEAN verdict)<br/>✅ Detection analysis: 74% detected, 390 false negatives<br/>Next: Automatic Sigma rule generation

    Note over Analyst,Export: ═══════════════════════════════════════════════════════
    Note over Analyst,Export: ⏱️ STEP 1: RETRIEVE CAMPAIGN DATA & RESULTS
    Note over Analyst,Export: ═══════════════════════════════════════════════════════

    Analyst->>Web: 1. Click "Analyze Campaign & Generate Rules"
    Web->>Web: Display analysis options form

    Analyst->>Web: 2. Select: {<br/>  campaign_id: 1001,<br/>  analysis_type: "full_automated",<br/>  include_vt_intel: true,<br/>  include_ml_analysis: true,<br/>  rule_generation_enabled: true,<br/>  auto_deploy: false<br/>}

    Web->>API: POST /campaigns/1001/analyze-and-generate-rules {analysis_params}

    API->>Queue: ENQUEUE rule_generation_pipeline {<br/>  campaign_id: 1001,<br/>  priority: "high",<br/>  analysis_depth: "comprehensive"<br/>}
    Queue-->>API: ✅ Job queued (job_id: job_rules_12345)

    API-->>Web: {<br/>  status: "rule_generation_started",<br/>  campaign_id: 1001,<br/>  job_id: "job_rules_12345",<br/>  message: "Analyzing campaign data and generating detection rules..."<br/>}

    Web-->>Analyst: ⏳ ANALYSIS IN PROGRESS<br/>Campaign 1001<br/>Status: Analyzing email patterns and generating rules...<br/>Progress: [████░░░░░░░░░░░░░░] 20% - Retrieving campaign data

    Note over Analyst,Export: ═══════════════════════════════════════════════════════
    Note over Analyst,Export: ⏱️ STEP 2: RETRIEVE COMPLETE CAMPAIGN DATA
    Note over Analyst,Export: ═══════════════════════════════════════════════════════

    Queue->>Queue: TRIGGER rule_generation_job_main

    par RETRIEVE CAMPAIGN DATA (in parallel)

        par Retrieve Email Events
            Queue->>DB: SELECT * FROM campaign_events WHERE campaign_id = 1001
            DB-->>Queue: [1500 email events with:<br/>  - email address<br/>  - event_type (sent/opened/clicked/submitted)<br/>  - timestamp<br/>  - ip_address<br/>  - user_agent<br/>  - geolocation<br/>  - payload (if submitted)<br/>]

            Queue->>Queue: Load events into memory<br/>Total events: 1500<br/>Unique users: 950<br/>Events breakdown:<br/>├─ Sent: 1500<br/>├─ Opened: 960<br/>├─ Clicked: 540<br/>└─ Submitted: 135

        and Retrieve Email Content
            Queue->>DB: SELECT * FROM email_variants WHERE campaign_id = 1001
            DB-->>Queue: [30 email variants with:<br/>  - subject<br/>  - html_body<br/>  - text_body<br/>  - headers<br/>  - from_address<br/>  - reply_to<br/>  - custom_headers<br/>]

        and Retrieve Campaign Summary
            Queue->>DB: SELECT * FROM campaign_summary WHERE campaign_id = 1001
            DB-->>Queue: {<br/>  sent: 1500,<br/>  delivered: 1485,<br/>  opened: 960,<br/>  clicked: 540,<br/>  submitted: 135,<br/>  vt_verdict: "CLEAN",<br/>  vt_risk_score: 0.0325<br/>}

        and Retrieve Detection Results
            Queue->>DB: SELECT * FROM detection_results WHERE campaign_id = 1001
            DB-->>Queue: {<br/>  detection_rate: 0.74,<br/>  false_negative_count: 390,<br/>  confidence: 0.87,<br/>  per_variant_detection: [array of 30 detection rates]<br/>}

        and Retrieve VirusTotal Analysis
            Queue->>DB: SELECT * FROM vt_analyses WHERE campaign_id = 1001
            DB-->>Queue: {<br/>  url: "https://acme-hr-verify.example.com/login",<br/>  verdict: "CLEAN",<br/>  risk_score: 0.0325,<br/>  malicious_vendors: 2,<br/>  suspicious_vendors: 1,<br/>  vendor_results: {...complete JSON...}<br/>}

        and Retrieve Landing Page
            Queue->>DB: SELECT * FROM gophish_landing_pages WHERE campaign_id = 1001
            DB-->>Queue: {<br/>  gophish_page_id: 301,<br/>  html: "...<form method='POST'>...",<br/>  capture_credentials: true,<br/>  capture_passwords: true<br/>}

    end

    Queue->>Cache: SET campaign:1001:analysis_data {<br/>  emails: [...1500 events...],<br/>  variants: [...30 templates...],<br/>  summary: {...},<br/>  detection: {...},<br/>  vt: {...},<br/>  landing_page: {...},<br/>  ttl: 3600<br/>}
    Cache-->>Queue: ✅ Data cached

    Web-->>Analyst: Progress: [████████░░░░░░░░░░░] 40% - Campaign data retrieved

    Note over Analyst,Export: ═══════════════════════════════════════════════════════
    Note over Analyst,Export: ⏱️ STEP 3: LOG ANALYSIS & PATTERN EXTRACTION
    Note over Analyst,Export: ═══════════════════════════════════════════════════════

    Note over Queue,LogAnalyzer: ANALYZE EMAIL HEADERS & STRUCTURE

    LogAnalyzer->>LogAnalyzer: HEADER ANALYSIS (from all 30 variants)<br/><br/>Variant 1: "Urgent: Action Required"<br/>├─ From: noreply@acme-verify.example.com<br/>├─ Subject: "Urgent: Action Required on Your Account"<br/>├─ Reply-To: noreply@acme-verify.example.com<br/>├─ Return-Path: spoofed@acme-verify.example.com<br/>├─ X-Originating-IP: [192.168.1.100]<br/>├─ SPF: FAIL (not aligned)<br/>├─ DKIM: FAIL (signature missing)<br/>├─ DMARC: FAIL (policy: reject)<br/>└─ Suspicious indicators: 9/10<br/><br/>Variant 2: "Security Alert - Verify Identity"<br/>├─ From: security-alert@acme-verify.example.com<br/>├─ Subject: "Security Alert - Verify Your Identity Immediately"<br/>├─ Similar header patterns to Variant 1<br/>└─ Suspicious indicators: 8/10<br/><br/>(... analyze all 30 variants ...)<br/><br/>Variant 30: "Final Payment Confirmation"<br/>├─ From: accounts@acme-verify.example.com<br/>├─ Subject: "Final Payment Confirmation Required"<br/>└─ Suspicious indicators: 7/10

    LogAnalyzer->>LogAnalyzer: EXTRACT SENDER PATTERNS<br/><br/>From addresses across all variants:<br/>├─ noreply@acme-verify.example.com (12 variants)<br/>├─ security-alert@acme-verify.example.com (8 variants)<br/>├─ accounts@acme-verify.example.com (5 variants)<br/>├─ finance@acme-verify.example.com (3 variants)<br/>└─ support@acme-verify.example.com (2 variants)<br/><br/>Pattern: acme-verify.example.com domain (100% of emails)<br/>Legitimate domain: acme.com<br/>Similarity score: 95% (domain spoofing)<br/>Registration age: 5 days (NEW DOMAIN - SUSPICIOUS)

    LogAnalyzer->>LogAnalyzer: EXTRACT SUBJECT LINE PATTERNS<br/><br/>Keywords frequency analysis:<br/>├─ "URGENT" / "Urgent": 18/30 variants (60%)<br/>├─ "Action Required" / "Required": 15/30 (50%)<br/>├─ "Verify" / "Verify Account": 12/30 (40%)<br/>├─ "Confirm" / "Confirm Identity": 10/30 (33%)<br/>├─ "Alert" / "Security Alert": 8/30 (27%)<br/>├─ "Payment" / "Invoice": 7/30 (23%)<br/>└─ "Immediate" / "Immediately": 5/30 (17%)<br/><br/>Pattern Type: High-urgency language<br/>Phishing Indicator: Very High<br/>MITRE ATT&CK: T1598.003 (Spearphishing Link)

    LogAnalyzer->>LogAnalyzer: EXTRACT URL PATTERNS<br/><br/>URLs in email bodies:<br/>├─ Landing page URL: https://acme-hr-verify.example.com/login (100%)<br/>├─ URL shorteners detected: 8/30 variants<br/>│  ├─ bit.ly/xxx: 4 variants<br/>│  ├─ tinyurl.com/xxx: 2 variants<br/>│  └─ ow.ly/xxx: 2 variants<br/>├─ Tracking pixels: 30/30 (all variants)<br/>│  └─ Pattern: <img src="https://acme-verify.../track?r=abc" /><br/>└─ CTA buttons: 28/30 with "Click here" or "Verify Now"<br/><br/>Pattern: Mix of legitimate-looking domain + shorteners<br/>Phishing Indicator: High<br/>MITRE ATT&CK: T1598.003 (Spearphishing Link)

    LogAnalyzer->>LogAnalyzer: EXTRACT BODY CONTENT PATTERNS<br/><br/>NLP Analysis (using DistilBERT + BERT-based tokenizer):<br/>├─ Urgency sentiment: 85% (high urgency detected)<br/>├─ Formality level: 78% (formal/professional tone)<br/>├─ Perplexity score: 3.2 (suspicious - unnaturally structured)<br/>├─ Entropy: 4.8 (high entropy = structured)<br/>├─ Common phrases detected:<br/>│  ├─ "Please verify your": 18/30 (60%)<br/>│  ├─ "Click here to": 15/30 (50%)<br/>│  ├─ "Confirm your identity": 12/30 (40%)<br/>│  └─ "Do not reply to this email": 20/30 (67% - typical phishing)<br/>├─ Generic greetings: "Dear {{.FirstName}}" (30/30)<br/>└─ Linguistic markers of AI-generated content: 25/30 (83%)<br/><br/>Pattern: Generic formal template with urgency language<br/>Confidence: AI-generated content detected<br/>MITRE ATT&CK: T1566.001 (Phishing - Spearphishing)

    LogAnalyzer->>LogAnalyzer: EXTRACT ATTACHMENT PATTERNS<br/><br/>Analysis:<br/>├─ Attachments in emails: 5/30 variants<br/>│  ├─ PDF files: 3 variants<br/>│  ├─ ZIP files: 1 variant<br/>│  └─ XLS files: 1 variant<br/>├─ Attachment names:<br/>│  ├─ "Account_Verification.pdf": 2 variants<br/>│  ├─ "Invoice_20251120.pdf": 1 variant<br/>│  └─ "Credentials.zip": 1 variant<br/>└─ File size distribution: 150KB - 2MB<br/><br/>Pattern: Relatively few attachments (low risk for this campaign)<br/>Note: Can trigger additional rules for APT/malware concerns

    LogAnalyzer->>DB: INSERT extracted_patterns {<br/>  campaign_id: 1001,<br/>  sender_domain_pattern: "acme-verify.example.com",<br/>  sender_domain_spoofing: true,<br/>  sender_domain_similarity: 0.95,<br/>  sender_domain_age_days: 5,<br/>  urgency_keywords: ["URGENT", "Action Required", "Verify", "Confirm", ...],<br/>  urgency_keyword_frequency: 0.60,<br/>  url_patterns: ["https://acme-hr-verify.../login", "bit.ly/xxx", ...],<br/>  url_shortener_count: 8,<br/>  body_content_sentiment: "urgency",<br/>  body_perplexity_score: 3.2,<br/>  body_entropy_score: 4.8,<br/>  linguistic_markers: ["Dear {{FirstName}}", "Please verify", "Do not reply"],<br/>  ai_generated_confidence: 0.83,<br/>  attachment_patterns: [5 attachments across 5 variants],<br/>  extraction_timestamp: NOW()<br/>}
    DB-->>LogAnalyzer: ✅ Stored

    Web-->>Analyst: Progress: [████████████░░░░░░░] 60% - Pattern analysis complete

    Note over Analyst,Export: ═══════════════════════════════════════════════════════
    Note over Analyst,Export: ⏱️ STEP 4: CLUSTER SIMILAR PATTERNS & IDENTIFY RULES
    Note over Analyst,Export: ═══════════════════════════════════════════════════════

    Queue->>Queue: CLUSTERING & DEDUPLICATION<br/><br/>Clustering algorithm: K-means on pattern vectors<br/>Distance metric: Cosine similarity<br/>Features: sender_domain, urgency_keywords, urls, body_content

    Queue->>Queue: CLUSTER 1: DOMAIN SPOOFING<br/>Variants: 1, 2, 3, 5, 7, 10, 12, 15, 18, 22, 25, 28, 30 (13 variants)<br/>Characteristics:<br/>├─ Domain: acme-verify.example.com<br/>├─ Legitimate domain: acme.com<br/>├─ Similarity: 95%<br/>├─ Registration age: 5 days<br/>├─ SPF/DKIM/DMARC: All FAIL<br/>├─ Impact on detection: 145 false negatives (37% of total FN)<br/>├─ False positive risk: 5% (internal domain transitions)<br/>└─ Rule confidence: 95%<br/>RULE CANDIDATE: "Phishing - Domain Spoofing (Acme Lookalike)"

    Queue->>Queue: CLUSTER 2: URGENCY + URL SHORTENERS<br/>Variants: 4, 6, 8, 11, 14, 16, 19, 21, 24, 26, 29 (11 variants)<br/>Characteristics:<br/>├─ Keywords: "URGENT", "Action Required", "Verify", "Confirm"<br/>├─ Keyword frequency: 60%<br/>├─ URL shorteners: bit.ly, tinyurl, ow.ly<br/>├─ Shortener frequency: 27% of variants (8/30)<br/>├─ Click rate: 44% (very high effectiveness)<br/>├─ Impact on detection: 120 false negatives (31% of total FN)<br/>├─ False positive risk: 8% (marketing campaigns with shorteners)<br/>└─ Rule confidence: 92%<br/>RULE CANDIDATE: "Phishing - Urgency Keywords + URL Shorteners"

    Queue->>Queue: CLUSTER 3: HEADER MANIPULATION & SPF BYPASS<br/>Variants: 2, 9, 13, 17, 20, 23, 27 (7 variants)<br/>Characteristics:<br/>├─ Forged headers:<br/>│  ├─ Reply-To mismatch: 100%<br/>│  ├─ Return-Path spoofed: 100%<br/>│  └─ X-Originating-IP spoofed: 85%<br/>├─ SPF result: FAIL (60% of emails)<br/>├─ DKIM result: FAIL (60% of emails)<br/>├─ DMARC result: FAIL (100%)<br/>├─ Impact on detection: 85 false negatives (22% of total FN)<br/>├─ False positive risk: 12% (forwarded emails, third-party services)<br/>└─ Rule confidence: 88%<br/>RULE CANDIDATE: "Phishing - Email Header Manipulation & SPF/DKIM Bypass"

    Queue->>Queue: CLUSTER 4: AI-GENERATED LANGUAGE MARKERS<br/>Variants: 3, 6, 12, 18, 24, 30 (6 variants)<br/>Characteristics:<br/>├─ Linguistic patterns:<br/>│  ├─ Formal but generic tone: 78%<br/>│  ├─ Perplexity score: 3.2 (unnaturally structured)<br/>│  ├─ Entropy score: 4.8 (high - structured patterns)<br/>│  ├─ Repetition patterns: 2+ similar phrases<br/>│  └─ AI confidence: 83%<br/>├─ Phrases: "Dear {{FirstName}}", "Please verify", "I hope this email"<br/>├─ Impact on detection: 40 false negatives (10% of total FN)<br/>├─ False positive risk: 15% (legitimate automated emails, templates)<br/>└─ Rule confidence: 78%<br/>RULE CANDIDATE: "Phishing - AI-Generated Email Content Indicators"

    Queue->>Queue: UNIQUE PATTERNS NOT YET COVERED<br/>Variants with unique characteristics: 1 variant<br/>├─ Unusual geolocation patterns<br/>├─ Time-based delivery patterns<br/>└─ Action: Monitor (low impact, insufficient data for rule)

    Queue->>DB: INSERT pattern_clusters {<br/>  campaign_id: 1001,<br/>  clusters: [<br/>    {<br/>      cluster_id: 1,<br/>      name: "Domain Spoofing",<br/>      variant_count: 13,<br/>      coverage_fn_count: 145,<br/>      confidence: 0.95,<br/>      fp_risk: 0.05<br/>    },<br/>    {<br/>      cluster_id: 2,<br/>      name: "Urgency + Short URL",<br/>      variant_count: 11,<br/>      coverage_fn_count: 120,<br/>      confidence: 0.92,<br/>      fp_risk: 0.08<br/>    },<br/>    {<br/>      cluster_id: 3,<br/>      name: "Header Manipulation",<br/>      variant_count: 7,<br/>      coverage_fn_count: 85,<br/>      confidence: 0.88,<br/>      fp_risk: 0.12<br/>    },<br/>    {<br/>      cluster_id: 4,<br/>      name: "AI-Generated Content",<br/>      variant_count: 6,<br/>      coverage_fn_count: 40,<br/>      confidence: 0.78,<br/>      fp_risk: 0.15<br/>    }<br/>  ],<br/>  total_fn_coverage: 390,<br/>  total_rules_recommended: 4<br/>}
    DB-->>Queue: ✅ Clusters stored

    Web-->>Analyst: Progress: [██████████████░░░░░░] 75% - Pattern clustering complete<br/>Identified 4 rule candidates covering 390 false negatives

    Note over Analyst,Export: ═══════════════════════════════════════════════════════
    Note over Analyst,Export: ⏱️ STEP 5: LLM-POWERED SIGMA RULE GENERATION
    Note over Analyst,Export: ═══════════════════════════════════════════════════════

    Note over RuleGen,LLM: PREPARE SIGMA RULE GENERATION PROMPT

    RuleGen->>RuleGen: BUILD COMPREHENSIVE PROMPT<br/><br/>Prompt template:<br/>"Generate 4 Sigma detection rules for PhishTrek campaign 1001<br/>based on identified attack patterns.<br/><br/>Pattern 1: Domain Spoofing<br/>- Fake domain: acme-verify.example.com<br/>- Legitimate domain: acme.com<br/>- Similarity: 95%<br/>- Registration age: 5 days (new domain)<br/>- Indicators: Email from_domain, SPF/DKIM failure<br/>- Coverage: 145 false negatives (37% of FN)<br/>- Confidence: 95%<br/>- MITRE ATT&CK: T1566.001<br/>- Logsource: email gateway / Exchange / Proofpoint<br/>- Detection logic: Detect emails from domain similar to legitimate domain with SPF/DKIM fail<br/><br/>Pattern 2: Urgency Keywords + Short URL<br/>- Keywords: URGENT, ACTION REQUIRED, VERIFY ACCOUNT, CONFIRM IDENTITY<br/>- URL patterns: bit.ly, tinyurl.com, short.link<br/>- Click rate: 44% (high effectiveness)<br/>- Coverage: 120 false negatives (31% of FN)<br/>- Confidence: 92%<br/>- MITRE ATT&CK: T1598.003<br/>- Logsource: Email body + HTTP logs<br/>- Detection logic: Detect emails with urgency keywords AND shortened URLs<br/><br/>Pattern 3: Header Manipulation<br/>- Forged Reply-To and Return-Path<br/>- SPF/DKIM/DMARC bypass: 60%<br/>- Coverage: 85 false negatives (22% of FN)<br/>- Confidence: 88%<br/>- MITRE ATT&CK: T1187<br/>- Logsource: Email headers<br/>- Detection logic: Detect spoofed headers and auth failures<br/><br/>Pattern 4: AI-Generated Content<br/>- Linguistic markers: high entropy, formal structure<br/>- Perplexity score: 3.2<br/>- Entropy: 4.8<br/>- Coverage: 40 false negatives (10% of FN)<br/>- Confidence: 78%<br/>- MITRE ATT&CK: T1566.001<br/>- Logsource: Email body content<br/>- Detection logic: Detect AI-generated patterns<br/><br/>For each rule, generate complete Sigma YAML with:<br/>- title, id, status, date, description<br/>- author: PhishTrek Automated Generator<br/>- logsource (product, service)<br/>- detection (selection, condition, filters)<br/>- field mappings for Splunk/ELK/Sentinel<br/>- level: high/medium based on confidence<br/>- tags: attack.phishing, mitre_att&ck mapping<br/>- falsepositives: known FP scenarios<br/>- custom fields for enrichment<br/><br/>Format: Valid YAML that passes sigma-cli validation\""

    RuleGen->>LLM: POST /completions {<br/>  model: "gpt-4",<br/>  prompt: "...(above prompt)...",<br/>  temperature: 0.3,<br/>  max_tokens: 8000,<br/>  top_p: 0.95<br/>}

    alt ✅ LLM GENERATION SUCCESS (200 OK)
        LLM-->>RuleGen: {<br/>  choices: [<br/>    {<br/>      text: "# Sigma Detection Rules for PhishTrek Campaign 1001\n\n---\ntitle: Phishing - Domain Spoofing (Acme Lookalike)\nid: acme-phishing-domain-spoof-001\nstatus: experimental\ndate: 2025-11-20\nauthor: PhishTrek Automated Generator\ndescription: Detects emails from spoofed Acme domains (acme-verify.com, acme-auth.com, acme-login.com)\nlogsource:\n  product: email\n  service: gateway\ndetection:\n  selection:\n    From_Domain|endswith:\n      - 'acme-verify.com'\n      - 'acme-auth.com'\n      - 'acme-login.com'\n    SPF_Result: 'fail'\n    DKIM_Result: 'fail'\n  condition: selection\nfields:\n  - From_Address\n  - To_Address\n  - From_Domain\n  - Subject\n  - Timestamp\nfalsepositives:\n  - Internal domain transitions (check with IT)\n  - Legitimate vendor communications\nlevel: high\ntags:\n  - attack.phishing\n  - attack.t1566.001\n  - phishing_campaign_acme_nov2025\n---\ntitle: Phishing - Urgency Keywords with URL Shorteners\nid: phishing-urgency-short-url-001\n...\n...\n"<br/>    }<br/>  ],<br/>  usage: {<br/>    prompt_tokens: 2150,<br/>    completion_tokens: 5200,<br/>    total_tokens: 7350<br/>  }<br/>}

        RuleGen->>RuleGen: PARSE LLM OUTPUT<br/>Extract 4 YAML rules from response:<br/>1. detection_domain_spoofing_acme_verify.yml<br/>2. detection_urgency_short_url.yml<br/>3. detection_header_manipulation.yml<br/>4. detection_ai_generated_content.yml

        RuleGen->>Queue: Store rules in memory for further processing

    else ⚠️ RATE LIMITED (429)
        LLM-->>RuleGen: {error: "Rate limited"}

        RuleGen->>Queue: ENQUEUE retry_llm_generation {<br/>  backoff_delay: 60,<br/>  max_retries: 3<br/>}
        Note over RuleGen: Retry after 60 seconds

    else ❌ ERROR (5xx)
        LLM-->>RuleGen: {error: "Server error"}

        RuleGen->>RuleGen: Use template-based fallback<br/>Generate rules from Sigma repository templates<br/>Note: Lower confidence than LLM-generated

    end

    RuleGen->>DB: INSERT generated_sigma_rules {<br/>  campaign_id: 1001,<br/>  rule_count: 4,<br/>  llm_model: "GPT-4",<br/>  llm_tokens: 7350,<br/>  rules_yaml: [...4 YAML rules...],<br/>  generation_timestamp: NOW(),<br/>  status: "generated"<br/>}
    DB-->>RuleGen: ✅ Rules stored (rule_ids: 1001_1, 1001_2, 1001_3, 1001_4)

    Web-->>Analyst: Progress: [███████████████████░] 90% - Rules generated (4 total)

    Note over Analyst,Export: ═══════════════════════════════════════════════════════
    Note over Analyst,Export: ⏱️ STEP 6: RULE VALIDATION (Syntax & Logic)
    Note over Analyst,Export: ═══════════════════════════════════════════════════════

    Note over RuleVal: VALIDATE 4 SIGMA RULES

    par Validate Rule 1 (Domain Spoofing)
        RuleVal->>RuleVal: RULE 1 VALIDATION<br/>Title: "Phishing - Domain Spoofing (Acme Lookalike)"<br/>ID: acme-phishing-domain-spoof-001

        RuleVal->>RuleVal: 1. YAML SYNTAX CHECK<br/>├─ Parse YAML structure: ✅ VALID<br/>├─ Required fields present: ✅ (title, id, status, date, author, description, logsource, detection, level, tags)<br/>├─ Field format validation: ✅ OK<br/>└─ Overall: ✅ SYNTAX VALID

        RuleVal->>RuleVal: 2. LOGSOURCE VALIDATION<br/>Logsource: email / gateway<br/>├─ Product "email": ✅ STANDARD<br/>├─ Service "gateway": ✅ SUPPORTED (Proofpoint, Cisco, MS EXO)<br/>├─ Field compatibility check:<br/>│  ├─ From_Domain: ✅ EXISTS in email logs<br/>│  ├─ SPF_Result: ✅ EXISTS in email headers<br/>│  ├─ DKIM_Result: ✅ EXISTS in email headers<br/>│  └─ Standard email fields: ✅ OK<br/>└─ Overall: ✅ LOGSOURCE VALID

        RuleVal->>RuleVal: 3. DETECTION LOGIC VALIDATION<br/>Detection: {<br/>  selection: {<br/>    From_Domain|endswith: [acme-verify.com, ...],<br/>    SPF_Result: fail,<br/>    DKIM_Result: fail<br/>  },<br/>  condition: selection<br/>}<br/>├─ Condition syntax: ✅ VALID<br/>├─ Field operators: ✅ endswith is valid Sigma operator<br/>├─ Modifiers: ✅ Correctly applied<br/>├─ Logic: AND between fields: ✅ CORRECT (all must match)<br/>└─ Overall: ✅ DETECTION LOGIC VALID

        RuleVal->>RuleVal: 4. MITRE ATT&CK MAPPING<br/>Tags: attack.phishing, attack.t1566.001<br/>├─ Technique T1566.001: ✅ EXISTS<br/>├─ Technique name: "Phishing: Spearphishing Attachment" ✅ CORRECT<br/>├─ Tag format: ✅ attack.t1566.001 is valid<br/>└─ Overall: ✅ MITRE MAPPING VALID

        RuleVal->>RuleVal: 5. FALSE POSITIVE ESTIMATION<br/>Stated FP scenarios: "Internal domain transitions, Legitimate vendor communications"<br/>├─ Common legitimate scenarios: ✅ COVERED<br/>├─ Filter recommendations: Add whitelist for internal domains: ✅ GOOD<br/>└─ Estimated FP rate: 5% (from pattern analysis)<br/>Overall: ✅ FP ASSESSMENT REASONABLE

        RuleVal->>RuleVal: 6. COVERAGE ANALYSIS<br/>Pattern coverage: 145 false negatives out of 390<br/>├─ Coverage percentage: 145/390 = 37%<br/>├─ Expected detection rate: 96% (from pattern analysis)<br/>├─ Expected FP rate: 5%<br/>└─ Overall: ✅ COVERAGE GOOD

        RuleVal-->>Queue: RULE 1 VALIDATION RESULT: ✅ PASS<br/>Confidence: 95%<br/>Recommendation: APPROVED - Ready for use<br/>Note: Monitor FP rate for internal domain whitelist needs

    and Validate Rule 2 (Urgency + Short URL)
        RuleVal->>RuleVal: RULE 2 VALIDATION<br/>Title: "Phishing - Urgency Keywords with URL Shorteners"<br/>ID: phishing-urgency-short-url-001

        RuleVal->>RuleVal: (Similar validation as Rule 1...)

        RuleVal-->>Queue: RULE 2 VALIDATION RESULT: ✅ PASS<br/>Confidence: 92%<br/>Recommendation: APPROVED - Ready for use

    and Validate Rule 3 (Header Manipulation)
        RuleVal->>RuleVal: RULE 3 VALIDATION<br/>Title: "Phishing - Email Header Manipulation & SPF/DKIM Bypass"<br/>ID: phishing-header-spoof-001

        RuleVal->>RuleVal: (Similar validation...)

        RuleVal-->>Queue: RULE 3 VALIDATION RESULT: ✅ PASS<br/>Confidence: 88%<br/>Recommendation: APPROVED - With caveats for third-party services

    and Validate Rule 4 (AI-Generated Content)
        RuleVal->>RuleVal: RULE 4 VALIDATION<br/>Title: "Phishing - AI-Generated Email Content Indicators"<br/>ID: phishing-ai-content-001

        RuleVal->>RuleVal: 1. YAML SYNTAX: ✅ VALID<br/>2. LOGSOURCE: email / gateway - ✅ OK<br/>3. DETECTION FIELDS: Body, Entropy_Score, Perplexity_Score<br/>   - Body: ✅ STANDARD field<br/>   - Entropy_Score: ⚠️ CUSTOM field (requires enrichment)<br/>   - Perplexity_Score: ⚠️ CUSTOM field (requires enrichment)<br/>4. LOGSOURCE COMPATIBILITY:<br/>   - These fields not standard in basic email logs<br/>   - Requires log enrichment/preprocessing<br/>   - May need custom log source definition<br/>5. OVERALL: ⚠️ CONDITIONAL PASS<br/>   - Logic is correct: ✅<br/>   - But requires custom log enrichment<br/>   - Confidence: 78% (lower due to custom fields)<br/>6. RECOMMENDATION: Approve with note about enrichment requirement

        RuleVal-->>Queue: RULE 4 VALIDATION RESULT: ⚠️ CONDITIONAL PASS<br/>Confidence: 78%<br/>Recommendation: APPROVED - Requires email body enrichment pipeline

    end

    Queue->>DB: INSERT rule_validation_results {<br/>  campaign_id: 1001,<br/>  rule_1: {status: "pass", confidence: 0.95, issues: []},<br/>  rule_2: {status: "pass", confidence: 0.92, issues: []},<br/>  rule_3: {status: "pass", confidence: 0.88, issues: ["third-party-service-fp"]},<br/>  rule_4: {status: "conditional_pass", confidence: 0.78, issues: ["custom-fields-require-enrichment"]},<br/>  avg_confidence: 0.883,<br/>  validation_timestamp: NOW()<br/>}
    DB-->>Queue: ✅ Validation results stored

    Web-->>Analyst: ✅ Rules validated (confidence: 88.3% average)

    Note over Analyst,Export: ═══════════════════════════════════════════════════════
    Note over Analyst,Export: ⏱️ STEP 7: RULE CONVERSION TO SIEM FORMATS
    Note over Analyst,Export: ═══════════════════════════════════════════════════════

    Note over RuleConv,SigmaRepo: QUERY SIEM TEMPLATES & CONVERSION

    RuleConv->>SigmaRepo: GET /templates?target_siems=[splunk, elastic, azure, arcsight]

    SigmaRepo-->>RuleConv: {<br/>  templates: {<br/>    splunk: {...conversion template...},<br/>    elastic: {...conversion template...},<br/>    azure: {...conversion template...},<br/>    arcsight: {...conversion template...}<br/>  }<br/>}

    par Convert Rules to Multiple SIEM Formats

        par Convert Rule 1 (Domain Spoofing)
            RuleConv->>RuleConv: CONVERT RULE 1 TO SPLUNK SPL<br/>Input: Sigma YAML<br/>Output: Splunk Search Processing Language<br/><br/>```spl<br/>sourcetype=email from_domain IN (acme-verify.com, acme-auth.com, acme-login.com)<br/>  spf_result=fail dkim_result=fail<br/>| table _time, from, to, from_domain, subject<br/>| where isnotnull(from)<br/>```

            RuleConv->>RuleConv: CONVERT RULE 1 TO ELASTIC KQL<br/><br/>```kql<br/>email.from_domain: (acme-verify.com OR acme-auth.com OR acme-login.com)<br/>  AND email.spf_result: fail<br/>  AND email.dkim_result: fail<br/>```

            RuleConv->>RuleConv: CONVERT RULE 1 TO AZURE KQL<br/><br/>```kql<br/>EmailEvents<br/>| where SenderFromDomain in ("acme-verify.com", "acme-auth.com", "acme-login.com")<br/>| where EmailDirection == "Inbound"<br/>| where SenderPFReason contains "Fail" OR SenderDKIMResult contains "Fail"<br/>| project TimeGenerated, SenderMailFromDomain, RecipientEmailAddress, Subject<br/>```

            RuleConv->>RuleConv: CONVERT RULE 1 TO ARCSIGHT ESL<br/><br/>```esl<br/>rule_name: Phishing_Domain_Spoofing_Acme<br/>event_filter: (emailFrom REGEX ".*@acme-verify\\.com")<br/>  AND (spfResult="fail")<br/>  AND (dkimResult="fail")<br/>alert_severity: high<br/>```

        and Convert Rule 2 (Urgency + Short URL)
            RuleConv->>RuleConv: (Convert to Splunk, Elastic, Azure, ArcSight...)

        and Convert Rule 3 (Header Manipulation)
            RuleConv->>RuleConv: (Convert to all SIEM formats...)

        and Convert Rule 4 (AI-Generated Content)
            RuleConv->>RuleConv: (Convert to all SIEM formats...)

        end

    end

    RuleConv-->>Queue: CONVERSION COMPLETE<br/>├─ 4 Sigma rules<br/>├─ × 4 SIEM formats (Splunk, Elastic, Azure, ArcSight)<br/>└─ = 16 total conversion outputs

    Queue->>DB: INSERT rule_conversions {<br/>  campaign_id: 1001,<br/>  rule_id: 1001_1,<br/>  conversions: {<br/>    splunk_spl: "sourcetype=email from_domain IN...",<br/>    elastic_kql: "email.from_domain: (acme-verify.com OR...)",<br/>    azure_kql: "EmailEvents | where SenderFromDomain in...",<br/>    arcsight_esl: "rule_name: Phishing_Domain_Spoofing..."<br/>  },<br/>  conversion_timestamp: NOW()<br/>}
    DB-->>Queue: ✅ Stored (repeated for rules 1001_2, 1001_3, 1001_4)

    Web-->>Analyst: ✅ Rules converted to Splunk, Elastic, Azure, ArcSight formats

    Note over Analyst,Export: ═══════════════════════════════════════════════════════
    Note over Analyst,Export: ⏱️ STEP 8: ANALYST REVIEW & APPROVAL
    Note over Analyst,Export: ═══════════════════════════════════════════════════════

    Web->>Web: Display rule review panel to analyst

    Web-->>Analyst: 🎯 SIGMA RULES GENERATED & VALIDATED<br/><br/>Campaign: Acme HR Q4 Test<br/>Rules Generated: 4<br/>Average Confidence: 88.3%<br/><br/>Rule 1: Domain Spoofing Detection<br/>  ├─ Confidence: 95%<br/>  ├─ Coverage: 145 FN detected<br/>  ├─ FP Risk: 5%<br/>  └─ [View Details] [Preview SPL] [Edit] [Approve]<br/><br/>Rule 2: Urgency + Short URL<br/>  ├─ Confidence: 92%<br/>  ├─ Coverage: 120 FN detected<br/>  ├─ FP Risk: 8%<br/>  └─ [View Details] [Preview SPL] [Edit] [Approve]<br/><br/>Rule 3: Header Manipulation<br/>  ├─ Confidence: 88%<br/>  ├─ Coverage: 85 FN detected<br/>  ├─ FP Risk: 12%<br/>  └─ [View Details] [Preview SPL] [Edit] [Approve]<br/><br/>Rule 4: AI-Generated Content (⚠️ Requires enrichment)<br/>  ├─ Confidence: 78%<br/>  ├─ Coverage: 40 FN detected<br/>  ├─ FP Risk: 15%<br/>  └─ [View Details] [Preview SPL] [Edit] [Approve]

    par Analyst reviews Rule 1

        Analyst->>Web: Click "Preview SPL" for Rule 1
        Web->>Web: Display Splunk SPL in modal
        Web-->>Analyst: ```spl<br/>sourcetype=email from_domain IN (acme-verify.com, acme-auth.com, acme-login.com)<br/>  spf_result=fail dkim_result=fail<br/>| table _time, from, to, from_domain, subject<br/>| where isnotnull(from)<br/>```<br/><br/>Analyst reviews:<br/>├─ Syntax: ✅ Looks correct<br/>├─ Logsource: ✅ email/gateway matches environment<br/>├─ Logic: ✅ Domain spoofing check makes sense<br/>├─ FP concern: Internal domain transitions - reasonable<br/>└─ Verdict: APPROVE ✅

        Analyst->>Web: Click "Approve" for Rule 1

    and Analyst edits Rule 4

        Analyst->>Web: Click "Edit" for Rule 4 (AI-Generated Content)
        Web->>Analyst: Display YAML editor

        Analyst->>Web: Modify YAML:<br/>├─ Remove pattern "Dear (Sir|Madam|User|Friend)" (too many FP)<br/>├─ Keep AI marker patterns<br/>├─ Add note: "Requires email body enrichment"<br/>├─ Set level: "medium" (not high)<br/>└─ Save changes

        Web->>API: PATCH /campaigns/1001/sigma-rules/1001_4 {<br/>  rule_yaml: "...modified YAML...",<br/>  edit_reason: "Removed overly broad pattern to reduce FP"<br/>}

        Analyst->>Web: Click "Approve" for Rule 4 (modified)

    end

    Analyst->>Web: Click "Approve & Export"

    Web->>API: POST /campaigns/1001/sigma-rules/approve {<br/>  approved_rules: [1001_1, 1001_2, 1001_3, 1001_4],<br/>  rule_modifications: {<br/>    1001_4: {<br/>      removed_patterns: ["Dear (Sir|Madam|User|Friend)"],<br/>      notes: "Reduces FP. Requires email body enrichment."<br/>    }<br/>  },<br/>  export_formats: ["yaml", "splunk_spl", "elastic_kql", "csv"],<br/>  approval_timestamp: NOW()<br/>}

    API->>DB: UPDATE generated_sigma_rules SET {<br/>  approval_status: "approved",<br/>  approved_by: analyst_user_id,<br/>  approval_timestamp: NOW(),<br/>  approved_rule_ids: [1001_1, 1001_2, 1001_3, 1001_4],<br/>  rule_modifications: {...},<br/>  ready_for_export: true<br/>}
    DB-->>API: ✅ Rules approved

    Web-->>Analyst: ✅ All 4 rules approved

    Note over Analyst,Export: ═══════════════════════════════════════════════════════
    Note over Analyst,Export: ⏱️ STEP 9: MULTI-FORMAT EXPORT
    Note over Analyst,Export: ═══════════════════════════════════════════════════════

    API->>Export: Generate export package {<br/>  campaign_id: 1001,<br/>  formats: ["yaml", "splunk_spl", "elastic_kql", "csv", "pdf"]<br/>}

    par Generate YAML Export

        Export->>Export: GENERATE SIGMA YAML FILE<br/>├─ Combine 4 approved rules<br/>├─ Add metadata: campaign_id, approval_date, analyst<br/>├─ Add comments: coverage %, FP rate, MITRE mapping<br/>├─ Format: Valid Sigma YAML<br/>└─ Output: sigma_rules_acme_hr_nov2025.yaml

        Export->>Cache: Cache YAML file (ttl: 86400)

    and Generate SPL Export

        Export->>Export: GENERATE SPLUNK DEPLOYMENT<br/>├─ Convert 4 rules to SPL format<br/>├─ Create saved searches<br/>├─ Add alert actions: email on match<br/>├─ Format: Splunk JSON (importable via REST API)<br/>└─ Output: phistrek_rules_splunk_deployment.json

    and Generate KQL Export

        Export->>Export: GENERATE ELASTIC/AZURE EXPORT<br/>├─ Convert 4 rules to KQL<br/>├─ Create detection rules in Kibana format<br/>├─ Add alerting rules<br/>├─ Format: NDJSON (newline-delimited JSON)<br/>└─ Output: phistrek_rules_elastic_export.ndjson

    and Generate CSV Report

        Export->>Export: GENERATE CSV REPORT<br/>Header row: Rule ID, Title, MITRE ATT&CK, Coverage (FN), Confidence, Logsource, FP Rate, Status<br/>Row 1: phishing-domain-spoof-001, "Domain Spoofing", T1566.001, 145, 95%, email/gateway, 5%, Approved<br/>Row 2: phishing-urgency-short-url-001, "Urgency + URL", T1598.003, 120, 92%, email/gateway, 8%, Approved<br/>Row 3: phishing-header-spoof-001, "Header Manip", T1187, 85, 88%, email/gateway, 12%, Approved<br/>Row 4: phishing-ai-content-001, "AI Content", T1566.001, 40, 78%, email/gateway, 15%, Approved<br/>Output: phistrek_rules_report.csv

    and Generate PDF Documentation

        Export->>Export: GENERATE COMPREHENSIVE PDF<br/>├─ Page 1: Executive Summary<br/>│  ├─ Campaign overview<br/>│  ├─ Key statistics<br/>│  └─ High-level findings<br/>├─ Pages 2-5: Rule Details (1 per rule)<br/>│  ├─ Rule logic explanation<br/>│  ├─ Full YAML code<br/>│  ├─ MITRE ATT&CK mapping<br/>│  ├─ Expected alerts & FP<br/>│  ├─ Deployment instructions<br/>│  └─ Splunk/Elastic/Azure examples<br/>├─ Page 6: Deployment Guide<br/>│  ├─ Prerequisites<br/>│  ├─ Step-by-step deployment<br/>│  └─ Validation tests<br/>├─ Page 7: Troubleshooting<br/>│  ├─ Common issues<br/>│  ├─ Error resolution<br/>│  └─ Support contacts<br/>└─ Appendix: Raw JSON vendor detection data

        Export->>Export: Generate PDF (2.1 MB)

    end

    Export-->>API: Export package complete:<br/>├─ sigma_rules_acme_hr_nov2025.yaml (5 KB)<br/>├─ phistrek_rules_splunk_deployment.json (8 KB)<br/>├─ phistrek_rules_elastic_export.ndjson (7 KB)<br/>├─ phistrek_rules_report.csv (2 KB)<br/>├─ phistrek_rules_documentation.pdf (2.1 MB)<br/>└─ phistrek_deployment_package.zip (all files, 2.2 MB)

    API->>DB: INSERT rule_exports {<br/>  campaign_id: 1001,<br/>  export_formats: ["yaml", "splunk", "elastic", "csv", "pdf"],<br/>  file_paths: {...},<br/>  package_url: "https://phistrek.example.com/download/export_1001.zip",<br/>  export_timestamp: NOW(),<br/>  analyst_id: analyst_user_id<br/>}
    DB-->>API: ✅ Export logged

    API-->>Web: {<br/>  status: "exported",<br/>  files: {<br/>    yaml: "sigma_rules_acme_hr_nov2025.yaml",<br/>    splunk: "phistrek_rules_splunk_deployment.json",<br/>    elastic: "phistrek_rules_elastic_export.ndjson",<br/>    csv: "phistrek_rules_report.csv",<br/>    pdf: "phistrek_rules_documentation.pdf"<br/>  },<br/>  zip_file: "phistrek_deployment_package_1001.zip",<br/>  download_link: "https://phistrek.example.com/download/export_1001"<br/>}

    Web-->>Analyst: ✅ EXPORT READY<br/><br/>📦 Download Options:<br/>├─ [Download ZIP Package] (all files 2.2 MB)<br/>├─ [Download Sigma YAML Only]<br/>├─ [Download Splunk JSON Only]<br/>├─ [Download Elastic NDJSON Only]<br/>├─ [View PDF in Browser]<br/>└─ [Email Rules to SOC Team]<br/><br/>📋 Files in Package:<br/>├─ sigma_rules_acme_hr_nov2025.yaml<br/>├─ phistrek_rules_splunk_deployment.json<br/>├─ phistrek_rules_elastic_export.ndjson<br/>├─ phistrek_rules_report.csv<br/>├─ phistrek_rules_documentation.pdf<br/>└─ README.md (deployment instructions)

    Analyst->>Web: Click "Download ZIP Package"
    Web-->>Analyst: 📥 phistrek_deployment_package_1001.zip (2.2 MB)

    Note over Analyst,Export: ═══════════════════════════════════════════════════════
    Note over Analyst,Export: ⏱️ STEP 10: OPTIONAL - DIRECT SIEM DEPLOYMENT
    Note over Analyst,Export: ═══════════════════════════════════════════════════════

    alt Analyst chooses: Deploy to Splunk directly

        Analyst->>Web: Click "Deploy to Splunk"
        Web->>Web: Show SIEM connection form

        Analyst->>Web: Enter: {<br/>  siem: "splunk",<br/>  endpoint: "https://splunk.example.com:8089",<br/>  auth_token: "***encrypted***"<br/>}

        Web->>API: POST /campaigns/1001/sigma-rules/deploy {<br/>  target_siem: "splunk",<br/>  splunk_endpoint: "https://splunk.example.com:8089",<br/>  auth_token: "***"<br/>}

        API->>API: VALIDATE SPLUNK CONNECTION<br/>Test connection to Splunk API<br/>Verify authentication token<br/>Check permissions for saved searches

        alt Connection successful
            API->>API: Create 4 Splunk saved searches via REST API

            API->>API: For each rule:<br/>POST /servicesNS/admin/search/saved/searches {<br/>  name: "PhishTrek_Rule_1_Domain_Spoofing",<br/>  search: "sourcetype=email from_domain IN...",<br/>  description: "Rule: Phishing - Domain Spoofing",<br/>  alert: true,<br/>  alert_type: "always",<br/>  alert_threshold: "count > 0",<br/>  actions: "send_email",<br/>  email_to: "soc-team@company.com",<br/>  owner: "PhishTrek"<br/>}

            API-->>API: ✅ 4 rules deployed to Splunk

            API-->>Web: {<br/>  status: "deployed",<br/>  siem: "splunk",<br/>  rules_deployed: 4,<br/>  deployment_time: NOW(),<br/>  message: "Rules deployed to Splunk. Alerts active immediately."<br/>}

            Web-->>Analyst: ✅ RULES DEPLOYED TO SPLUNK<br/>4 saved searches created and active<br/>Alerts will be sent to: soc-team@company.com

        else Connection failed
            API-->>Web: {error: "Cannot connect to Splunk"}
            Web-->>Analyst: ❌ Deployment failed<br/>Download rules and import manually

        end

    else Analyst chooses: Manual import

        Analyst->>Analyst: Manual import process:<br/>1. Extract ZIP file<br/>2. Open phistrek_rules_splunk_deployment.json<br/>3. In Splunk UI: Settings → Saved Searches → Import<br/>4. Select JSON file<br/>5. Click Import<br/>6. Verify 4 new saved searches appear<br/>7. Alerts now active

    end

    Note over Analyst,Export: ═══════════════════════════════════════════════════════
    Note over Analyst,Export: ⏱️ STEP 11: ENRICHMENT & POST-GENERATION LOGGING
    Note over Analyst,Export: ═══════════════════════════════════════════════════════

    par Post-Generation Processing

        API->>DB: INSERT rule_generation_summary {<br/>  campaign_id: 1001,<br/>  generation_timestamp: NOW(),<br/>  total_rules_generated: 4,<br/>  rules_validated: 4,<br/>  rules_approved: 4,<br/>  avg_confidence: 0.883,<br/>  total_fn_coverage: 390,<br/>  detection_improvement_percent: 37,<br/>  siem_formats_converted: 4,<br/>  export_formats: 5,<br/>  analyst_review_time_minutes: 15,<br/>  total_processing_time_minutes: 45<br/>}

        and Update Campaign Status

            API->>DB: UPDATE campaigns SET {<br/>  status: "rule_generation_complete",<br/>  rule_generation_complete_at: NOW(),<br/>  rules_generated_count: 4,<br/>  rules_deployed_count: 0,<br/>  next_action_required: "Deploy rules to SIEM or monitor FP rate"<br/>}

        and Create Audit Log

            API->>DB: INSERT audit_log {<br/>  campaign_id: 1001,<br/>  action: "sigma_rules_generated_and_exported",<br/>  timestamp: NOW(),<br/>  user_id: analyst_user_id,<br/>  details: {<br/>  rules_count: 4,<br/>  rule_ids: [1001_1, 1001_2, 1001_3, 1001_4],<br/>  export_formats: [yaml, splunk, elastic, csv, pdf],<br/>  download_count: 0,<br/>  deployment_status: "manual_export_provided"<br/>  }<br/>}

        and Link to VirusTotal Intelligence

            API->>DB: INSERT rule_vt_enrichment {<br/>  campaign_id: 1001,<br/>  rules_count: 4,<br/>  vt_vendor_categories_detected: ["phishing", "suspicious"],<br/>  vt_data_used_for_rules: true,<br/>  top_vendors_for_reference: ["Kaspersky", "McAfee", "Sophos"],<br/>  vt_integration_notes: "Top vendor detections included in rule tags for context"<br/>}

    end

    Note over Analyst,Export: ═══════════════════════════════════════════════════════
    Note over Analyst,Export: ✅ AUTOMATED SIGMA RULE GENERATION COMPLETE
    Note over Analyst,Export: ═══════════════════════════════════════════════════════

    rect rgb(0, 150, 0)
        Note over Analyst,Export: ✅ SIGMA RULE GENERATION PIPELINE COMPLETE
        Note over Analyst,Export: • Campaign Data Retrieval ✅ (1500 events, 30 variants)
        Note over Analyst,Export: • Log Analysis & Pattern Extraction ✅ (Sender, Subject, URL, Body, Headers)
        Note over Analyst,Export: • Pattern Clustering ✅ (4 clusters, 390 FN coverage)
        Note over Analyst,Export: • LLM-Powered Generation ✅ (4 Sigma YAML rules)
        Note over Analyst,Export: • Syntax Validation ✅ (100% pass rate)
        Note over Analyst,Export: • Logic Validation ✅ (88.3% avg confidence)
        Note over Analyst,Export: • Logsource Compatibility ✅ (All standard email fields)
        Note over Analyst,Export: • MITRE ATT&CK Mapping ✅ (All techniques correct)
        Note over Analyst,Export: • Multi-SIEM Conversion ✅ (4 formats: SPL, KQL, Azure, ArcSight)
        Note over Analyst,Export: • Analyst Review ✅ (All rules approved with edits)
        Note over Analyst,Export: • Multi-Format Export ✅ (YAML, SPL, KQL, CSV, PDF)
        Note over Analyst,Export: • Direct SIEM Deployment ✅ (Optional - Splunk ready)
        Note over Analyst,Export: • Audit & Logging ✅ (Complete traceability)
        Note over Analyst,Export: 📊 TOTAL TIME: 45 minutes | RULES: 4 | FN COVERAGE: 390 | CONFIDENCE: 88.3%
    end
```