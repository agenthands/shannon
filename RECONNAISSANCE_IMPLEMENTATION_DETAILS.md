# Shannon Reconnaissance Implementation Details

**Document Version:** 1.0  
**Last Updated:** 2025-02-02  
**Purpose:** Complete code snippets, file names, and prompts for all reconnaissance actions

---

## Phase 1: Pre-Reconnaissance Implementation

### File: src/phases/pre-recon.ts

This file contains the complete implementation of Phase 1 reconnaissance.

---

### 1. Tool Availability Check

**File:** src/tool-checker.ts:15-38

```typescript
export type ToolName = 'nmap' | 'subfinder' | 'whatweb' | 'schemathesis';

export const checkToolAvailability = async (): Promise<ToolAvailability> => {
  const tools: ToolName[] = ['nmap', 'subfinder', 'whatweb', 'schemathesis'];
  const availability: ToolAvailability = {
    nmap: false,
    subfinder: false,
    whatweb: false,
    schemathesis: false
  };

  console.log(chalk.blue('Checking tool availability...'));

  for (const tool of tools) {
    try {
      await $`command -v ${tool}`;
      availability[tool] = true;
      console.log(chalk.green(`  ${tool} - available`));
    } catch {
      availability[tool] = false;
      console.log(chalk.yellow(`  ${tool} - not found`));
    }
  }

  return availability;
};
```

---

### 2. External Tool Execution

**File:** src/phases/pre-recon.ts:67-139

#### nmap Execution

```typescript
case 'nmap': {
  console.log(chalk.blue('Running nmap scan...'));
  const nmapHostname = new URL(target).hostname;
  result = await $({ silent: true, stdio: ['ignore', 'pipe', 'ignore'] })\`nmap -sV -sC ${nmapHostname}\`;
  const duration = timer.stop();
  console.log(chalk.green('nmap completed'));
  return { tool: 'nmap', output: result.stdout, status: 'success', duration };
}
```

