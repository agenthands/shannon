# Shannon Prompt Architecture: Complete Reference

**Document Version:** 1.0
**Last Updated:** 2025-02-02
**Purpose:** Comprehensive guide to Shannon's LLM prompt system, including all agent prompts, their purposes, and cross-phase coordination patterns.

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Prompt System Overview](#prompt-system-overview)
3. [Phase-by-Phase Prompt Analysis](#phase-by-phase-prompt-analysis)
4. [Cross-Phase Coordination Patterns](#cross-phase-coordination-patterns)
5. [Data Flow and Handoff Mechanisms](#data-flow-and-handoff-mechanisms)
6. [Shared Partial Templates](#shared-partial-templates)
7. [Queue-Based Validation System](#queue-based-validation-system)
8. [Prompt Design Principles](#prompt-design-principles)
9. [Pipeline-Testing Variants](#pipeline-testing-variants)
10. [Prompt Reference Cards](#prompt-reference-cards)

---

## Executive Summary

Shannon uses a **sophisticated 13-agent prompt architecture** that coordinates a white-box penetration testing workflow through five distinct phases. Each agent receives a highly specialized prompt (300-400 lines) that defines:

1. **Role & Objectives** - Clear specialist identity
2. **System Architecture Context** - Position in the workflow pipeline
3. **Input/Output Specifications** - Data handoff points
4. **Methodology** - Step-by-step analysis procedures
5. **Tool Usage Guidelines** - Task agent delegation, MCP tools
6. **Evidence Requirements** - Proof standards for findings

**Key Innovation:** The "No Exploit, No Report" principle ensures only vulnerabilities successfully exploited in Phase 4 reach the final report, eliminating false positives.

---

## Prompt System Overview

### Prompt File Structure

```
prompts/
├── shared/                          # Reusable partial templates
│   ├── _target.txt                  # Target URL injection
│   ├── _rules.txt                   # Avoid/focus rules from config
│   ├── _vuln-scope.txt              # External attacker scope definition
│   ├── _exploit-scope.txt           # Exploitation constraints + verdict classifications
│   └── login-instructions.txt       # Authentication flow (form/SSO/TOTP)
│
├── pre-recon-code.txt               # Phase 1: Code intelligence gathering
├── recon.txt                        # Phase 2: Attack surface mapping
│
├── vuln-injection.txt               # Phase 3a: Injection vulnerability analysis
├── vuln-xss.txt                     # Phase 3a: XSS vulnerability analysis
├── vuln-auth.txt                    # Phase 3a: Authentication vulnerability analysis
├── vuln-authz.txt                   # Phase 3a: Authorization vulnerability analysis
├── vuln-ssrf.txt                    # Phase 3a: SSRF vulnerability analysis
│
├── exploit-injection.txt            # Phase 4: Injection exploitation
├── exploit-xss.txt                  # Phase 4: XSS exploitation
├── exploit-auth.txt                 # Phase 4: Authentication exploitation
├── exploit-authz.txt                # Phase 4: Authorization exploitation
├── exploit-ssrf.txt                 # Phase 4: SSRF exploitation
│
├── report-executive.txt             # Phase 5: Final report generation
│
└── pipeline-testing/                # Minimal prompts for workflow validation
    ├── pre-recon-code.txt
    ├── recon.txt
    ├── vuln-*.txt (5 files)
    ├── exploit-*.txt (5 files)
    └── report-executive.txt
```

### Prompt Statistics

| Agent Type | Prompt Lines | Task Agents Used | Phase |
|------------|--------------|------------------|-------|
| Pre-Recon   | 394          | 6 (3 phases)     | 1     |
| Recon       | 370          | 5 (parallel)     | 2     |
| Vuln Agents | 150-371      | 1-2              | 3     |
| Exploit Agents| 150-200    | TodoWrite-driven | 4     |
| Report      | 127          | 0                | 5     |
| **Total Production** | **~2,400** | **~40** | **5** |

---

## Phase-by-Phase Prompt Analysis

### Phase 1: Pre-Reconnaissance (Code Analysis)

**Prompt File:** `pre-recon-code.txt` (394 lines)
**Agent Name:** Pre-Recon Agent
**Role:** Principal Engineer / Code Intelligence Gatherer

#### Purpose

**The ONLY agent with full source code access.** Establishes the foundational intelligence baseline that all subsequent agents depend on. Performs comprehensive security-relevant architectural analysis.

#### Key Responsibilities

1. **Technology Stack Mapping** - Frameworks, languages, dependencies
2. **Entry Point Discovery** - All network-accessible endpoints, API schemas
3. **Security Pattern Identification** - Auth flows, session management, RBAC
4. **Vulnerability Sink Mapping** - XSS sinks, SSRF sinks, injection sources
5. **Data Flow Tracing** - Sensitive data handling, encryption, secrets
6. **Schema File Extraction** - OpenAPI, GraphQL, JSON Schema discovery

#### Task Agent Coordination (6 Agents in 3 Phases)

**Phase 1 (Parallel - Discovery):**
```
1. Architecture Scanner Agent → Tech stack, frameworks, security implications
2. Entry Point Mapper Agent → All network-accessible endpoints + API schemas
3. Security Pattern Hunter Agent → Auth, authz, session management, security middleware
```

**Phase 2 (Parallel - Vulnerability Analysis):**
```
4. XSS/Injection Sink Hunter Agent → Dangerous sinks with exact file:line locations
5. SSRF/External Request Tracer Agent → User-influenced outbound requests
6. Data Security Auditor Agent → PII handling, encryption, secrets management
```

**Phase 3 (Synthesis):**
```
7. Main Agent → Synthesize findings, generate report, save deliverables
```

#### Prompt Structure Highlights

```xml
<critical>
**Cascade Impact:** Your analysis is the foundation for the entire security assessment.
An incomplete analysis creates blind spots that persist through all 10+ subsequent agents.
</critical>

<system_architecture>
**WHAT HAPPENED BEFORE YOU:**
- Target application source code has been cloned
- You are the FIRST AI AGENT in the workflow

**WHAT HAPPENS AFTER YOU:**
- Reconnaissance agent uses your architectural analysis to prioritize
- 5 Vulnerability Analysis specialists use your security component mapping
- 5 Exploitation specialists use your attack surface catalog
- Final reporting agent uses your technical baseline
</system_architecture>
```

#### Deliverables

1. **`deliverables/code_analysis_deliverable.md`** - Comprehensive security architecture report
2. **`outputs/schemas/`** - All discovered API schema files

#### Critical Sections Produced

- Executive Summary (security posture overview)
- Architecture & Technology Stack
- Authentication & Authorization Deep Dive (includes exhaustive auth endpoint list)
- Data Security & Storage
- Attack Surface Analysis (network-accessible only)
- Infrastructure & Operational Security
- Overall Codebase Indexing
- Critical File Paths (categorized by security relevance)
- XSS Sinks and Render Contexts
- SSRF Sinks

---

### Phase 2: Reconnaissance (Attack Surface Mapping)

**Prompt File:** `recon.txt` (370 lines)
**Agent Name:** Recon Agent
**Role:** Reconnaissance Analyst / Attack Surface Mapper

#### Purpose

Builds a **comprehensive attack surface map** by correlating pre-recon data with live application exploration and source code analysis. Creates the detailed intelligence that all vulnerability specialists depend on.

#### Key Responsibilities

1. **API Endpoint Inventory** - All endpoints with authorization details
2. **Authentication Flow Mapping** - Complete auth mechanisms, session management
3. **Authorization Architecture** - Role hierarchies, privilege lattices, entry points
4. **Input Vector Enumeration** - Every user-controllable input point
5. **Network Interaction Mapping** - Entities, flows, guards (system boundaries)
6. **Authorization Vulnerability Candidates** - Pre-prioritized testing targets
7. **Injection Source Tracing** - Complete source-to-sink paths via Task Agent

#### Task Agent Coordination (5 Agents in Parallel)

```
1. Route Mapper Agent → Map endpoints to exact handler functions (file:line)
2. Authorization Checker Agent → Find authorization middleware, guards, permission checks
3. Input Validator Agent → Analyze input validation logic for all discovered fields
4. Session Handler Agent → Trace session/token handling (creation, storage, validation)
5. Authorization Architecture Agent → Complete role hierarchy, permission models, object ownership
```

#### Prompt Structure Highlights

```xml
<systematic_approach>
**Step 1: Synthesize Initial Data**
- Read deliverables/pre_recon_deliverable.md

**Step 2: Interactive Application Exploration**
- Navigate to target, map user-facing functionality
- Observe network requests to identify API calls

**Step 3: Correlate with Source Code using Parallel Task Agents**
- Launch 5 agents IN PARALLEL to correlate browser findings with backend code

**Step 4: Enumerate and Document**
- Synthesize findings, create comprehensive attack surface maps
</systematic_approach>
```

#### Deliverables

**`deliverables/recon_deliverable.md`** - Comprehensive attack surface map

#### Critical Sections Produced

**Section 0: How to Read This** - Guide for Authorization Analysis specialist
**Section 1: Executive Summary** - Application overview
**Section 2: Technology & Service Map** - Frontend, backend, infrastructure
**Section 3: Authentication & Session Management** - Complete auth flow with code pointers
**Section 4: API Endpoint Inventory** - Table with Method, Path, Required Role, Object ID Parameters, Authorization Mechanism
**Section 5: Potential Input Vectors** - Every user-controlled input with file:line
**Section 6: Network & Interaction Map** - Entities, flows, guards (system boundaries visualization)
**Section 7: Role & Privilege Architecture** - Discovered roles, privilege lattice, role entry points
**Section 8: Authorization Vulnerability Candidates** - Prioritized horizontal/vertical/context targets
**Section 9: Injection Sources** - Via Task Agent coordination

---

### Phase 3a: Vulnerability Analysis (5 Parallel Agents)

#### Overview

Five specialized agents analyze **specific vulnerability categories** in parallel. Each agent:
1. Reads reconnaissance deliverable
2. Uses Task Agent for code analysis
3. Creates **two** deliverables:
   - **Analysis Report** (markdown) - Secure vectors documented
   - **Exploitation Queue** (JSON) - Only exploitable vulnerabilities

---

#### Agent 1: Injection Analysis Specialist

**Prompt File:** `vuln-injection.txt` (371 lines)
**Role:** Injection Analysis Specialist
**Vulnerability Types:** SQLi, Command Injection, LFI/RFI, SSTI, Path Traversal, Deserialization

##### Purpose

Identify **source-to-sink data flow paths** where untrusted input reaches dangerous sinks without proper defenses.

##### Methodology: Negative Injection Analysis (Pre-Exploitation)

**Step 1: Create Todos**
Read `deliverables/pre_recon_deliverable.md` Section 7 (Injection Sources) → Create todo for each source

**Step 2: Trace Data Flow Paths**
For each source, identify every unique route to a sink:
- Path forking = separate analysis
- Document: transformations, sanitizers (ordered), concatenations

**Step 3: Detect Sinks & Label Slot Types**
- SQLi: `SQL-val`, `SQL-like`, `SQL-num`, `SQL-enum`, `SQL-ident`
- Command: `CMD-argument`, `CMD-part-of-string`
- File: `FILE-path`, `FILE-include`
- SSTI: `TEMPLATE-expression`
- Deserialization: `DESERIALIZE-object`
- Path: `PATH-component`

**Step 4: Match Sanitization to Sink Context**
- **SQL:** Parameter binding for val/like/num; whitelist for enum/ident
- **Command:** Array args OR `shlex.quote()`
- **File:** Whitelist paths OR `resolve()` + boundary check
- **SSTI:** Sandboxed context + autoescape
- **Deserialize:** Trusted sources only; safe formats + HMAC

**Step 5: Make the Call**
- **Vulnerable** if tainted input reaches sink with no/mismatched defense
- **Safe** if correct sanitization AND no post-sanitization concat

**Step 6: Append to Findings**
- If `verdict: vulnerable` → Add to `injection_exploitation_queue.json`
- If `verdict: safe` → Document in markdown report (not queue)

**Step 7: Score Confidence**
- **High:** Binds on value/like/num; no post-sanitization concat
- **Medium:** Binds present but upstream transforms unclear
- **Low:** Concat into syntax slots; regex-only sanitization

##### Queue Format

```json
{
  "ID": "INJ-VULN-01",
  "vulnerability_type": "SQLi | CommandInjection | LFI | RFI | SSTI | PathTraversal | InsecureDeserialization",
  "externally_exploitable": true,
  "source": "param name & file:line",
  "path": "controller → fn → sink",
  "sink_call": "file:line and function",
  "slot_type": "SQL-val | CMD-argument | FILE-path | TEMPLATE-expression | DESERIALIZE-object",
  "sanitization_observed": "name & file:line (all, in order)",
  "concat_occurrences": "each concat/format/join with file:line; flag after sanitization",
  "verdict": "vulnerable",
  "mismatch_reason": "1-2 lines in plain language",
  "witness_payload": "minimal input to show structure influence",
  "confidence": "high | med | low",
  "notes": "assumptions, untraversed branches"
}
```

##### Deliverables

1. **`deliverables/injection_analysis_deliverable.md`** - Analysis with secure vectors
2. **`deliverables/injection_exploitation_queue.json`** - Only `externally_exploitable: true`

---

#### Agent 2: XSS Analysis Specialist

**Prompt File:** `vuln-xss.txt` (150+ lines)
**Role:** XSS Analysis Specialist
**Vulnerability Types:** Reflected, Stored, DOM-based XSS

##### Purpose

Identify **context mismatch** in output encoding where sanitization doesn't match render context.

##### Methodology: Sink-to-Source Backward Analysis

**Step 1: Create Todos**
Read `deliverables/pre_recon_deliverable.md` Section 9 (XSS Sinks) → Create todo for each sink

**Step 2: Trace Backward**
For each sink, trace origin backward:
- **Early Termination Rule:** If correct sanitizer encountered AND no mutations → SAFE
- **Path Forking:** Multiple code paths = separate analysis
- **Track Mutations:** String operations that invalidate sanitization

**Step 3: Database Read Checkpoint (Stored XSS)**
If trace reaches DB read without sanitizer:
- Declare vulnerable (Stored XSS)
- Capture DB read operation (file:line, data field)
- **Simplification:** Don't trace to DB write (read = sufficient flaw)

**Step 4: Classify Vulnerability Type**
- **Reflected:** Source is HTTP request (URL param, header, body)
- **Stored:** Source is database read, file read, or storage mechanism
- **DOM-based:** Sink is client-side JavaScript (no server-side source)

**Step 5: Verify Externally Exploitable**
- Must be reachable via public internet
- Exclude internal-only sinks

##### Queue Format

```json
{
  "ID": "XSS-VULN-01",
  "vulnerability_type": "Reflected | Stored | DOM-based",
  "externally_exploitable": true,
  "source": "parameter/header/storage mechanism",
  "source_detail": "DB read operation (Stored) or HTTP input (Reflected/DOM)",
  "path": "complete data flow from source to sink",
  "sink_function": "function or template construct",
  "render_context": "HTML_BODY | HTML_ATTRIBUTE | JAVASCRIPT_STRING | URL_PARAM | CSS_VALUE",
  "encoding_observed": "name & file:line (or 'None')",
  "verdict": "vulnerable",
  "mismatch_reason": "why encoding is wrong for context",
  "witness_payload": "minimal payload proving context control",
  "confidence": "high | med | low",
  "notes": "CSP, HttpOnly, WAF behavior"
}
```

##### Deliverables

1. **`deliverables/xss_analysis_deliverable.md`** - Analysis with secure vectors
2. **`deliverables/xss_exploitation_queue.json`** - Only `externally_exploitable: true`

---

#### Agent 3: Authentication Analysis Specialist

**Prompt File:** `vuln-auth.txt` (150+ lines)
**Role:** Authentication Analysis Specialist
**Vulnerability Types:** Authentication bypass, session flaws, login logic issues

##### Purpose

Identify **logical flaws in identity verification** - where the app fails to answer "Are you who you say you are?"

##### Methodology Categories

**1) Transport & Caching**
- Enforce HTTPS (no HTTP fallbacks)
- Verify HSTS
- Check `Cache-Control: no-store` / `Pragma: no-cache` on auth responses

**2) Rate Limiting / CAPTCHA / Monitoring**
- Per-IP and/or per-account rate limits on login, signup, reset
- Lockout/backoff or CAPTCHA on repeated failures
- Monitoring/alerting for failed login spikes

**3) Session Management (Cookies)**
- `HttpOnly`, `Secure`, `SameSite` flags
- Session ID rotation after login
- Logout invalidates server-side session
- Idle and absolute timeouts
- No session IDs in URLs

**4) Token/Session Properties**
- Cryptographic randomness (no sequential IDs)
- HTTPS-only transmission
- No logging of tokens
- Explicit expiration (TTL)
- Invalidation on logout

**5) Session Fixation**
- Compare pre-login vs post-login session identifiers
- Require new ID on auth success

**6) Password & Account Policy**
- No default credentials in code/fixtures
- Password complexity requirements
- Password change verification
- Account enumeration prevention

##### Queue Format

```json
{
  "ID": "AUTH-VULN-01",
  "vulnerability_type": "Authentication_Bypass | Session_Management_Flaw | Login_Flow_Logic | Token_Management_Issue | Reset_Recovery_Flaw | Transport_Exposure | Abuse_Defenses_Missing | OAuth_Flow_Issue",
  "externally_exploitable": true,
  "source_endpoint": "POST /api/auth/login",
  "vulnerable_code_location": "file:line of flawed logic",
  "missing_defense": "No rate limit on POST /login",
  "exploitation_hypothesis": "Attacker can successfully log into an account by guessing simple password",
  "suggested_exploit_technique": "brute_force_login | credential_stuffing | session_hijacking | session_fixation",
  "confidence": "High | Medium | Low",
  "notes": "session state, roles, headers"
}
```

##### Deliverables

1. **`deliverables/auth_analysis_deliverable.md`** - Analysis with secure mechanisms
2. **`deliverables/auth_exploitation_queue.json`** - Only `externally_exploitable: true`

---

#### Agent 4: Authorization Analysis Specialist

**Prompt File:** `vuln-authz.txt` (150+ lines)
**Role:** Authorization Analysis Specialist
**Vulnerability Types:** Horizontal, Vertical, Context-based authorization flaws

##### Purpose

Identify **logical flaws in access control** - where the app fails to answer "Are you allowed to do this?"

##### Methodology Categories

**1) Horizontal Authorization**
**Source:** `deliverables/recon_deliverable.md` Section 8.1

Create todo for each candidate:
- Start at endpoint
- Trace backward to **sufficient guard** or **side effect without guard**
- **Side Effect:** Unauthorized read/write of sensitive resources
  - Database read/write of user/org objects
  - File or config changes
  - Cross-tenant data access/modification
  - Shared resource leakage between tenants
- **Sufficient Guard (horizontal):**
  - Session authentication
  - Bind identifier → current user/org (ownership check)
  - Tenant/org membership validation
  - User/org/tenant identifier filtering in queries
  - Prevents identifier manipulation
  - Maintains shared service isolation
  - Runs before side effect

**2) Vertical Authorization**
**Source:** `deliverables/recon_deliverable.md` Section 8.2

Create todo for each candidate:
- Start at privileged endpoint
- Trace backward to **sufficient guard** or **side effect without guard**
- **Sufficient Guard (vertical):**
  - Session authentication
  - Role/privilege verification (higher than current user)
  - Permission/permission group check
  - Runs before side effect

**3) Context-Based Authorization**
**Source:** `deliverables/recon_deliverable.md` Section 8.3

