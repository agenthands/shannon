# Shannon Documentation Index

**Version:** 1.0
**Last Updated:** 2025-02-02
**Purpose:** Master index for all Shannon technical documentation

---

## Overview

Shannon is an AI-powered penetration testing framework that uses a hybrid approach combining:
- External security tools (nmap, subfinder, whatweb)
- Deep source code analysis (AI-powered static analysis)
- Live application exploration (Playwright browser automation)
- Intelligent exploitation (multi-modal payload delivery)

This documentation set provides complete technical details for understanding, using, and extending Shannon.

---

## Documentation Structure

### Core Architecture Documents

#### 1. [RECONNAISSANCE_ARCHITECTURE.md](./RECONNAISSANCE_ARCHITECTURE.md)
**Purpose:** High-level overview of Shannon's two-phase reconnaissance system

**Contents:**
- Executive summary of hybrid reconnaissance approach
- Phase 1: Pre-Reconnaissance (external tools + source code)
- Phase 2: Reconnaissance (live exploration + code correlation)
- Comparison with traditional DAST crawlers
- Tool integration and data flow
- Key files reference

**When to read:** First document to read for understanding Shannon's reconnaissance methodology

---

#### 2. [RECONNAISSANCE_IMPLEMENTATION_DETAILS.md](./RECONNAISSANCE_IMPLEMENTATION_DETAILS.md)
**Purpose:** Complete code snippets, file names, and prompts for all reconnaissance actions

**Contents:**
- External tool execution code (nmap, subfinder, whatweb, schemathesis)
- AI agent prompts with exact text
- Task Agent prompts (Architecture Scanner, Route Mapper, etc.)
- AI-guided exploration examples with Playwright
- Complete execution flows
- File paths with line numbers

**When to read:** When you need implementation details or want to understand how reconnaissance works at the code level

---

#### 3. [PAYLOAD_CREATION_AND_SENDING.md](./PAYLOAD_CREATION_AND_SENDING.md)
**Purpose:** Detailed documentation of payload creation and HTTP request sending mechanisms

**Contents:**
- Three-tier payload architecture (curl, Playwright, sqlmap)
- Payload creation prompts (exploit-injection.txt, exploit-xss.txt, etc.)
- HTTP request mechanisms (Bash, Playwright MCP, automated tools)
- Exploitation decision flow
- MCP server architecture for exploitation
- Key files reference

**When to read:** When studying exploitation mechanisms or understanding how payloads are crafted and delivered

---

#### 4. [VULNERABILITY_VALIDATION_AND_EXPLOITATION_CODE_REVIEW.md](./VULNERABILITY_VALIDATION_AND_EXPLOITATION_CODE_REVIEW.md)
**Purpose:** Deep dive into validation and exploitation mechanisms

**Contents:**
- Three-layer validation architecture
- Queue-based validation system
- Agent output validation
- Exploitation workflow
- Error handling and retry logic
- Security and reliability features
- Critical code paths

**When to read:** When studying validation, queuing, or exploitation workflow orchestration

---

#### 5. [PROMPT_ARCHITECTURE.md](./PROMPT_ARCHITECTURE.md)
**Purpose:** Complete catalog of all 13 agent prompts and their cross-phase coordination

**Contents:**
- Phase-by-phase prompt analysis
- Task Agent usage patterns
- Cross-phase coordination patterns (7 critical patterns)
- Data flow diagrams and handoff mechanisms
- Prompt design principles
- Pipeline-testing variants

**When to read:** When studying how AI agents are coordinated or prompt engineering patterns

---

### Quick Reference Guides

#### 6. [CLAUDE.md](./CLAUDE.md)
**Purpose:** Development guide for contributors working with Shannon codebase

**Contents:**
- Overview of Shannon architecture
- Commands for running pentests
- Architecture and component documentation
- Development commands
- Troubleshooting guide
- Key files and directories

**When to read:** If you're a developer working on Shannon itself

---

## Document Reading Order

### For Understanding Shannon Architecture

**Beginner Path:**
1. Start here: **RECONNAISSANCE_ARCHITECTURE.md** - Understand the hybrid approach
2. Then: **RECONNAISSANCE_IMPLEMENTATION_DETAILS.md** - See how it works
3. Then: **PAYLOAD_CREATION_AND_SENDING.md** - Understand exploitation
4. Finally: **VULNERABILITY_VALIDATION_AND_EXPLOITATION_CODE_REVIEW.md** - Deep dive into validation

**Advanced Path:**
1. **PROMPT_ARCHITECTURE.md** - Understand agent coordination
2. **VULNERABILITY_VALIDATION_AND_EXPLOITATION_CODE_REVIEW.md** - Study validation/exploitation
3. **RECONNAISSANCE_IMPLEMENTATION_DETAILS.md** - Review implementation details
4. **PAYLOAD_CREATION_AND_SENDING.md** - Study payload mechanisms

