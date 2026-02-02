# Shannon Reconnaissance Architecture

**Document Version:** 1.0
**Last Updated:** 2025-02-02
**Purpose:** Deep dive into Shannon's two-phase reconnaissance system and how it differs from traditional DAST crawlers

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Shannon's Hybrid Approach](#shannons-hybrid-approach)
3. [Phase 1: Pre-Reconnaissance](#phase-1-pre-reconnaissance)
4. [Phase 2: Reconnaissance](#phase-2-reconnaissance)
5. [Comparison with Traditional DAST](#comparison-with-traditional-dast)
6. [Tool Integration](#tool-integration)
7. [Data Flow Between Phases](#data-flow-between-phases)
8. [Key Files Reference](#key-files-reference)

---

## Executive Summary

Shannon uses a **two-phase hybrid reconnaissance approach** that combines:

1. **External tool-based scanning** (nmap, subfinder, whatweb)
2. **Deep source code analysis** (AI-powered static analysis)
3. **Live application exploration** (Playwright browser automation)

**Key Differentiator:** Shannon does NOT use traditional web crawlers/spiders. Instead, it uses **AI-guided intelligent exploration** via Playwright MCP server, combined with Task Agent-based source code analysis.

---

## Shannon's Hybrid Approach

```
┌─────────────────────────────────────────────────────────────────┐
│ SHANNON RECONNAISSANCE ARCHITECTURE                             │
└─────────────────────────────────────────────────────────────────┘

┌─ Phase 1: Pre-Reconnaissance (pre-recon agent) ────────────────┐
│                                                                  │
│ External Tools (Traditional DAST):                               │
│   • nmap        - Port scanning, service discovery               │
│   • subfinder   - Subdomain enumeration                          │
│   • whatweb     - Technology fingerprinting                      │
│                                                                  │
│ + Source Code Analysis (AI-powered):                             │
│   • Task Agent architecture scanning                             │
│   • Security component mapping                                   │
│   • Attack surface identification                                │
│   • Tech stack extraction                                        │
│                                                                  │
│ Output: deliverables/pre_recon_deliverable.md                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌─ Phase 2: Reconnaissance (recon agent) ────────────────────────┐
│                                                                  │
│ Live Application Exploration (AI-guided via Playwright):         │
│   • Browser automation to navigate app                           │
│   • Map user-facing functionality                                │
│   • Observe network requests/API calls                           │
│   • Document authentication flows                                │
│   • Discover forms, inputs, endpoints                           │
│                                                                  │
│ + Source Code Correlation (Parallel Task Agents):               │
│   • Route Mapper Agent       - Map endpoints to handlers        │
│   • Authorization Checker   - Find auth middleware/guards        │
│   • Input Validator Agent   - Analyze validation logic          │
│   • Session Handler Agent   - Trace session management          │
│   • Auth Architecture Agent - Map role hierarchy                 │
│                                                                  │
│ Output: deliverables/recon_deliverable.md                        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
                              ↓
                    Vulnerability Analysis (5 agents)
```

---

## Phase 1: Pre-Reconnaissance

### Agent: `pre-recon`
**Prompt:** `prompts/pre-recon-code.txt`
**MCP Assignment:** `playwright-agent1`
**Output:** `deliverables/code_analysis_deliverable.md`

### Objectives

1. **External Tool Scanning:**
   - Run nmap for port/service discovery
   - Run subfinder for subdomain enumeration
   - Run whatweb for technology fingerprinting

2. **Source Code Architecture Analysis:**
   - Map application structure and tech stack
   - Identify security-relevant components
   - Catalog attack surfaces
   - Document defensive mechanisms

### Tool Availability Check

**File:** `src/tool-checker.ts`

Shannon validates tool availability before starting:

```typescript
export type ToolName = 'nmap' | 'subfinder' | 'whatweb' | 'schemathesis';

export const checkToolAvailability = async (): Promise<ToolAvailability> => {
  const tools: ToolName[] = ['nmap', 'subfinder', 'whatweb', 'schemathesis'];

  for (const tool of tools) {
    try {
      await $`command -v ${tool}`;
      availability[tool] = true;
      console.log(chalk.green(`  ✅ ${tool} - available`));
    } catch {
      availability[tool] = false;
      console.log(chalk.yellow(`  ⚠️ ${tool} - not found`));
    }
  }
};
```

**Tools Used:**

| Tool | Purpose | Category |
|------|---------|----------|
| **nmap** | Port scanning, service discovery, OS fingerprinting | Network Recon |
| **subfinder** | Subdomain enumeration via passive DNS | Surface Discovery |
| **whatweb** | Technology stack identification (frameworks, servers, libs) | Tech Fingerprinting |
| **schemathesis** | API schema testing (optional) | API Testing |

### Source Code Analysis Approach

**Prompt Instructions (lines 72-100):**

The pre-recon agent uses a **phased Task Agent approach**:

**Phase 1: Discovery Agents (Launch in Parallel)**

1. **Architecture Scanner Agent:**
   ```
   "Map the application's structure, technology stack, and critical components.
   Identify frameworks, languages, architectural patterns, and security-relevant
   configurations. Determine if this is a web app, API service, microservices,
   or hybrid."
   ```

2. **Security Component Hunter Agent:**
   ```
   "Find all security-critical components: authentication systems, authorization
   logic, input validation, output encoding, cryptographic operations, session
   management, data handling, and third-party integrations."
   ```

3. **Attack Surface Mapper Agent:**
   ```
   "Identify all potential attack surfaces: API endpoints, web routes, form inputs,
   file upload handlers, database queries, external service calls, and data
   processing operations."
   ```

**Key Principle:** Pre-recon is the **ONLY agent with complete source code access**. All other agents depend on its analysis.

**From prompt (lines 6-9):**
> **Cascade Impact:** Your analysis is the foundation for the entire security assessment.
> An incomplete analysis here creates blind spots that persist through all 10+ subsequent
> agents. This is not just a code review - this is intelligence gathering that determines
> whether critical vulnerabilities are found or missed.

---

## Phase 2: Reconnaissance

### Agent: `recon`
**Prompt:** `prompts/recon.txt`
**MCP Assignment:** `playwright-agent2`
**Input:** `deliverables/pre_recon_deliverable.md`
**Output:** `deliverables/recon_deliverable.md`

### Objectives

1. **Live Application Exploration (via Playwright):**
   - Navigate to target application
   - Map user-facing functionality
   - Document authentication flows
   - Observe network requests
   - Discover forms, inputs, endpoints

2. **Source Code Correlation (via Task Agents):**
   - Map discovered endpoints to backend handlers
   - Find authorization middleware and guards
   - Analyze input validation logic
   - Trace session management

### Systematic Approach

**From prompt (lines 105-134):**

#### Step 1: Synthesize Initial Data
- Read entire `deliverables/pre_recon_deliverable.md`
- Create preliminary list of known technologies, subdomains, open ports

#### Step 2: Interactive Application Exploration (AI-guided via Playwright)
```
Use {{MCP_SERVER}}__browser_navigate to navigate to the target.
Map out all user-facing functionality:
  - Login forms
  - Registration flows
  - Password reset pages
  - Multi-step processes

Observe network requests to identify primary API calls.
```

**Example Flow:**
```javascript
// Navigate to application
await browser_navigate("https://target.com")

// Map authentication flow
await browser_click("#login-button")
await browser_type("#username", "test@example.com")
await browser_type("#password", "password123")
await browser_click("#submit")

// Observe network requests
// Note: /api/auth/login endpoint called
// Note: JWT token returned in response
// Note: Session cookie set

// Map user dashboard
await browser_navigate("https://target.com/dashboard")
// Note: Lists user orders, profile settings
// Note: API calls: /api/users/me, /api/orders
```

#### Step 3: Correlate with Source Code (Parallel Task Agents)

For each discovered functionality, launch **parallel Task agents**:

1. **Route Mapper Agent:**
   ```
   "Find all backend routes and controllers that handle the discovered endpoints:
   [/api/auth/login, /api/users/me, /api/orders]. Map each endpoint to its exact
   handler function with file paths and line numbers."
   ```

2. **Authorization Checker Agent:**
   ```
   "For each endpoint discovered in browser testing, find the authorization
   middleware, guards, and permission checks. Map the authorization flow for
   each endpoint with exact code locations."
   ```

3. **Input Validator Agent:**
   ```
   "Analyze the input validation logic for all discovered form fields and API
   parameters. Find validation rules, sanitization, and data processing for
   each input with exact file paths."
   ```

4. **Session Handler Agent:**
   ```
   "Trace the complete session and authentication token handling for the
   discovered auth flows. Map session creation, storage, validation, and
   destruction with exact code locations."
   ```

#### Step 3.5: Authorization Architecture Analysis
```
Launch a dedicated Authorization Architecture Agent to comprehensively map
the authorization system:
  - User roles and hierarchies
  - Permission models
  - Authorization decision points
  - Object ownership patterns
  - Role-based access patterns
```

#### Step 4: Enumerate and Document
- Synthesize findings from all Task agents
- Cross-reference browser observations with source code
- Create comprehensive attack surface maps

### Deliverable Structure

**File:** `deliverables/recon_deliverable.md`

The recon agent creates a structured report with the following sections:

**0) How to Read This**
- Guide for downstream specialists
- Priority order for testing

**1) Executive Summary**
- Application purpose
- Core technology stack
- Primary attack surface components

**2) Technology & Service Map**
- Frontend: Framework, libraries
- Backend: Language, framework
- Infrastructure: Hosting, CDN, database
- Identified subdomains (from subfinder)
- Open ports & services (from nmap)

**3) Authentication & Session Management Flow**
- Entry points (/login, /register, /auth/sso)
- Mechanism (step-by-step process)
- Code pointers (file paths)

**3.1) Role Assignment Process**
- How roles are assigned
- Default role
- Role upgrade path

**3.2) Privilege Storage & Validation**
- Storage location (JWT, session, database)
- Validation points (middleware, decorators)
- Cache/session persistence