Create todo for each candidate:
- Start at workflow endpoint
- Trace backward to **sufficient guard** or **side effect without guard**
- **Sufficient Guard (context):**
  - Session authentication
  - Workflow state verification
  - Step completion checks
  - Runs before side effect

##### Queue Format

```json
{
  "ID": "AUTHZ-VULN-01",
  "vulnerability_type": "Horizontal | Vertical | Context_Workflow",
  "externally_exploitable": true,
  "endpoint": "GET /api/users/{user_id}",
  "vulnerable_code_location": "file:line where guard is missing/misplaced",
  "role_context": "role(s) that can trigger this",
  "guard_evidence": "no ownership check | role check after database write",
  "side_effect": "read other users profiles | delete any file",
  "reason": "1-2 lines explaining vulnerability",
  "minimal_witness": "change user_id param to another user",
  "confidence": "high | med | low",
  "notes": "assumptions, related findings"
}
```

##### Deliverables

1. **`deliverables/authz_analysis_deliverable.md`** - Analysis with secure controls
2. **`deliverables/authz_exploitation_queue.json`** - Only `externally_exploitable: true`

---

#### Agent 5: SSRF Analysis Specialist

**Prompt File:** `vuln-ssrf.txt` (150+ lines)
**Role:** SSRF Analysis Specialist
**Vulnerability Types:** URL manipulation, redirect abuse, webhook injection, API proxy bypass