---

### For Implementing Features

**If working on reconnaissance:**
- Read: **RECONNAISSANCE_IMPLEMENTATION_DETAILS.md**
- Reference files: `src/phases/pre-recon.ts`, `prompts/pre-recon-code.txt`, `prompts/recon.txt`

**If working on exploitation:**
- Read: **PAYLOAD_CREATION_AND_SENDING.md**
- Reference files: `prompts/exploit-*.txt`, `src/constants.ts` (MCP_AGENT_MAPPING)

**If working on validation:**
- Read: **VULNERABILITY_VALIDATION_AND_EXPLOITATION_CODE_REVIEW.md**
- Reference files: `src/queue-validation.ts`, `src/ai/claude-executor.ts`

**If working on prompts:**
- Read: **PROMPT_ARCHITECTURE.md**
- Reference files: `prompts/*.txt`

---

## Key Concepts Summary

### Two-Phase Reconnaissance

**Phase 1: Pre-Reconnaissance**
- External tools: nmap, subfinder, whatweb, schemathesis
- Source code analysis: AI-powered static analysis via Task Agents
- Output: Technology stack, security components, attack surfaces
- File: `deliverables/pre_recon_deliverable.md`

**Phase 2: Reconnaissance**
- Live exploration: Playwright browser automation (AI-guided, no crawler)
- Code correlation: Task Agents map endpoints to source code
- Output: API inventory, auth flows, role hierarchy, input vectors
- File: `deliverables/recon_deliverable.md`

### Why No Web Crawler?

Shannon uses **AI-guided exploration** instead of automated crawlers because:
- ✅ Focuses on high-value functionality (not static pages)
- ✅ Handles authenticated flows intelligently
- ✅ Understands business logic and context
- ✅ Correlates observations with source code
- ✅ Faster than crawling everything

Traditional crawlers:
- ❌ Waste time on low-value pages
- ❌ Can't handle authentication automatically
- ❌ Don't understand application context
- ❌ High false positives

### Three-Tier Payload Architecture

**Tier 1: Bash + curl**
- Use: Simple one-off payloads
- Full control over HTTP requests
- Fast for basic testing

**Tier 2: Playwright MCP**
- Use: Complex multi-step attacks
- Browser state management (cookies, sessions)
- Screenshot evidence capture

**Tier 3: Automated Tools (sqlmap)**
- Use: Systematic enumeration
- Database mapping and data extraction
- Automated exploitation

### Validation Architecture

**Three Layers:**
1. **MCP Tool Validation** (Real-time) - Validates on save_deliverable call
2. **Agent Output Validation** (Post-execution) - Checks deliverable existence
3. **Queue Validation** (Pre-exploitation) - Validates JSON structure and vulnerability count

**Result:** Only successfully exploited vulnerabilities reach final report ("No Exploit, No Report")

---

## File Structure Reference

### Core Implementation

```
shannon/
├── src/
│   ├── phases/
│   │   └── pre-recon.ts              # External tool execution
│   ├── temporal/
│   │   ├── workflows.ts              # Workflow orchestration
│   │   ├── activities.ts             # Activity implementations
│   │   └── worker.ts                 # Worker process
│   ├── ai/
│   │   └── claude-executor.ts        # Claude Agent SDK integration
│   ├── tool-checker.ts               # Tool availability validation
│   ├── queue-validation.ts           # Queue validation pipeline
│   └── constants.ts                  # Configuration mappings
├── prompts/
│   ├── pre-recon-code.txt            # Pre-recon AI agent prompt
│   ├── recon.txt                     # Recon AI agent prompt
│   ├── vuln-*.txt                    # Vulnerability analysis prompts
│   └── exploit-*.txt                 # Exploitation prompts
├── mcp-server/
│   └── src/
│       ├── index.ts                   # MCP server factory
│       └── tools/
│           └── save-deliverable.ts   # Deliverable saving with validation
└── deliverables/
    ├── code_analysis_deliverable.md  # Phase 1: Source code analysis
    ├── pre_recon_deliverable.md      # Phase 1: External tool results
    └── recon_deliverable.md          # Phase 2: Attack surface map
```

---

## Common Workflows

### Running a Pentest

```bash
# Basic usage
./shannon start URL=https://example.com REPO=/path/to/source

# With authentication
./shannon start URL=https://example.com REPO=/path/to/source CONFIG=./configs/app-config.yaml

# Pipeline testing (fast iteration)
./shannon start URL=https://example.com REPO=/path/to/source PIPELINE_TESTING=true

# Custom output directory
./shannon start URL=https://example.com REPO=/path/to/source OUTPUT=./my-pentest
```