**4) API Endpoint Inventory**
**Table columns:**
| Method | Endpoint | Required Role | Object ID Params | Auth Mechanism | Description |
|---|---|---|---|---|---|

**Example:**
```
| GET | /api/users/{user_id} | user | user_id | Bearer + ownership check | Fetch user profile. See users.controller.ts:45 |
| DELETE | /api/orders/{order_id} | user | order_id | Bearer + order ownership | Delete order. See orders.controller.ts:78 |
```

**5) Potential Input Vectors for Vulnerability Analysis**
- URL parameters
- POST body fields
- HTTP headers
- Cookie values

**6) Network & Interaction Map**
- Entities (services, databases, gateways)
- Flows (connections between entities)
- Guards (auth, authorization, rate limiting)
- Metadata (technical details)

**7) Role & Privilege Architecture**
- Discovered roles (with privilege levels)
- Privilege lattice (hierarchy)
- Role entry points (accessible routes)

**8) Authorization Vulnerability Candidates**
- High-priority horizontal escalation targets
- Vertical escalation targets per role
- Context-based workflow bypasses

---

## Comparison with Traditional DAST

### Traditional DAST Approach

```
┌─ Traditional DAST Workflow ─────────────────────────────────────┐
│                                                                  │
│ 1. Crawler/Spider discovers application:                         │
│    • Start from homepage                                         │
│    • Follow all links recursively                                │
│    • Extract forms, inputs, endpoints                            │
│    • Build complete site map                                     │
│                                                                  │
│ 2. Scanner tests discovered endpoints:                           │
│    • SQL injection payloads on all inputs                        │
│    • XSS payloads on all reflection points                       │
│    • Fuzzing with malformed data                                 │
│    • Check for common vulnerabilities                            │
│                                                                  │
│ Tools: Burp Suite, OWASP ZAP, Nessus, etc.                      │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

**Characteristics:**
- **Automated crawling:** Follows links blindly
- **Broad coverage:** Tests everything discovered
- **Shallow analysis:** No understanding of business logic
- **No source code:** Black-box testing only
- **High false positives:** Doesn't understand context
- **Pattern-based:** Detects known vulnerability patterns

### Shannon's AI-Guided Approach

```
┌─ Shannon Reconnaissance Workflow ────────────────────────────────┐
│                                                                  │
│ 1. Pre-reconnaissance (External tools + Source Code):           │
│    • Run nmap/subfinder/whatweb (network intel)                 │
│    • AI-powered source code analysis                             │
│    • Identify security components                                │
│    • Map attack surfaces                                         │
│                                                                  │
│ 2. Reconnaissance (AI-guided exploration + Code correlation):    │
│    • Navigate app intelligently via Playwright                   │
│    • Focus on user-facing functionality                          │
│    • Observe authentication flows                                │
│    • Document API calls and endpoints                            │
│    • Launch Task Agents to correlate with source code            │
│    • Map endpoints to handlers                                   │
│    • Find authorization middleware                               │
│    • Analyze input validation                                    │
│    • Build comprehensive attack surface map                      │
│                                                                  │
│ 3. Vulnerability Analysis (5 specialist agents):                 │
│    • Each specialist uses attack surface map                     │
│    • Source code-aware testing                                   │
│    • Business logic understanding                                │
│    • Context-aware vulnerability detection                       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