##### Purpose

Identify where **user input influences outbound HTTP requests** without proper validation or restrictions.

##### Methodology Categories

**1) Identify HTTP Client Usage Patterns**
- Trace URL/callback/webhook/file path parameters to HTTP clients
- Look for: requests, urllib, axios, fetch, HttpClient, etc.
- Identify: URL fetching, image processing, webhook calls, API proxying

**2) Protocol and Scheme Validation**
- Verify protocol allowlisting (https://, http://)
- Check for dangerous schemes blocked: file://, ftp://, gopher://, dict://, ldap://

**3) Hostname and IP Address Validation**
- Block internal/private IP ranges: 127.0.0.0/8, 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
- Verify hostname allowlisting vs blocklisting
- Protect against DNS rebinding and localhost access

**4) Port Restriction and Service Access Controls**
- Restrict to approved ports (80, 443, 8080, 8443)
- Block internal service ports (22, 23, 25, 53, 135, 445, 993, 995)
- Block cloud metadata endpoints (169.254.169.254, metadata.google.internal)

**5) URL Parsing and Validation Bypass**
- Test for bypass techniques: encoding, double encoding, Unicode normalization
- Check redirect following behavior
- Verify handling of malformed URLs, IPv6, international domain names

##### Queue Format

```json
{
  "ID": "SSRF-VULN-01",
  "vulnerability_type": "URL_Manipulation | Redirect_Abuse | Webhook_Injection | API_Proxy_Bypass | File_Fetch_Abuse | Service_Discovery",
  "externally_exploitable": true,
  "source_endpoint": "POST /api/fetch-url",
  "vulnerable_parameter": "url",
  "vulnerable_code_location": "file:line where HTTP client makes request",
  "missing_defense": "No URL allowlist validation",
  "exploitation_hypothesis": "Attacker can access internal services by providing internal URLs",
  "suggested_exploit_technique": "internal_service_access | cloud_metadata_retrieval | port_scanning | webhook_abuse",
  "confidence": "High | Medium | Low",
  "notes": "request format, auth requirements, timeouts"
}
```

##### Deliverables

1. **`deliverables/ssrf_analysis_deliverable.md`** - Analysis with secure controls
2. **`deliverables/ssrf_exploitation_queue.json`** - Only `externally_exploitable: true`

---

### Phase 4: Exploitation (5 Parallel Agents)

#### Overview

Five specialized agents **weaponize** vulnerabilities from analysis queues. Each agent:
1. Reads **three** intelligence files (pre-recon, recon, analysis)
2. Processes **every** vulnerability in their queue via TodoWrite
3. Pursues each to definitive conclusion (EXPLOITED or proven NOT EXPLOITABLE)
4. Creates **one** deliverable: evidence report

#### Common Exploitation Prompt Structure

All 5 exploitation agents share this structure:

```xml
<role>
You are a world-class [VULN TYPE] Exploitation Specialist.
You are not an analyst; you are an active penetration tester.
</role>

<critical>
**PROOF-BASED EXPLOITATION REQUIREMENTS**
- Claims require evidence
- Assumptions must be validated
- Failures must be documented
- Default to skepticism
- Partial results are not complete

**EXPLOITATION TARGET REQUIREMENT**
All exploitation must be performed against: {{WEB_URL}}

**EXPLOIT DEFINITION**
An exploit is executing an unauthorized or unintended action
against the target application {{WEB_URL}}.

**Proof Levels:**
Level 1 - Weakness identified (POTENTIAL - Low)
Level 2 - Partial bypass (POTENTIAL - Medium)
Level 3 - Exploitation confirmed (EXPLOITED)
Level 4 - Critical impact demonstrated (EXPLOITED - CRITICAL)

**MUST reach Level 3+ to mark as EXPLOITED**
</critical>

<starting_context>
**ACTIONABLE EXPLOITATION QUEUE:**
deliverables/[type]_exploitation_queue.json

**INTELLIGENCE GATHERING (MANDATORY):**
1. deliverables/pre_recon_deliverable.md - Architecture overview
2. deliverables/recon_deliverable.md - API inventory, input vectors
3. deliverables/[type]_analysis_deliverable.md - Strategic intel

**TODOWRITE TRACKING:**
Manage work with TodoWrite tool - one task per vulnerability
</starting_context>
```

---

#### Agent 1: Injection Exploitation Specialist

**Prompt File:** `exploit-injection.txt` (200+ lines)
**Role:** Injection Exploitation Specialist
**Queue:** `deliverables/injection_exploitation_queue.json`

##### Purpose

**Systematically and exhaustively weaponize** every SQL Injection and Command Injection vulnerability to achieve data exfiltration (SQLi) or remote code execution (Command Injection).

##### Methodology: OWASP Exploitation Workflow

**TodoWrite-Driven Execution:**

**1. Initialization**
- Read queue file
- Create todo for each vulnerability with hypothesis

**2. Execution Loop** (Repeat until all tasks completed):

**A. Get Next Task**
- Review todo list, pick next pending task

**B. Plan & Decompose**
- Mark task as in_progress
- Decompose complex tasks into smaller steps

**C. Execute**
- Run command (curl, sqlmap, custom script)

**D. Update & Document**
- Mark completed if successful
- Document outcome (exact command, result)

**E. Expand Plan**
- If successful, add next logical steps (OWASP workflow)

**Stage 1: Confirmation & Probing**
- Inject error-inducing characters: `'`, `"`, `)`, `;`, `\`
- Boolean toggles: `AND 1=1` vs `AND 1=2`
- Time delays: `AND SLEEP(5)`

**Stage 2: Fingerprinting & Enumeration**
- Database version, current user
- List all tables
- Identify sensitive table
- List column names

**Stage 3: Targeted Exfiltration**
- Extract first 5 rows from sensitive table
- Achieve impact goal

##### Tool Strategy

- **`curl`** - Manual probing, initial confirmation, WAF bypass crafting
- **`sqlmap`** - Automation after manual confirmation, blind injections
- **Task Agent** - Custom scripting for payload loops, enumeration workflows

##### Task Agent Template

```
Role: Security testing script writer
Write and execute Python script to probe injection

Inputs:
1. Target URL: [endpoint]
2. Vulnerability details: [from queue]
3. Payload variations: [list]