### Monitoring Progress

```bash
# View worker logs
./shannon logs

# Query specific workflow
./shannon query ID=<workflow-id>

# Temporal Web UI
open http://localhost:8233
```

### Stopping Shannon

```bash
# Stop containers (preserves workflow data)
./shannon stop

# Full cleanup (removes volumes)
./shannon stop CLEAN=true
```

---

## Architecture Diagrams

### Five-Phase Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│ SHANNON FIVE-PHASE WORKFLOW                                     │
└─────────────────────────────────────────────────────────────────┘

Phase 1: Pre-Reconnaissance (sequential)
  ├─ External tools: nmap, subfinder, whatweb
  └─ AI agent: Source code analysis
  Output: deliverables/pre_recon_deliverable.md

Phase 2: Reconnaissance (sequential)
  ├─ Playwright: Live application exploration
  └─ Task Agents: Source code correlation
  Output: deliverables/recon_deliverable.md

Phase 3: Vulnerability Analysis (5 parallel agents)
  ├─ injection-vuln  → injection_exploitation_queue.json
  ├─ xss-vuln        → xss_exploitation_queue.json
  ├─ auth-vuln       → auth_exploitation_queue.json
  ├─ ssrf-vuln       → ssrf_exploitation_queue.json
  └─ authz-vuln      → authz_exploitation_queue.json

Phase 4: Exploitation (5 parallel agents, conditional)
  ├─ injection-exploit (runs if queue has vulnerabilities)
  ├─ xss-exploit (runs if queue has vulnerabilities)
  ├─ auth-exploit (runs if queue has vulnerabilities)
  ├─ ssrf-exploit (runs if queue has vulnerabilities)
  └─ authz-exploit (runs if queue has vulnerabilities)

Phase 5: Reporting (sequential)
  ├─ Assemble evidence from exploitation
  ├─ Generate comprehensive security report
  └─ Inject model metadata
  Output: comprehensive_security_assessment_report.md
```

### Data Flow

```
External Tools (nmap, subfinder, whatweb)
    ↓
Source Code Analysis (Task Agents)
    ↓
Pre-Recon Deliverable (tech stack, attack surfaces)
    ↓
Live Exploration (Playwright)
    ↓
Code Correlation (Task Agents)
    ↓
Recon Deliverable (API inventory, auth flows, input vectors)
    ↓
Vulnerability Analysis (5 specialist agents)
    ↓
Exploitation Queues (vulnerability lists with externally_exploitable flag)
    ↓
Conditional Exploitation (only if queue has vulnerabilities)
    ↓
Exploitation Evidence (proof of exploitation)
    ↓
Final Report (executive summary + evidence)
```

---

## Troubleshooting

### Common Issues

**"Repository not found"**
- Ensure target local directory exists and is accessible
- Check REPO path is correct

**"Temporal not ready"**
- Wait for health check
- Run: `docker compose logs temporal`

**"Worker not processing"**
- Ensure worker container is running: `docker compose ps`
- Check worker logs: `./shannon logs`

**"Missing tools"**
- nmap: `brew install nmap` (macOS) or `apt install nmap` (Ubuntu)
- subfinder: `go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest`
- whatweb: `gem install whatweb`

### Resetting State

```bash
# Reset workflow state
./shannon stop CLEAN=true  # Removes all Temporal data and volumes

# Or just restart services
docker compose restart
```

---

## Contributing

When contributing to Shannon:

1. **Adding a new agent:**
   - Define agent in `src/session-manager.ts`
   - Create prompt in `prompts/`
   - Add activity in `src/temporal/activities.ts`
   - Register in `src/temporal/workflows.ts`

2. **Modifying prompts:**
   - Test with `PIPELINE_TESTING=true` for faster iteration
   - Follow prompt design principles in PROMPT_ARCHITECTURE.md
   - Update cross-phase coordination if needed

3. **Validation changes:**
   - Update `src/queue-validation.ts` for queue rules
   - Update `src/ai/claude-executor.ts` for output validation
   - Update `src/constants.ts` for validator mappings

---

## Additional Resources

- **Temporal Documentation:** https://docs.temporal.io
- **Claude Agent SDK:** https://docs.anthropic.com/claude-agent-sdk
- **Playwright MCP:** https://github.com/modelcontextprotocol/servers
- **OWASP Testing Guide:** https://owasp.org/www-project-web-security-testing-guide

---

## Document Metadata

**Version History:**
- 1.0 (2025-02-02) - Initial documentation set

**Maintainers:**
- Keygraph, Inc. (https://keygraphhq.com)

**License:**
- GNU Affero General Public License v3.0

---

**END OF INDEX**