**Characteristics:**
- **AI-guided exploration:** Intelligent navigation, not blind crawling
- **Source code aware:** Correlates observations with implementation
- **Deep understanding:** Business logic, authorization flows, data sensitivity
- **Context-aware:** Knows what's critical vs. what's trivial
- **Low false positives:** Understands application context
- **Hybrid approach:** External tools + source code + live testing

### Key Differences

| Aspect | Traditional DAST | Shannon |
|--------|------------------|---------|
| **Discovery Method** | Automated crawler/spider | AI-guided Playwright exploration |
| **Source Code Access** | No (black-box only) | Yes (white-box analysis) |
| **Coverage Strategy** | Broad, test everything | Targeted, high-value areas |
| **Authentication Handling** | Scripts, manual recording | AI-driven flow understanding |
| **Business Logic Understanding** | No | Yes (via source code) |
| **False Positive Rate** | High | Low (context-aware) |
| **Authorization Testing** | Basic (role switching) | Advanced (role hierarchy analysis) |
| **Time Efficiency** | Slow (crawl everything) | Fast (focused exploration) |
| **Scalability** | Limited by crawler depth | Limited by AI agent scope |

---

## Tool Integration

### External Tools (Pre-recon)

**File:** `src/tool-checker.ts`

| Tool | Command | Purpose | Output |
|------|---------|---------|--------|
| **nmap** | `nmap -p- <target>` | Port scanning | Open ports, services, versions |
| **subfinder** | `subfinder -d <domain>` | Subdomain enumeration | List of subdomains |
| **whatweb** | `whatweb <target>` | Technology fingerprinting | Frameworks, servers, libs |
| **schemathesis** | `schemathesis run <url>` | API schema testing | API vulnerabilities |