Success criteria: [define evidence to collect]
```

##### Deliverable

**`deliverables/injection_exploitation_evidence.md`** - Evidence for every vulnerability:
- Successfully exploited with payloads, commands, exfiltrated data
- Proven non-vulnerable with full log of failed attempts

---

#### Agent 2: XSS Exploitation Specialist

**Prompt File:** `exploit-xss.txt` (150+ lines)
**Role:** XSS Exploitation Specialist
**Queue:** `deliverables/xss_exploitation_queue.json`

##### Purpose

**Systematically and exhaustively weaponize** every XSS vulnerability to achieve JavaScript execution and session hijacking.

##### Proof of Exploitation Levels

**Level 1 - Payload Blocked/Encoded**
- Evidence: Payload reflected but encoded/sanitized
- Classification: NOT VULNERABLE

**Level 2 - Payload Injected But Blocked**
- Evidence: Payload in HTML but execution prevented
- Classification: POTENTIAL (requires bypass)

**Level 3 - JavaScript Execution Confirmed**
- Evidence: alert(), console.log, DOM manipulation
- Classification: EXPLOITED ✓

**Level 4 - Critical Impact Demonstrated**
- Evidence: Session hijacking, data theft, unauthorized actions
- Classification: EXPLOITED (CRITICAL) ✓

**MUST reach Level 3+ to mark as EXPLOITED**

##### Methodology

**Per-Vulnerability Workflow:**

1. **Initial Payload Testing**
   - Use `witness_payload` from analysis queue
   - Test with Playwright (DOM-based) or curl (reflected)
   - Verify actual JavaScript execution (not just reflection)

2. **Defense Analysis**
   - Check CSP headers (Content-Security-Policy)
   - Identify WAF patterns
   - Note input sanitization

3. **Bypass Attempts** (systematic escalation)
   - Encoding variants (URL encode, double encode, Unicode)
   - Case variations
   - Comment insertion
   - Alternative event handlers
   - DOM-based bypasses

4. **Impact Demonstration**
   - Steal session cookie
   - Perform unauthorized action
   - Exfiltrate data
   - Hijack user session

##### Tool Strategy

- **Playwright** - Primary tool for DOM-based and Stored XSS, actual browser execution
- **curl** - Reflected XSS testing, raw HTML response inspection
- **Task Agent** - Payload iteration scripts, exfiltration listeners, DOM helpers

##### Deliverable

**`deliverables/xss_exploitation_evidence.md`** - Evidence for every vulnerability:
- Successfully exploited with payloads, screenshots, stolen cookies
- Proven blocked with bypass attempts log

---

#### Agent 3: Authentication Exploitation Specialist

**Prompt File:** `exploit-auth.txt` (150+ lines)
**Role:** Authentication Exploitation Specialist
**Queue:** `deliverables/auth_exploitation_queue.json`

##### Purpose

**Systematically and exhaustively weaponize** every authentication flaw to achieve account takeover and session hijacking.

##### Proof of Exploitation Levels

**Level 1 - Authentication Weakness Identified**
- Evidence: Theoretical bypass found
- Classification: POTENTIAL (Low)

**Level 2 - Partial Bypass Achieved**
- Evidence: Access to non-critical resources
- Classification: POTENTIAL (Medium)

**Level 3 - Authentication Bypass Confirmed**
- Evidence: Access to protected functionality as unauthorized user
- Classification: EXPLOITED ✓

**Level 4 - Account Takeover Demonstrated**
- Evidence: Complete user impersonation
- Classification: EXPLOITED (CRITICAL) ✓

**MUST reach Level 3+ to mark as EXPLOITED**

##### What "Exploit" Means

An exploit is an **ACTIVE ATTACK**, not just confirmation:

- "No rate limiting" → Attempt brute force with many requests
- "Weak password policy" → Create weak accounts + attempt access
- "User enumeration" → Build valid user list
- "Missing security headers" → Abuse the missing protection

**Chain exploits:** Use enumerated users in password attacks

##### Methodology

**Per-Vulnerability Workflow:**

1. **Setup**
   - Authenticate if needed (use provided credentials)
   - Establish baseline behavior
   - Identify bypass mechanism

2. **Attack Execution**
   - Execute suggested technique from queue
   - Adapt based on observed response
   - Escalate if initial attempts blocked

3. **Verification**
   - Confirm unauthorized access
   - Demonstrate impact
   - Document proof (requests, responses, session tokens)

##### Tool Strategy

- **Playwright** - Multi-step auth flows, session hijacking, account takeover verification
- **curl** - Session replay, token testing, brute force attempts
- **Task Agent** - Enumeration scripts, brute force loops, timing attack automation

##### Deliverable

**`deliverables/auth_exploitation_evidence.md`** - Evidence for every vulnerability:
- Successfully bypassed auth with payloads, captured sessions
- Proven secure with exhaustive bypass attempts

---

#### Agent 4: Authorization Exploitation Specialist

**Prompt File:** `exploit-authz.txt` (150+ lines)
**Role:** Authorization Exploitation Specialist
**Queue:** `deliverables/authz_exploitation_queue.json`

##### Purpose

**Systematically and exhaustively weaponize** every authorization flaw to achieve privilege escalation and unauthorized data access.

##### Proof of Exploitation Levels

**Level 1 - Authorization Weakness Identified**
- Evidence: Theoretical bypass found
- Classification: POTENTIAL (Low)

**Level 2 - Partial Access Control Bypass**
- Evidence: Access to some protected resources
- Classification: POTENTIAL (Medium)

**Level 3 - Access Control Bypass Confirmed**
- Evidence: Access to protected functionality/data as unauthorized user
- Classification: EXPLOITED ✓

**Level 4 - Critical Privilege Escalation Demonstrated**
- Evidence: Complete privilege escalation to admin
- Classification: EXPLOITED (CRITICAL) ✓

**MUST reach Level 3+ to mark as EXPLOITED**

##### Methodology by Vulnerability Type

**Horizontal Privilege Escalation**
- Use valid user credentials
- Manipulate object IDs (user_id, order_id, etc.)
- Attempt to access other users' resources
- Verify unauthorized access

**Vertical Privilege Escalation**
- Use low-privilege user credentials
- Attempt to access admin/privileged endpoints
- Verify privilege escalation
- Demonstrate admin functionality access

**Context-Based Bypass**
- Skip workflow steps
- Access endpoints out of order
- Verify state violation
- Demonstrate workflow bypass

##### Tool Strategy

- **Playwright** - Complex authorization flows, role-based access control verification
- **curl** - IDOR sweeps, parameter manipulation, endpoint access attempts
- **Task Agent** - Multi-user iteration, role escalation loops, workflow automation

##### Task Agent Template

```
Role: Authorization testing script writer
Write script to test horizontal privilege escalation

Inputs:
1. Base URL: [target]
2. Vulnerability: [from queue]
3. User accounts: [credentials]
4. Target IDs: [list to test]

Success criteria: [define unauthorized access]
```

##### Deliverable

**`deliverables/authz_exploitation_evidence.md`** - Evidence for every vulnerability:
- Successfully bypassed access controls with requests, responses
- Proven secure with systematic testing log

---

#### Agent 5: SSRF Exploitation Specialist

**Prompt File:** `exploit-ssrf.txt` (similar structure to other exploit agents)
**Role:** SSRF Exploitation Specialist
**Queue:** `deliverables/ssrf_exploitation_queue.json`

##### Purpose

**Systematically and exhaustively weaponize** every SSRF vulnerability to achieve internal service access and cloud metadata retrieval.

##### Methodology

**Per-Vulnerability Workflow:**

1. **Internal Service Access**
   - Attempt to access localhost: admin panels, databases, caches
   - Test internal IP ranges: 127.0.0.0/8, 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
   - Verify successful connection (timeouts, responses)

2. **Cloud Metadata Retrieval**
   - Attempt to access cloud metadata endpoints
   - AWS: 169.254.169.254/latest/meta-data/
   - GCP: metadata.google.internal/computeMetadata/v1/
   - Verify credentials exposure

3. **Port Scanning**
   - Scan common internal service ports
   - Identify open services
   - Map internal infrastructure

4. **File Access**
   - Attempt to read local files via file:// URLs
   - Access configuration files, credentials

##### Tool Strategy

- **curl** - Primary tool for SSRF testing with custom URLs
- **Playwright** - Browser-based SSRF if application uses client-side requests
- **Task Agent** - Port scanning scripts, metadata extraction automation

##### Deliverable

**`deliverables/ssrf_exploitation_evidence.md`** - Evidence for every vulnerability:
- Successfully accessed internal services with URLs, responses
- Proven blocked with systematic testing attempts

---

### Phase 5: Reporting (Final Synthesis)

**Prompt File:** `report-executive.txt` (127 lines)
**Agent Name:** Report Agent
**Role:** Executive Summary Writer and Report Cleaner

#### Purpose

**MODIFIES** the concatenated report (not creates new) by:
1. Adding Executive Summary and Network Reconnaissance sections at the top
2. **Cleaning all hallucinated content** (removing sections without vulnerability IDs)
3. Preserving only exploitation evidence with proper IDs

#### Input Files

```
1. deliverables/pre_recon_deliverable.md - Initial recon
2. deliverables/recon_deliverable.md - Attack surface mapping
3. deliverables/comprehensive_security_assessment_report.md - TO MODIFY
```

#### Modification Instructions

**ADD at Top:**

```markdown
# Security Assessment Report

## Executive Summary
- Target: {{WEB_URL}}
- Assessment Date: [current date]
- Scope: Auth, XSS, Injection, SSRF, Authz testing

## Summary by Vulnerability Type

**Authentication Vulnerabilities:**
[Check "Authentication Exploitation Evidence" section]
[Summarize exploited + blocked by security controls]
[If none: "No authentication vulnerabilities were found."]

**Authorization Vulnerabilities:**
[Check "Authorization Exploitation Evidence" section]
[Same format]

**Cross-Site Scripting (XSS) Vulnerabilities:**
[Check "XSS Exploitation Evidence" section]
[Same format]

**SQL/Command Injection Vulnerabilities:**
[Check "Injection Exploitation Evidence" section]
[Same format]

**Server-Side Request Forgery (SSRF) Vulnerabilities:**
[Check "SSRF Exploitation Evidence" section]
[Same format]

## Network Reconnaissance
[Extract from nmap, subfinder, whatweb]
[Open ports, subdomains, security headers]
[SKIP stack details]
```

**KEEP existing sections (in original order):**

- `# [Type] Exploitation Evidence`
- `## Successfully Exploited Vulnerabilities`
- Vulnerability lists with IDs matching: `### [TYPE]-VULN-[NUMBER]`

**REMOVE all other sections:**

- `## Potential Vulnerabilities (Validation Blocked)`
- "Recommendations", "Conclusion", "Summary", "Next Steps"
- "Additional Analysis", false positives sections
- Meta-commentary, intros, counts

#### Deliverable

**`deliverables/comprehensive_security_assessment_report.md`** (modified in-place)

**Final Structure:**
```
1. Executive Summary (NEW)
2. Network Reconnaissance (NEW)
3. Authentication Exploitation Evidence (CLEANED)
4. Authorization Exploitation Evidence (CLEANED)
5. XSS Exploitation Evidence (CLEANED)
6. Injection Exploitation Evidence (CLEANED)
7. SSRF Exploitation Evidence (CLEANED)
```

---

## Cross-Phase Coordination Patterns

### Pattern 1: Cascade Impact Awareness

Every vulnerability analysis prompt emphasizes that their work cascades through all subsequent agents.

**Example from Injection Analysis:**
```xml
<critical>
**Cascade Impact:** An incomplete analysis creates blind spots that persist through all 10+ subsequent agents.
**Thoroughness is Non-Negotiable:** Do NOT terminate early.
</critical>
```

**Pattern:**
- **Pre-Recon** → Creates baseline for 10+ agents
- **Recon** → Maps attack surface for 5 vuln agents
- **Vuln Agent** → Identifies targets for corresponding exploit agent
- **Exploit Agent** → Provides evidence for final report
- **Report Agent** → Synthesizes all findings into executive summary

---

### Pattern 2: Shared Intelligence Handoff