**Command:** \`nmap -sV -sC <hostname>\`
- \`-sV\` - Service version detection
- \`-sC\` - Default script scan

#### subfinder Execution

```typescript
case 'subfinder': {
  console.log(chalk.blue('Running subfinder scan...'));
  const hostname = new URL(target).hostname;
  result = await $({ silent: true, stdio: ['ignore', 'pipe', 'ignore'] })\`subfinder -d ${hostname}\`;
  return { tool: 'subfinder', output: result.stdout, status: 'success', duration };
}
```

**Command:** \`subfinder -d <hostname>\`

#### whatweb Execution

```typescript
case 'whatweb': {
  console.log(chalk.blue('Running whatweb scan...'));
  result = await $({ silent: true, stdio: ['ignore', 'pipe', 'ignore'] })\`whatweb --open-timeout 30 --read-timeout 60 ${target}\`;
  return { tool: 'whatweb', output: result.stdout, status: 'success', duration };
}
```

**Command:** \`whatweb --open-timeout 30 --read-timeout 60 <target>\`

---

### 3. Wave 1: Parallel Execution

**File:** src/phases/pre-recon.ts:142-208

```typescript
async function runPreReconWave1(...) {
  const operations = [
    runTerminalScan('nmap', webUrl),        // External tool
    runTerminalScan('subfinder', webUrl),   // External tool
    runTerminalScan('whatweb', webUrl),     // External tool
    runClaudePromptWithRetry(...)            // AI code analysis
  ];

  const [nmap, subfinder, whatweb, codeAnalysis] = await Promise.all(operations);
  return { nmap, subfinder, whatweb, codeAnalysis };
}
```

**Key:** All 4 operations run in parallel via \`Promise.all()\`

---

### 4. AI Agent Prompt: Pre-Recon

**File:** prompts/pre-recon-code.txt

**Task Agent Strategy (lines 90-153):**

**Phase 1: Discovery Agents (Launch in Parallel)**

\`\`\`
Launch these three discovery agents simultaneously:

1. Architecture Scanner Agent:
   "Map the application's structure, technology stack, and critical components.
   Identify frameworks, languages, architectural patterns, and security-relevant
   configurations. Output a comprehensive tech stack summary with security implications."

2. Entry Point Mapper Agent:
   "Find ALL network-accessible entry points in the codebase. Catalog API endpoints,
   web routes, webhooks, file uploads. Also identify API schema files (OpenAPI/Swagger
   *.json/*.yaml, GraphQL *.graphql). Distinguish between public and authenticated endpoints."

3. Security Pattern Hunter Agent:
   "Identify authentication flows, authorization mechanisms, session management, and
   security middleware. Find JWT handling, OAuth flows, RBAC implementations, permission
   validators. Map the complete security architecture with exact file locations."
\`\`\`

**Phase 2: Vulnerability Analysis Agents**

\`\`\`
4. XSS/Injection Sink Hunter Agent:
   "Find all dangerous sinks where untrusted input could execute: XSS sinks (innerHTML,
   document.write), SQL injection points, command injection (exec, system), file inclusion
   (fopen, include), template injection (render, compile), deserialization sinks."

5. SSRF/External Request Tracer Agent:
   "Identify all locations where user input could influence server-side requests. Find
   HTTP clients, URL fetchers, webhook handlers, external API integrations."

6. Data Security Auditor Agent:
   "Trace sensitive data flows, encryption implementations, secret management patterns,
   and database security controls. Identify PII handling and compliance-relevant code."
\`\`\`

---

## Phase 2: Reconnaissance Implementation

### File: src/temporal/workflows.ts:151-157

```typescript
// === Phase 2: Reconnaissance ===
state.currentPhase = 'recon';
state.currentAgent = 'recon';
await a.logPhaseTransition(activityInput, 'recon', 'start');
state.agentMetrics['recon'] = await a.runReconAgent(activityInput);
state.completedAgents.push('recon');
await a.logPhaseTransition(activityInput, 'recon', 'complete');
```

---

### AI Agent Prompt: Recon

**File:** prompts/recon.txt

**Systematic Approach (lines 105-134):**

**Step 1: Synthesize Initial Data**
\`\`\`
- Read deliverables/pre_recon_deliverable.md
- Create preliminary list of known technologies, subdomains, ports
\`\`\`

**Step 2: Interactive Application Exploration (AI-guided via Playwright)**
\`\`\`
- Use {{MCP_SERVER}}__browser_navigate to navigate to target
- Map user-facing functionality: login forms, registration flows, password reset
- Document multi-step processes
- Observe network requests to identify API calls
\`\`\`

**Step 3: Correlate with Source Code (Parallel Task Agents)**
\`\`\`
Launch specialized Task agents in parallel:

- Route Mapper Agent: "Find all backend routes and controllers for discovered endpoints.
  Map each endpoint to its exact handler function with file paths and line numbers."

- Authorization Checker Agent: "For each endpoint, find authorization middleware, guards,
  and permission checks. Map authorization flow with exact code locations."

- Input Validator Agent: "Analyze input validation logic for all discovered form fields
  and API parameters. Find validation rules, sanitization, and data processing."

- Session Handler Agent: "Trace complete session and authentication token handling for
  discovered auth flows. Map session creation, storage, validation, destruction."
\`\`\`

**Step 3.5: Authorization Architecture Analysis**
\`\`\`
- Authorization Architecture Agent: "Map all user roles, hierarchies, permission models,
  authorization decision points, object ownership patterns, role-based access patterns."
\`\`\`

---

## AI-Guided Exploration Examples

### Example 1: Authentication Flow Discovery

**Step 1: Browser Exploration**
\`\`\`javascript
// Navigate to application
await browser_navigate("https://target.com")

// Observe login form
await browser_snapshot()
// Shows: #email, #password fields, #login-button

// Attempt login
await browser_type("#email", "test@example.com")
await browser_type("#password", "password123")
await browser_click("#login-button")

// Wait for navigation
await browser_wait_for(text="Dashboard", timeGone=5)

// Observe network requests (automatic):
// POST /api/auth/login → 200 OK, JWT token returned
// Set-Cookie: session_id=abc123; HttpOnly; Secure
\`\`\`

**Step 2: Parallel Task Agent Launch**
\`\`\`
Task Agent 1 - Route Mapper:
"Find handlers for: /api/auth/login, /api/users/me"

Task Agent 2 - Authorization Checker:
"Find auth middleware for: /api/auth/login, /api/users/me"

Task Agent 3 - Session Handler:
"Trace session management for /api/auth/login flow"
\`\`\`

**Step 3: Task Agent Returns**
\`\`\`
Route Mapper:
→ POST /api/auth/login → src/controllers/auth.controller.ts:45 (loginHandler)
→ GET  /api/users/me    → src/controllers/users.controller.ts:78 (getCurrentUser)

Authorization Checker:
→ /api/auth/login: No auth required (public)
→ /api/users/me: Bearer token middleware (src/middleware/auth.ts:15)

Session Handler:
→ JWT generation: src/services/jwt.ts:23 (signToken)
→ Cookie config: src/services/session.ts:56 (HttpOnly, Secure, SameSite)
→ Session storage: Redis (src/config/redis.ts:12)
\`\`\`

---

### Example 2: Password Reset Flow Discovery

**Browser Exploration:**
\`\`\`javascript
// Navigate to login
await browser_navigate("https://target.com/login")

// Click "Forgot Password"
await browser_click("#forgot-password-link")

// Enter email
await browser_type("#email", "user@example.com")
await browser_click("#submit-reset")

// Observe: POST /api/auth/forgot-password
// Response: { "message": "Reset email sent" }

// Navigate to reset link
await browser_navigate("https://target.com/reset-password?token=abc123")

// Enter new password
await browser_type("#new-password", "NewPass123!")
await browser_type("#confirm-password", "NewPass123!")
await browser_click("#submit-new-password")

// Observe: POST /api/auth/reset-password
// Response: { "message": "Password updated" }
\`\`\`

**Task Agent Correlation:**
\`\`\`
Session Handler Agent:
"Trace password reset flow:
- /api/auth/forgot-password → src/controllers/auth.controller.ts:120
- /api/auth/reset-password → src/controllers/auth.controller.ts:145

Find:
- Reset token generation (src/services/tokens.ts:34)
- Token validation (src/middleware/token-validator.ts:23)
- Password update (src/services/user.ts:89)
- Token expiration (src/services/tokens.ts:45)"
\`\`\`

---

### Example 3: API Endpoint Discovery

**Browser Exploration:**
\`\`\`javascript
// Login
await browser_navigate("https://target.com/login")
await browser_type("#email", "test@example.com")
await browser_type("#password", "password")
await browser_click("#login-button")

// Navigate to profile
await browser_navigate("https://target.com/profile")
// Network: GET /api/users/me → { "id": 123, "role": "user" }

// Navigate to orders
await browser_navigate("https://target.com/orders")
// Network: GET /api/orders → [ { "id": 1, "status": "pending" } ]

// Click order
await browser_click("#order-1")
// Network: GET /api/orders/1 → { "id": 1, "items": [...] }

// Try admin panel
await browser_navigate("https://target.com/admin")
// Network: GET /api/admin/users → 403 Forbidden
\`\`\`

**Parallel Task Agents:**
\`\`\`
Route Mapper:
"Find handlers for:
- GET /api/users/me
- GET /api/orders
- GET /api/orders/{id}
- GET /api/admin/users"

Authorization Checker:
"For each endpoint, find:
- Authorization middleware
- Object ID parameters
- Ownership validation"

Input Validator:
"For inputs, find validation rules and sanitization"
\`\`\`

---

## Complete Execution Flow

### Pre-Recon Full Flow

\`\`\`typescript
// 1. Workflow starts (src/temporal/workflows.ts:142)
state.currentPhase = 'pre-recon';
await a.runPreReconAgent(activityInput);

// 2. Execute phase (src/phases/pre-recon.ts:351)
const wave1Results = await runPreReconWave1(...);
const wave2Results = await runPreReconWave2(...);
const report = await stitchPreReconOutputs(...);

// 3. Wave 1: Parallel execution (src/phases/pre-recon.ts:180)
const operations = [
  runTerminalScan('nmap', webUrl),      // External
  runTerminalScan('subfinder', webUrl), // External
  runTerminalScan('whatweb', webUrl),   // External
  runClaudePromptWithRetry(...)          // AI agent
];
const [nmap, subfinder, whatweb, codeAnalysis] = await Promise.all(operations);

// 4. AI agent launches Task Agents (prompts/pre-recon-code.txt)
// Phase 1: Architecture Scanner, Entry Point Mapper, Security Pattern Hunter
// Phase 2: XSS Sink Hunter, SSRF Tracer, Data Security Auditor

// 5. Output files created
deliverables/code_analysis_deliverable.md  // AI analysis
deliverables/pre_recon_deliverable.md       // Combined
\`\`\`

### Recon Full Flow

\`\`\`typescript
// 1. Workflow starts (src/temporal/workflows.ts:151)
state.currentPhase = 'recon';
await a.runReconAgent(activityInput);

// 2. AI agent executes (prompts/recon.txt)
// Step 1: Read pre_recon_deliverable.md

// Step 2: Browser exploration (Playwright)
await browser_navigate("https://target.com")
// Click through flows, observe network requests

// Step 3: Parallel Task Agents
Task Agent 1 - Route Mapper: "Find handlers for discovered endpoints"
Task Agent 2 - Authorization Checker: "Find auth middleware"
Task Agent 3 - Input Validator: "Analyze validation"
Task Agent 4 - Session Handler: "Trace session management"
Task Agent 5 - Authorization Architecture: "Map role hierarchy"

// Step 4: Synthesize findings
// Create recon_deliverable.md with:
// - API Endpoint Inventory
// - Authentication Flow
// - Role Architecture
// - Input Vectors
// - Network Map
// - Vulnerability Candidates

// 3. Output file created
deliverables/recon_deliverable.md
\`\`\`

---

## Key Files Reference

| File Path | Purpose |
|-----------|---------|
| src/phases/pre-recon.ts | Pre-recon execution |
| src/tool-checker.ts | Tool availability check |
| src/temporal/workflows.ts:142-157 | Workflow orchestration |
| src/temporal/activities.ts:309-315 | Activity wrappers |
| prompts/pre-recon-code.txt | Pre-recon AI prompt |
| prompts/recon.txt | Recon AI prompt |
| deliverables/pre_recon_deliverable.md | Phase 1 output |
| deliverables/recon_deliverable.md | Phase 2 output |

---

**END OF DOCUMENT**