### Browser Automation (Recon)

**MCP Server:** Playwright (assigned via `MCP_AGENT_MAPPING`)

**Tools Used:**
- `browser_navigate(url)` - Navigate to pages
- `browser_click(selector)` - Click elements
- `browser_type(selector, text)` - Fill forms
- `browser_snapshot()` - Get page structure
- `browser_take_screenshot()` - Capture evidence
- `browser_network_requests()` - Observe API calls

**Note:** Recon uses `playwright-agent2` (isolated from pre-recon's `playwright-agent1`)

### AI Task Agents (Both Phases)

**Tool:** Task Agent (Claude Agent SDK)

**Specialist Agents Launched:**
- Architecture Scanner Agent
- Security Component Hunter Agent
- Attack Surface Mapper Agent
- Route Mapper Agent
- Authorization Checker Agent
- Input Validator Agent
- Session Handler Agent
- Authorization Architecture Agent

---

## Data Flow Between Phases

```
┌─────────────────────────────────────────────────────────────────┐
│ DATA FLOW: PRE-RECON → RECON → VULN ANALYSIS                   │
└─────────────────────────────────────────────────────────────────┘

Pre-Recon Agent
  │
  ├─ Runs nmap → ports.txt
  ├─ Runs subfinder → subdomains.txt
  ├─ Runs whatweb → tech.txt
  │
  ├─ Runs Task Agents (parallel):
  │   ├─ Architecture Scanner → tech_stack_analysis
  │   ├─ Security Component Hunter → security_components
  │   └─ Attack Surface Mapper → entry_points
  │
  └─ Saves: deliverables/code_analysis_deliverable.md
      Contains:
        • Tech stack (frontend, backend, infrastructure)
        • Open ports & services
        • Subdomains discovered
        • Security components (auth, session management)
        • Attack surfaces (API endpoints, form inputs)
        • Defensive mechanisms (WAF, rate limiting)
        • Critical file paths

                ↓

Recon Agent
  │
  ├─ Reads: deliverables/code_analysis_deliverable.md
  │   (knows tech stack, components, entry points)
  │
  ├─ Uses Playwright to navigate live app:
  │   ├─ Navigate to homepage
  │   ├─ Click through user flows
  │   ├─ Fill login forms
  │   ├─ Observe API calls
  │   └─ Document discovered functionality
  │
  ├─ Runs Task Agents (parallel):
  │   ├─ Route Mapper Agent → maps /api/* to handlers
  │   ├─ Authorization Checker → finds auth middleware
  │   ├─ Input Validator Agent → analyzes validation logic
  │   ├─ Session Handler Agent → traces session management
  │   └─ Auth Architecture Agent → maps role hierarchy
  │
  └─ Saves: deliverables/recon_deliverable.md
      Contains:
        • Executive summary (app purpose, tech stack)
        • Technology & Service Map (subdomains, ports)
        • Authentication Flow (mechanism, code pointers)
        • Role & Privilege Architecture (hierarchy, escalation paths)
        • API Endpoint Inventory (endpoints, roles, object IDs)
        • Input Vectors (URL params, POST fields, headers, cookies)
        • Network & Interaction Map (entities, flows, guards)
        • Authorization Vulnerability Candidates (prioritized testing list)

                ↓

Vulnerability Analysis Agents (5 parallel specialists)
  │
  ├─ injection-vuln agent
  │   └─ Uses: API Endpoint Inventory + Input Vectors
  │       → Tests SQL/command injection on all inputs
  │
  ├─ xss-vuln agent
  │   └─ Uses: Input Vectors + render contexts
  │       → Tests XSS on all reflection points
  │
  ├─ auth-vuln agent
  │   └─ Uses: Authentication Flow + Role Architecture
  │       → Tests auth bypass, session fixation
  │
  ├─ ssrf-vuln agent
  │   └─ Uses: API Endpoint Inventory + Network Map
  │       → Tests SSRF on all URL-accepting endpoints
  │
  └─ authz-vuln agent
      └─ Uses: Authorization Vulnerability Candidates
          → Tests IDOR, privilege escalation on prioritized endpoints

                ↓

Exploitation Agents (5 parallel)
  └─ Use: Vulnerability queues + code analysis
      → Craft exploits, verify impact, capture evidence
```

---

## Key Files Reference

### Reconnaissance Implementation Files

| File Path | Purpose | Key Content |
|-----------|---------|-------------|
| `src/tool-checker.ts` | External tool validation | Checks nmap, subfinder, whatweb availability |
| `src/constants.ts:35-62` | MCP_AGENT_MAPPING | Assigns pre-recon → playwright-agent1, recon → playwright-agent2 |
| `src/temporal/workflows.ts:142-149` | Pre-recon workflow | Phase 1 execution (logPhaseTransition, runPreReconAgent) |
| `src/temporal/workflows.ts:151-157` | Recon workflow | Phase 2 execution (logPhaseTransition, runReconAgent) |
| `src/temporal/activities.ts:309-311` | Pre-recon activity | runPreReconAgent wrapper |
| `src/temporal/activities.ts:313-315` | Recon activity | runReconAgent wrapper |

### Prompt Files

| File Path | Agent | Approach | MCP Assignment |
|-----------|-------|----------|----------------|
| `prompts/pre-recon-code.txt` | pre-recon | External tools + Task Agent code analysis | playwright-agent1 |
| `prompts/recon.txt` | recon | Playwright exploration + Task Agent correlation | playwright-agent2 |

### Deliverables

| File Path | Created By | Used By | Content |
|-----------|------------|---------|---------|
| `deliverables/code_analysis_deliverable.md` | pre-recon | recon, all vuln agents | Tech stack, security components, attack surfaces |
| `deliverables/recon_deliverable.md` | recon | all vuln agents | API inventory, auth flows, role hierarchy, input vectors |

---

## Summary: Shannon's Reconnaissance Advantage

### No Traditional Crawler

Shannon does NOT use automated web crawlers/spiders because:

1. **Blind crawling is inefficient:**
   - Wastes time on low-value pages (static content, help docs)
   - Misses authenticated functionality (can't login automatically)
   - Can't handle complex flows (multi-step wizards, MFA)

2. **AI-guided exploration is smarter:**
   - Navigates like a human user (login, explore features)
   - Focuses on high-value functionality (user data, admin panels)
   - Understands context (auth flows, API calls, session management)

3. **Source code correlation provides depth:**
   - Knows which endpoints exist before testing
   - Understands authorization architecture
   - Maps inputs to validation logic
   - Identifies business logic vulnerabilities

### The Power of Hybrid Reconnaissance

**Shannon = External Tools + Source Code Analysis + Live Testing**

```
External Tools (nmap, subfinder, whatweb)
    ↓
    + Source Code Analysis (Task Agents)
    ↓
    + Live Exploration (Playwright)
    ↓
    + Code Correlation (Task Agents)
    ↓
= Comprehensive Attack Surface Map
```

**Result:**
- **Faster** than full crawling (focused exploration)
- **Deeper** than DAST (source code awareness)
- **Smarter** than automated tools (AI-guided)
- **More accurate** than black-box testing (context-aware)

---

**END OF DOCUMENT**