**Every phase reads outputs from ALL previous phases:**

**Vulnerability Agents read:**
```
1. deliverables/pre_recon_deliverable.md
2. deliverables/recon_deliverable.md
```

**Exploitation Agents read:**
```
1. deliverables/pre_recon_deliverable.md - Architecture
2. deliverables/recon_deliverable.md - API inventory
3. deliverables/[type]_analysis_deliverable.md - Strategic intel
```

**Report Agent reads:**
```
1. deliverables/pre_recon_deliverable.md - Network recon
2. deliverables/recon_deliverable.md - Attack surface
3. deliverables/comprehensive_security_assessment_report.md - TO MODIFY
```

**Coordination Benefit:** Later agents have complete context without re-analyzing.

---

### Pattern 3: Queue-Based Validation

**Critical Design Decision:** Vulnerability agents create **two** deliverables:

1. **Analysis Report (Markdown)** - Documents ALL findings including secure vectors
2. **Exploitation Queue (JSON)** - ONLY `externally_exploitable: true` vulnerabilities

**Queue Structure:**
```json
{
  "vulnerabilities": [
    {
      "ID": "TYPE-VULN-NN",
      "externally_exploitable": true,  // GATEKEEPER FIELD
      "verdict": "vulnerable",
      "confidence": "high|med|low",
      ... // vulnerability-specific fields
    }
  ]
}
```

**Validation Logic (from `src/queue-validation.ts`):**
```typescript
checkExploitationQueue() {
  const queueData = JSON.parse(queueFile);
  const vulnCount = queueData.vulnerabilities.length;

  if (vulnCount === 0) {
    return { shouldExploit: false, shouldRetry: false };
  }

  return {
    shouldExploit: true,
    vulnerabilityCount: vulnCount,
    vulnType
  };
}
```

**Result:** Exploitation agents skip entire phase if no vulnerabilities found in queue.

---

### Pattern 4: File:line Location Passing

**All prompts require exact code locations for handoff:**

**Pre-Recon → Recon:**
```markdown
## 8. Critical File Paths
- Authentication & Authorization: `auth/jwt_middleware.go:45`, `internal/user/permissions.go:123`
- API & Routing: `cmd/api/main.go:89`, `internal/handlers/user_routes.go:56`
```

**Recon → Vuln Agents:**
```markdown
## API Endpoint Inventory
| Method | Endpoint | Code Pointer |
|---|---|---|
| GET | /api/users/{id} | See `users.controller.ts:145` |
```

**Vuln Agent → Exploit Agent:**
```json
{
  "vulnerable_code_location": "controllers/user.go:145",
  "sink_call": "db/query.go:234 (db.Query)"
}
```

**Benefit:** Downstream agents don't waste time searching for vulnerable code.

---

### Pattern 5: Scope Boundary Reinforcement

**Every prompt reinforces network-accessible scope:**

**Shared Partial (`_vuln-scope.txt`):**
```markdown
**EXTERNAL ATTACKER SCOPE:** Only report vulnerabilities exploitable
via {{WEB_URL}} from the internet. Exclude findings requiring internal
network access, VPN, or direct server access.
```

**Agent Prompts Reference:**
```xml
<scope>
@include(shared/_vuln-scope.txt)
</scope>
```

**Queue Field:**
```json
{
  "externally_exploitable": true  // GATEKEEPER
}
```

**Result:** Internal-only findings never reach exploitation or final report.

---

### Pattern 6: Proof Level Escalation

**Exploitation prompts define 4-level proof framework:**

**Example from XSS Exploitation:**
```
Level 1 - Payload Blocked/Encoded → NOT VULNERABLE
Level 2 - Injected But Blocked → POTENTIAL (Medium)
Level 3 - JavaScript Execution Confirmed → EXPLOITED ✓
Level 4 - Critical Impact Demonstrated → EXPLOITED (CRITICAL) ✓

**MUST reach Level 3+ to mark as EXPLOITED**
```

**Coordination Benefit:** Clear threshold prevents false positives in final report.

---

### Pattern 7: Task Agent Workflow Coordination

**Different agents use Task agents differently:**

**Pre-Recon (Phased Parallel):**
```
Phase 1: Launch 3 agents → Wait for ALL → Mark Phase 1 complete
Phase 2: Launch 3 agents → Wait for ALL → Mark Phase 2 complete
Phase 3: Synthesis
```

**Recon (Parallel Discovery):**
```
Launch 5 agents IN PARALLEL → Wait for ALL → Synthesize
```

**Vuln Agents (Sequential Analysis):**
```
For each source → Launch 1 Task Agent → Wait → Document → Next source
```

**Exploit Agents (TodoWrite-Driven):**
```
Create todo for each vulnerability → Process sequentially → Mark completed
```

**Benefit:** Workflow optimized for each phase's specific needs.

---

## Data Flow and Handoff Mechanisms

### Complete Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│ PHASE 1: PRE-RECONNAISSANCE (Code Analysis)                        │
│                                                                     │
│ Input: Source code (cloned repo)                                    │
│ External Tools: nmap, subfinder, whatweb                           │
│ Task Agents: 6 (3 phases: discovery → vuln → synthesis)            │
│                                                                     │
│ Output:                                                              │
│   • deliverables/code_analysis_deliverable.md                      │
│   • outputs/schemas/ (API schema files)                            │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────┐
│ PHASE 2: RECONNAISSANCE (Attack Surface Mapping)                   │
│                                                                     │
│ Input:                                                              │
│   • deliverables/code_analysis_deliverable.md                      │
│   • Live application (via Playwright)                               │
│ Task Agents: 5 (parallel code correlation)                         │
│                                                                     │
│ Output:                                                              │
│   • deliverables/recon_deliverable.md                               │
│     (API inventory, auth architecture, role hierarchy,             │
│      authorization candidates, injection sources)                  │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
                ┌──────────┴──────────┐
                │                     │
                ▼                     ▼
┌───────────────────────────┐  ┌───────────────────────────┐
│   VULN AGENTS (5 Parallel) │  │   VULN AGENTS (5 Parallel) │
│                           │  │                           │
│ 1. Injection Vuln         │  │ 2. XSS Vuln               │
│    Task Agents: 1-2       │  │    Task Agents: 1-2       │
│    Output:                 │  │    Output:                 │
│      • analysis.md        │  │      • analysis.md        │
│      • queue.json         │  │      • queue.json         │
│                           │  │                           │
│ 3. Auth Vuln              │  │ 4. Authz Vuln             │
│    Task Agents: 1         │  │    Task Agents: 1         │
│    Output:                 │  │    Output:                 │
│      • analysis.md        │  │      • analysis.md        │
│      • queue.json         │  │      • queue.json         │
│                           │  │                           │
│ 5. SSRF Vuln              │  │                           │
│    Task Agents: 1         │  │                           │
│    Output:                 │  │                           │
│      • analysis.md        │  │                           │
│      • queue.json         │  │                           │
└───────────┬───────────────┘  └───────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────────────────────┐
│ QUEUE VALIDATION GATE                                               │
│                                                                     │
│ For each vuln type:                                                 │
│   IF queue.json has vulnerabilities > 0                            │
│      THEN launch corresponding exploit agent                        │
│      ELSE skip exploitation (mark as complete)                      │
└──────────────────────────┬──────────────────────────────────────────┘
                           │
                ┌──────────┴──────────┐
                ▼                     ▼
┌───────────────────────────┐  ┌───────────────────────────┐
│  EXPLOIT AGENTS (5 Cond.) │  │  EXPLOIT AGENTS (5 Cond.) │
│                           │  │                           │
│ 1. Injection Exploit      │  │ 2. XSS Exploit            │
│    Reads: 3 intel files   │  │    Reads: 3 intel files   │
│    Workflow: TodoWrite    │  │    Workflow: TodoWrite    │
│    Output: evidence.md    │  │    Output: evidence.md    │
│                           │  │                           │
│ 3. Auth Exploit           │  │ 4. Authz Exploit          │
│    Reads: 3 intel files   │  │    Reads: 3 intel files   │
│    Workflow: TodoWrite    │  │    Workflow: TodoWrite    │
│    Output: evidence.md    │  │    Output: evidence.md    │
│                           │  │                           │
│ 5. SSRF Exploit           │  │                           │
│    Reads: 3 intel files   │  │                           │
│    Workflow: TodoWrite    │  │                           │
│    Output: evidence.md    │  │                           │
└───────────┬───────────────┘  └───────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────────────────────┐
│ PHASE 5: REPORTING (Executive Summary)                              │
│                                                                     │
│ Input:                                                              │
│   • All analysis deliverables (5 md files)                         │
│   • All exploitation evidence (5 md files)                         │
│   • Concatenated report (already created)                          │
│                                                                     │
│ Task:                                                                │
│   1. ADD Executive Summary                                          │
│   2. ADD Network Recon                                             │
│   3. CLEAN hallucinated content                                     │
│   4. REMOVE sections without vulnerability IDs                     │
│                                                                     │
│ Output:                                                              │
│   • deliverables/comprehensive_security_assessment_report.md      │
│     (modified in-place)                                             │
└─────────────────────────────────────────────────────────────────────┘
```

### Handoff Mechanisms

#### **Mechanism 1: File-Based Deliverables**

**All coordination happens via files in `deliverables/`:**

```
deliverables/
├── code_analysis_deliverable.md           # Phase 1
├── recon_deliverable.md                   # Phase 2
│
├── injection_analysis_deliverable.md      # Phase 3a
├── injection_exploitation_queue.json      # Phase 3a → 4
│
├── xss_analysis_deliverable.md            # Phase 3a
├── xss_exploitation_queue.json            # Phase 3a → 4
│
├── auth_analysis_deliverable.md           # Phase 3a
├── auth_exploitation_queue.json           # Phase 3a → 4
│
├── authz_analysis_deliverable.md          # Phase 3a
├── authz_exploitation_queue.json          # Phase 3a → 4
│
├── ssrf_analysis_deliverable.md           # Phase 3a
├── ssrf_exploitation_queue.json           # Phase 3a → 4
│
├── injection_exploitation_evidence.md     # Phase 4
├── xss_exploitation_evidence.md           # Phase 4
├── auth_exploitation_evidence.md          # Phase 4
├── authz_exploitation_evidence.md         # Phase 4
├── ssrf_exploitation_evidence.md          # Phase 4
│
└── comprehensive_security_assessment_report.md  # Phase 5
```

#### **Mechanism 2: MCP Tool Validation**

**`save_deliverable` MCP tool enforces correct file naming and structure:**

```typescript
// From mcp-server/src/tools/save-deliverable.ts
save_deliverable(deliverable_type, content) {
  // Auto-generates correct filename
  const filenames = {
    'CODE_ANALYSIS': 'code_analysis_deliverable.md',
    'RECON': 'recon_deliverable.md',
    'INJECTION_ANALYSIS': 'injection_analysis_deliverable.md',
    'INJECTION_QUEUE': 'injection_exploitation_queue.json',
    // ... etc
  };

  // Validates queue structure
  if (deliverable_type.endsWith('_QUEUE')) {
    const data = JSON.parse(content);
    if (!data.vulnerabilities || !Array.isArray(data.vulnerabilities)) {
      return { status: 'error', errorType: 'validation' };
    }
  }

  // Atomic write to deliverables/
  fs.writeFile(`deliverables/${filenames[deliverable_type]}`, content);

  return { status: 'success', filepath: `deliverables/${filenames[deliverable_type]}` };
}
```

#### **Mechanism 3: Variable Substitution

**Prompts use template variables injected at runtime:**

```bash
# From src/temporal/activities.ts
const prompt = loadPrompt('vuln-injection.txt')
  .replace('{{WEB_URL}}', config.webUrl)
  .replace('{{RULES_AVOID}}', config.rules.avoid.join('\n'))
  .replace('{{RULES_FOCUS}}', config.rules.focus.join('\n'))
  .replace('{{LOGIN_INSTRUCTIONS}}', generateLoginInstructions(config))
  .replace('{{MCP_SERVER}}', assignMcpServer(agentName));
```

**Shared partials use `@include` directive:**
```xml
<scope>
@include(shared/_vuln-scope.txt)
</scope>
```

#### **Mechanism 4: Queue Validation Logic

**From `src/queue-validation.ts`:**

```typescript
export async function checkExploitationQueue(
  sourceDir: string,
  vulnType: VulnType
): Promise<SafeValidationResult> {
  const queueFile = getQueuePath(vulnType); // e.g., injection_exploitation_queue.json
  const queuePath = path.join(sourceDir, 'deliverables', queueFile);

  // Check queue exists
  if (!fs.existsSync(queuePath)) {
    return { success: false, error: 'Queue file not found' };
  }

  // Parse and validate
  const queueData = JSON.parse(fs.readFileSync(queuePath, 'utf-8'));
  const vulnerabilities = queueData.vulnerabilities || [];

  // Check count
  if (vulnerabilities.length === 0) {
    return { success: true, data: { shouldExploit: false } };
  }

  // Return exploitation decision
  return {
    success: true,
    data: {
      shouldExploit: true,
      vulnerabilityCount: vulnerabilities.length,
      vulnType
    }
  };
}
```

**Called from workflow in `src/temporal/workflows.ts`:**
```typescript
const decision = await a.checkExploitationQueue(activityInput, vulnType);

if (decision.shouldExploit) {
  const exploitMetrics = await runExploitAgent(activityInput);
} else {
  // Skip exploitation
  state.completedAgents.push(`${vulnType}-exploit`);
}
```

---

## Shared Partial Templates

### `_target.txt`

**Purpose:** Inject target URL into prompts.

**Content:**
```markdown
URL: {{WEB_URL}}
```

**Usage:** All 13 agent prompts.

---

### `_rules.txt`

**Purpose:** Inject user-configured avoid/focus rules from YAML config.

**Content:**
```markdown
Rules to Avoid:
{{RULES_AVOID}}
```

**Example:**
```yaml
# From config
rules:
  avoid:
    - description: "Don't test logout functionality"
      type: path
      url_path: "/logout"
```

**Usage:** All vulnerability and exploitation agents.

---

### `_vuln-scope.txt`

**Purpose:** Define external attacker scope - internet-only, no internal access.

**Content:**
```markdown
**EXTERNAL ATTACKER SCOPE:** Only report vulnerabilities exploitable
via {{WEB_URL}} from the internet. Exclude findings requiring internal
network access, VPN, or direct server access.
```

**Usage:** All 5 vulnerability analysis agents.

**Impact:** Ensures only network-reachable vulnerabilities enter queues.

---

### `_exploit-scope.txt`

**Purpose:** Define exploitation constraints and verdict classifications.

**Content:**
```markdown
**MANDATORY EXPLOITATION CONSTRAINTS:**
- ALL exploitation attempts MUST be performed against {{WEB_URL}} from external network
- DO NOT attempt to exploit vulnerabilities that require:
  - Internal network access (Tailscale, VPN, etc.)
  - Direct server access
  - Database console access

**VERDICT CLASSIFICATIONS:**
- EXPLOITED: Successfully demonstrated impact via public interface
- BLOCKED_BY_SECURITY: Valid vulnerability but blocked by WAF/controls
- OUT_OF_SCOPE_INTERNAL: Requires internal access - DO NOT PURSUE
- FALSE_POSITIVE: Not actually vulnerable

Only EXPLOITED and BLOCKED_BY_SECURITY should be reported as findings.
```

**Usage:** All 5 exploitation agents.

**Impact:** Prevents wasting time on internal-only vulnerabilities during exploitation.

---

### `login-instructions.txt`

**Purpose:** Comprehensive authentication flow template for all agents requiring login.

**Structure (82 lines):**
```xml
<!-- BEGIN:COMMON -->
**IF you are not already logged in when you open playwright...**
<user_provided_configuration>
{{user_instructions}}
</user_provided_configuration>

<authentication_execution>
Execute the login flow based on login_type:
<!-- END:COMMON -->

<!-- BEGIN:FORM -->
**Form-based authentication:**
1. Navigate to login_url
2. Execute login_flow steps:
   - Replace $username, $password, $totp (via generate_totp MCP tool)
   - Perform actions (type, click)
3. Wait for page navigation
4. Handle consent dialogs
<!-- END:FORM -->

<!-- BEGIN:SSO -->
**SSO authentication:**
1. Navigate to login_url
2. Execute login_flow steps:
   - Click SSO button
   - Handle account selection
   - Replace credentials, $totp
   - Handle OAuth consent (click "Allow", "Accept")
   - Handle "Continue as [user]" dialogs
3. Wait for OAuth callback
<!-- END:SSO -->

<!-- BEGIN:VERIFICATION -->
</authentication_execution>

<success_verification>
After login:
1. Check success_condition:
   - url_contains: Verify URL contains value
   - url_equals_exactly: Verify URL exact match
   - element_present: Verify element exists
2. Confirm authentication state (not on login screen)
3. If verification fails → Retry once after 5s wait
   - If second attempt fails → Report failure and STOP
</success_verification>

<error_handling>
Common issues:
- OAuth consent screens requiring explicit "Allow"
- "Continue as [user]" prompts
- TOTP/2FA timing issues
- Page loading delays
</error_handling>
<!-- END:VERIFICATION -->
```

**Usage:** Recon, all 5 exploitation agents.

**Key Features:**
- Supports form-based and SSO authentication
- TOTP generation via MCP tool
- Success verification logic
- Error handling with retry
- Multi-factor authentication support

---

## Queue-Based Validation System

### Queue Structure (Universal Pattern)

**All 5 vulnerability types use this queue structure:**

```json
{
  "vulnerabilities": [
    {
      "ID": "TYPE-VULN-NN",
      "vulnerability_type": "[VULN_TYPE_ENUM]",
      "externally_exploitable": true,

      // Common fields
      "confidence": "high|med|low",
      "notes": "context details",

      // Type-specific fields (example: Injection)
      "source": "param & file:line",
      "path": "data flow path",
      "sink_call": "file:line and function",
      "verdict": "vulnerable",
      "witness_payload": "minimal payload"
    }
  ]
}
```

### Vulnerability Type Enums

```typescript
// Injection
"vulnerability_type": "SQLi | CommandInjection | LFI | RFI | SSTI | PathTraversal | InsecureDeserialization"

// XSS
"vulnerability_type": "Reflected | Stored | DOM-based"

// Auth
"vulnerability_type": "Authentication_Bypass | Session_Management_Flaw | Login_Flow_Logic | Token_Management_Issue | Reset_Recovery_Flaw | Transport_Exposure | Abuse_Defenses_Missing | OAuth_Flow_Issue"

// Authz
"vulnerability_type": "Horizontal | Vertical | Context_Workflow"

// SSRF
"vulnerability_type": "URL_Manipulation | Redirect_Abuse | Webhook_Injection | API_Proxy_Bypass | File_Fetch_Abuse | Service_Discovery"
```

### Validation Logic Flow

```typescript
// From src/queue-validation.ts

// Step 1: Check file existence
const queuePath = `deliverables/${vulnType}_exploitation_queue.json`;
if (!fs.existsSync(queuePath)) {
  return { shouldExploit: false, shouldRetry: false };
}

// Step 2: Parse and validate JSON
const queueData = JSON.parse(fs.readFileSync(queuePath, 'utf-8'));

// Step 3: Check vulnerability count
const vulnerabilities = queueData.vulnerabilities || [];

if (vulnerabilities.length === 0) {
  return { shouldExploit: false, vulnerabilityCount: 0 };
}

// Step 4: Filter by externally_exploitable
const externalVulns = vulnerabilities.filter(v => v.externally_exploitable === true);

if (externalVulns.length === 0) {
  return { shouldExploit: false, vulnerabilityCount: 0 };
}

// Step 5: Return exploitation decision
return {
  shouldExploit: true,
  vulnerabilityCount: externalVulns.length,
  vulnType
};
```

### Workflow Integration

**From `src/temporal/workflows.ts` (pipelined execution):**

```typescript
// For each vulnerability type in parallel
const pipelineResults = await Promise.allSettled([
  runVulnExploitPipeline('injection',
    () => a.runInjectionVulnAgent(input),
    () => a.runInjectionExploitAgent(input)
  ),
  runVulnExploitPipeline('xss',
    () => a.runXssVulnAgent(input),
    () => a.runXssExploitAgent(input)
  ),
  // ... etc for auth, authz, ssrf
]);

async function runVulnExploitPipeline(
  vulnType: VulnType,
  runVulnAgent: () => Promise<AgentMetrics>,
  runExploitAgent: () => Promise<AgentMetrics>
): Promise<VulnExploitPipelineResult> {
  // Step 1: Run vulnerability agent
  const vulnMetrics = await runVulnAgent();

  // Step 2: Check exploitation queue (GATEKEEPER)
  const decision = await a.checkExploitationQueue(input, vulnType);

  // Step 3: Conditionally run exploit agent
  let exploitMetrics: AgentMetrics | null = null;
  if (decision.shouldExploit) {
    exploitMetrics = await runExploitAgent();
  }

  return {
    vulnType,
    vulnMetrics,
    exploitMetrics,
    exploitDecision: {
      shouldExploit: decision.shouldExploit,
      vulnerabilityCount: decision.vulnerabilityCount
    }
  };
}
```

**Result:** Pipelined parallelism where exploit agents start immediately after their vuln agent completes, without waiting for other vuln agents.

---

## Prompt Design Principles

### Principle 1: Role Clarity

Every prompt begins with a **precise role definition:**

```xml
<role>
You are a [SPECIALIST] Specialist, an expert in [DOMAIN].
Your primary function is to [PRIMARY_TASK].
</role>
```

**Examples:**
- "Principal Engineer specializing in rapid, security-focused code review"
- "Reconnaissance Analyst and Attack Surface Mapper"
- "Injection Analysis Specialist, expert in white-box code analysis and data flow tracing"
- "World-class Injection Exploitation Specialist (not an analyst, active penetration tester)"

**Benefit:** Sets clear expectations and behavioral boundaries.

---

### Principle 2: System Architecture Context

Every prompt includes **workflow position and coordination requirements:**

```xml
<system_architecture>
**PENTESTING WORKFLOW - YOUR POSITION:**

**Phase Sequence:** [PREVIOUS] → **YOU** → [NEXT]

**Your Input:** [INPUT_FILES]
**Your Output:** [OUTPUT_FILES]

**WHAT HAPPENED BEFORE YOU:**
- [PREVIOUS_PHASE_SUMMARY]

**WHAT HAPPENS AFTER YOU:**
- [NEXT_PHASE_SUMMARY]

**YOUR CRITICAL ROLE:**
- [YOUR_IMPACT_ON_WORKFLOW]

**COORDINATION REQUIREMENTS:**
- [WHAT_DOWNSTREAM_AGENTS_DEPEND_ON]
</system_architecture>
```

**Benefit:** Agents understand their position and dependencies in the workflow.

---

### Principle 3: Evidence Requirements

All prompts define **explicit proof standards:**

**Vulnerability Analysis Prompts:**
```xml
<proof_obligation_criteria>
**What counts as proof in the analysis phase:**
- Documented path from source → sanitization → sink
- Clear sanitizer-to-context mismatch OR missing defense
- Concatenation after sanitization recorded
- Potential witness_payload crafted (not executed)
</proof_obligation_criteria>
```

**Exploitation Prompts:**
```xml
**Proof of Exploitation Levels:**
Level 1 - Weakness identified → POTENTIAL (Low)
Level 2 - Partial bypass → POTENTIAL (Medium)
Level 3 - Exploitation confirmed → EXPLOITED ✓
Level 4 - Critical impact → EXPLOITED (CRITICAL) ✓

**MUST reach Level 3+ to mark as EXPLOITED**
```

**Benefit:** Prevents false positives and ensures quality findings.

---

### Principle 4: Tool Usage Guidelines

Every prompt specifies **tool usage strategy:**

**Vulnerability Analysis Prompts:**
```xml
<available_tools>
**CRITICAL TOOL USAGE RESTRICTIONS:**
- NEVER use Read tool for application source code analysis
- ALWAYS delegate to Task Agent for code examination

**Available Tools:**
- Task Agent (Code Analysis): MANDATORY for all source code analysis
- save_deliverable (MCP Tool): Saves deliverables with validation
- TodoWrite Tool: Create/manage analysis task list
- Bash tool: For creating directories, copying files
- {{MCP_SERVER}} (Playwright): For live application interaction
</available_tools>
```

**Exploitation Prompts:**
```xml
<available_tools>
- {{MCP_SERVER}} (Playwright): Primary tool for exploitation
- save_deliverable (MCP Tool): Saves evidence files
- Bash tool: Creating directories, copying files
- Task Agent: Mandatory for custom scripting (payload loops, enumeration)
- TodoWrite tool: Essential for managing exploitation task list
</available_tools>
```

**Benefit:** Prevents tool misuse and ensures systematic workflows.

---

### Principle 5: Methodology Specification

All analysis prompts include **step-by-step methodology:**

**Example from Injection Analysis:**
```xml
<methodology>
## 1) Create a To Do for each Injection Source
   - Use TodoWrite to create task for each source

## 2) Trace Data Flow Paths
   - Identify unique routes from source to sink
   - Document: transformations, sanitizers, concatenations

## 3) Detect Sinks and Label Slot Types
   - SQLi: SQL-val, SQL-like, SQL-num, SQL-enum, SQL-ident
   - Command: CMD-argument, CMD-part-of-string

## 4) Match Sanitization to Sink Context
   - SQL: Parameter binding for val/like/num; whitelist for enum/ident

## 5) Make the Call (vulnerable or safe)
   - Vulnerable if tainted input + no/mismatched defense

## 6) Append to Findings
   - Add vulnerable paths to queue.json
   - Document secure paths in markdown

## 7) Score Confidence
   - High: Binds + no post-sanitization concat
   - Medium: Binds present but unclear transforms
   - Low: Concat into syntax slots
</methodology>
```

**Benefit:** Ensures consistent, thorough analysis across all agents.

---

### Principle 6: Scope Enforcement

All prompts reinforce **network-accessible scope boundaries:**

```xml
<scope_boundaries>
### In-Scope: Network-Reachable Components
- Publicly exposed web pages and API endpoints
- Endpoints requiring authentication
- Mistakenly exposed developer utilities

### Out-of-Scope: Locally Executable Only
- CLI tools (go run, python scripts)
- Build tools (CI/CD, Dagger)
- Local dev servers, test harnesses
- Static files not served by application
</scope_boundaries>
```

**Queue Field Enforcement:**
```json
{
  "externally_exploitable": true  // GATEKEEPER
}
```

**Benefit:** Prevents wasted effort on internal-only components.

---

### Principle 7: Cascade Impact Awareness

All prompts emphasize **impact on downstream agents:**

```xml
<critical>
**Cascade Impact:** Your analysis is the foundation for the entire
security assessment. An incomplete analysis creates blind spots that
persist through all 10+ subsequent agents.

**You are the ONLY agent with full source code access.** If you miss
a security component, no other agent can discover it.
</critical>
```

**Benefit:** Motivates thoroughness and prevents early termination.

---

## Pipeline-Testing Variants

### Purpose

**Fast workflow validation** without full analysis. Used for:
- Development iteration (10s retry vs 5min production)
- Workflow testing
- MCP server isolation verification

### Key Differences

| Aspect | Production Prompts | Pipeline-Testing Prompts |
|---------|-------------------|-------------------------|
| **Line Count** | 150-394 lines | 36 lines (90% reduction) |
| **Task Agents** | 1-6 parallel agents | 0 (no Task agents) |
| **Analysis Depth** | Comprehensive source-to-sink traces | None (minimal prompts) |
| **Deliverables** | Real findings | Mock/test data |
| **Purpose** | Actual pentest | Workflow validation |

### Example: Injection Agent

**Production Prompt (`vuln-injection.txt` - 371 lines):**
```xml
<role>
You are an Injection Analysis Specialist...
</role>

<objective>
Your mission is to identify where untrusted input reaches dangerous sinks...
Success criterion: Complete source-to-sink trace...
</objective>

<methodology>
## 1) Create a To Do for each Injection Source
## 2) Trace Data Flow Paths
## 3) Detect Sinks...
## 4) Match Sanitization...
## 5) Make the Call...
## 6) Append to Findings...
## 7) Score Confidence...
</methodology>
```

**Pipeline-Testing Prompt (`pipeline-testing/vuln-injection.txt` - 36 lines):**
```markdown
## 🧪 Pipeline Testing: MCP Isolation Test for Injection Agent

### Step 1: Test Browser Isolation
1. Navigate to example.com via {{MCP_SERVER}}
2. Take screenshot
3. Log results

### Step 2: Create Required Deliverables
Use save_deliverable to create:
- Analysis report with test content
- Exploitation queue with mock vulnerability

### Step 3: Verify MCP Isolation
This agent should use {{MCP_SERVER}} independently.
```

### Validation Workflow

**When `PIPELINE_TESTING=true`:**
1. All prompts use `prompts/pipeline-testing/` versions
2. Retry intervals: 10s (vs 5min production)
3. Maximum attempts: 5 (vs 50 production)
4. Agents navigate to `https://example.com`
5. Screenshots prove MCP server isolation
6. Mock queues validate workflow execution

**Benefits:**
- **Fast iteration:** Test prompt changes in ~30 seconds instead of 30 minutes
- **Workflow validation:** Verify all agents execute in correct order
- **MCP isolation:** Confirm parallel agents don't interfere
- **Queue structure:** Validate JSON schema compliance

---

## Prompt Reference Cards

### Quick Reference Prompt Matrix

| Phase | Agent | Input Files | Output Files | Task Agents | Line Count |
|-------|-------|------------|--------------|-------------|------------|
| **1** | Pre-Recon | Source code | code_analysis_deliverable.md<br/>outputs/schemas/ | 6 (3 phases) | 394 |
| **2** | Recon | code_analysis_deliverable.md | recon_deliverable.md | 5 (parallel) | 370 |
| **3a** | Injection Vuln | pre_recon_deliverable.md<br/>recon_deliverable.md | injection_analysis_deliverable.md<br/>injection_exploitation_queue.json | 1-2 (sequential) | 371 |
| **3a** | XSS Vuln | pre_recon_deliverable.md<br/>recon_deliverable.md | xss_analysis_deliverable.md<br/>xss_exploitation_queue.json | 1-2 (sequential) | 150+ |
| **3a** | Auth Vuln | pre_recon_deliverable.md<br/>recon_deliverable.md | auth_analysis_deliverable.md<br/>auth_exploitation_queue.json | 1 (sequential) | 150+ |
| **3a** | Authz Vuln | pre_recon_deliverable.md<br/>recon_deliverable.md | authz_analysis_deliverable.md<br/>authz_exploitation_queue.json | 1 (sequential) | 150+ |
| **3a** | SSRF Vuln | pre_recon_deliverable.md<br/>recon_deliverable.md | ssrf_analysis_deliverable.md<br/>ssrf_exploitation_queue.json | 1 (sequential) | 150+ |
| **4** | Injection Exploit | 3 intel files | injection_exploitation_evidence.md | TodoWrite-driven | 200+ |
| **4** | XSS Exploit | 3 intel files | xss_exploitation_evidence.md | TodoWrite-driven | 150+ |
| **4** | Auth Exploit | 3 intel files | auth_exploitation_evidence.md | TodoWrite-driven | 150+ |
| **4** | Authz Exploit | 3 intel files | authz_exploitation_evidence.md | TodoWrite-driven | 150+ |
| **4** | SSRF Exploit | 3 intel files | ssrf_exploitation_evidence.md | TodoWrite-driven | 150+ |
| **5** | Report | 3 intel files<br/>+ concatenated report | comprehensive_security_assessment_report.md<br/>(modified) | 0 | 127 |

---

### Agent Coordination Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    SHARED PARTIAL TEMPLATES                      │
│                                                                   │
│  _target.txt          →  {{WEB_URL}}                             │
│  _rules.txt           →  {{RULES_AVOID}} / {{RULES_FOCUS}}       │
│  _vuln-scope.txt      →  External attacker scope (vuln agents)   │
│  _exploit-scope.txt   →  Exploitation constraints (exploit)      │
│  login-instructions.txt →  Authentication flow (recon + exploit) │
└──────────────┬────────────────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 1: PRE-RECON                                               │
│ Prompt: pre-recon-code.txt (394 lines)                           │
│ Task Agents: 6 (Architecture, Entry Points, Security Patterns,  │
│                XSS/Injection Sinks, SSRF Tracer, Data Auditor)   │
│ Output: code_analysis_deliverable.md + schemas/                  │
└──────────────┬────────────────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 2: RECON                                                   │
│ Prompt: recon.txt (370 lines)                                    │
│ Task Agents: 5 (Route Mapper, Auth Checker, Input Validator,     │
│                Session Handler, Auth Architect)                  │
│ Reads: code_analysis_deliverable.md                              │
│ Output: recon_deliverable.md                                     │
│                                                                   │
│ Key Sections Created:                                             │
│ • API Endpoint Inventory (with authorization details)            │
│ • Authentication & Session Management Flow                       │
│ • Role & Privilege Architecture                                  │
│ • Authorization Vulnerability Candidates (Section 8)             │
│ • Injection Sources (Section 9)                                  │
│ • XSS Sinks (from pre-recon)                                     │
│ • SSRF Sinks (from pre-recon)                                    │
└──┬────────────┬────────────┬────────────┬────────────┬───────────┘
   │            │            │            │            │
   ▼            ▼            ▼            ▼            ▼
┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
│  INJ    │ │   XSS   │ │  AUTH   │ │  AUTHZ  │ │  SSRF   │
│  VULN   │ │  VULN   │ │  VULN   │ │  VULN   │ │  VULN   │
└────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘
     │           │           │           │           │
     ▼           ▼           ▼           ▼           ▼
   Queues     Queues     Queues     Queues     Queues
   (only)     (only)     (only)     (only)     (only)
   external   external   external   external   external
     │           │           │           │           │
     ▼           ▼           ▼           ▼           ▼
┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
│  INJ    │ │   XSS   │ │  AUTH   │ │  AUTHZ  │ │  SSRF   │
│EXPLOIT  │ │ EXPLOIT │ │ EXPLOIT │ │ EXPLOIT │ │ EXPLOIT │
└─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘
     │           │           │           │           │
     ▼           ▼           ▼           ▼           ▼
┌─────────────────────────────────────────────────────────────────┐
│ ALL EXPLOITATION EVIDENCE FILES                                 │
│                                                                   │
│ • injection_exploitation_evidence.md                            │
│ • xss_exploitation_evidence.md                                  │
│ • auth_exploitation_evidence.md                                 │
│ • authz_exploitation_evidence.md                                │
│ • ssrf_exploitation_evidence.md                                 │
└──────────────┬────────────────────────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────────────────────┐
│ PHASE 5: REPORT                                                 │
│ Prompt: report-executive.txt (127 lines)                         │
│ Task Agents: 0                                                   │
│ Reads: pre_recon, recon, all evidence, concatenated report       │
│ Output: comprehensive_security_assessment_report.md (MODIFIED)  │
│                                                                   │
│ Modifications:                                                   │
│ 1. ADD Executive Summary                                          │
│ 2. ADD Network Recon                                             │
│ 3. CLEAN hallucinated content (no ID = remove)                   │
│ 4. KEEP only exploitation evidence with proper IDs               │
└─────────────────────────────────────────────────────────────────┘
```

---

### Cross-Phase Data Dependencies

**Pre-Recon → Recon:**
```typescript
// Recon reads these sections from code_analysis_deliverable.md:
- Section 3: Authentication & Authorization Deep Dive
  → Recon builds detailed API inventory with auth endpoints

- Section 5: Attack Surface Analysis
  → Recon enumerates all input vectors with file:line locations

- Section 9: XSS Sinks
  → Recon includes sink context in authorization candidates

- Section 10: SSRF Sinks
  → Recon maps request patterns for SSRF specialist
```

**Recon → Vulnerability Agents:**
```typescript
// Injection reads:
- Section 7: Injection Sources
  → Creates todo for each source, traces data flow

// XSS reads:
- Section 9: XSS Sinks (from pre-recon, included in recon)
  → Creates todo for each sink, traces backward

// Auth reads:
- Section 3: Authentication & Session Management
  → Analyzes transport, rate limiting, session management, tokens

// Authz reads:
- Section 7: Role & Privilege Architecture
- Section 8: Authorization Vulnerability Candidates
  → Tests horizontal, vertical, context-based candidates

// SSRF reads:
- Section 10: SSRF Sinks (from pre-recon, included in recon)
  → Tests HTTP client usage, validation, bypasses
```

**Vulnerability Agents → Exploitation Agents:**
```typescript
// Each exploitation agent reads:
1. pre_recon_deliverable.md
   - Architecture, tech stack, code locations

2. recon_deliverable.md
   - API inventory, input vectors, data flows

3. [type]_analysis_deliverable.md
   - Strategic context (WAF behavior, CSP, defensive measures)

4. [type]_exploitation_queue.json
   - Actionable vulnerabilities with witness_payloads
```

**All Phases → Report:**
```typescript
// Report agent reads:
1. pre_recon_deliverable.md
   - Extract Network Reconnaissance (nmap, subfinder, whatweb)

2. recon_deliverable.md
   - Security-relevant architecture findings

3. comprehensive_security_assessment_report.md (TO MODIFY)
   - All exploitation evidence sections
   - Removes sections without vulnerability IDs
   - Adds Executive Summary at top
```

---

## Summary

### Key Architectural Insights

1. **Cascade Impact** - Early agent errors propagate through all 10+ downstream agents
2. **Queue-Based Gating** - Only `externally_exploitable: true` vulnerabilities proceed to exploitation
3. **Pipelined Parallelism** - No synchronization barriers; each vuln-exploit pair operates independently
4. **Shared Intelligence** - Each phase builds on previous outputs without re-analysis
5. **Proof Standards** - 4-level proof framework prevents false positives
6. **Scope Enforcement** - Network-accessible scope enforced at every phase via `externally_exploitable` field
7. **File:line Precision** - Exact code locations passed handoff-to-handoff
8. **Task Agent Coordination** - Different workflows optimized for each phase (phased, parallel, sequential, TodoWrite-driven)
9. **No Exploit, No Report** - Only successfully exploited vulnerabilities (Level 3+) reach final report
10. **Evidence Tracking** - Every vulnerability pursued to definitive conclusion (EXPLOITED or proven NOT)

### Prompt Statistics

- **Total Production Prompts:** 13 agents
- **Total Prompt Lines:** ~2,400 lines
- **Total Task Agent Coordination:** ~40 Task agents across all phases
- **Total Deliverable Files:** 21 (11 analysis + 5 queues + 5 evidence)
- **Average Prompt Length:** 185 lines per agent
- **Longest Prompt:** Pre-Recon (394 lines)
- **Shortest Prompt:** Report (127 lines)
- **Pipeline-Testing Reduction:** 90% shorter (36 lines vs 371 lines for injection)

---

**END OF DOCUMENT**
