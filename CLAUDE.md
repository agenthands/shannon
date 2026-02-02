# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Shannon is a sophisticated AI-powered penetration testing framework that emulates human pentesters through a multi-agent architecture. It combines white-box source code analysis with black-box dynamic exploitation, using Anthropic's Claude Agent SDK as its reasoning engine and Temporal for durable workflow orchestration.

**Core Design Philosophy:** Shannon uses a "proof-by-exploitation" model—only vulnerabilities that are successfully exploited in the real world are reported, eliminating false positives.

## Commands

### Prerequisites
- **Docker** - Container runtime
- **Anthropic API key** - Set in `.env` file

### Running the Penetration Testing Agent (Docker + Temporal)
```bash
# Configure credentials
cp .env.example .env
# Edit .env:
#   ANTHROPIC_API_KEY=your-key
#   CLAUDE_CODE_MAX_OUTPUT_TOKENS=64000  # Prevents token limits during long reports

# Start a pentest workflow
./shannon start URL=<url> REPO=<path>
```

Examples:
```bash
./shannon start URL=https://example.com REPO=/path/to/repo
./shannon start URL=https://example.com REPO=/path/to/repo CONFIG=./configs/my-config.yaml
./shannon start URL=https://example.com REPO=/path/to/repo OUTPUT=./my-reports
```

### Monitoring Progress
```bash
./shannon logs                      # View real-time worker logs
./shannon query ID=<workflow-id>    # Query specific workflow progress
# Temporal Web UI available at http://localhost:8233
```

### Stopping Shannon
```bash
./shannon stop                      # Stop containers (preserves workflow data)
./shannon stop CLEAN=true           # Full cleanup including volumes
```

### Options
```bash
CONFIG=<file>          YAML configuration file for authentication and testing parameters
OUTPUT=<path>          Custom output directory for session folder (default: ./audit-logs/)
PIPELINE_TESTING=true  Use minimal prompts and fast retry intervals (10s instead of 5min)
REBUILD=true           Force Docker rebuild with --no-cache (use when code changes aren't picked up)
ROUTER=true            Route requests through claude-code-router for multi-model support
```

### Generate TOTP for Authentication
TOTP generation is handled automatically via the `generate_totp` MCP tool during authentication flows.

### Development Commands
```bash
# Build TypeScript
npm run build

# Development Temporal commands (without Docker)
npm run temporal:server              # Start Temporal server only
npm run temporal:server:stop         # Stop Temporal server
npm run temporal:worker              # Run worker process
npm run temporal:start               # Start workflow via client
npm run temporal:query               # Query workflow progress

# Run with pipeline testing mode (fast, minimal deliverables, 10s retry vs 5min)
./shannon start URL=<url> REPO=<path> PIPELINE_TESTING=true
```

## Architecture & Components

### Core Architecture Patterns

**Pipelined Parallel Execution:**
Unlike traditional barrier-based parallelism, Shannon uses a sophisticated pipelining model where each vulnerability-exploit pair operates independently:
```
vuln-agent → queue-check → conditional exploit-agent
```
Each exploit starts immediately when its vuln agent finishes, not waiting for all vulnerabilities to complete. This provides ~5x faster execution than sequential processing.

**Git Checkpoint System:**
Every agent execution uses git checkpoints for workspace integrity:
1. Create checkpoint before agent starts
2. Commit on successful completion
3. Rollback on failure
4. Ensures clean workspace for each agent

**No Exploit, No Report:**
The system enforces strict validation—only successfully exploited vulnerabilities are reported. Hypotheses that cannot be proven are discarded as false positives.

**Activity Heartbeats:**
All long-running activities send heartbeats every 3 minutes to prevent Temporal timeout during resource-intensive operations (like LLM queries).

### Core Modules
- `src/config-parser.ts` - YAML configuration parsing, JSON Schema validation, and config distribution to agents
- `src/error-handling.ts` - Categorized error types (PentestError, ConfigError, NetworkError) with intelligent retry logic
- `src/tool-checker.ts` - Validates availability of external security tools (nmap, subfinder, whatweb) before execution
- `src/session-manager.ts` - Agent definitions, execution order, and prerequisite graph (AGENT_QUEUE, parallel groups)
- `src/queue-validation.ts` - Validates agent deliverables and determines if exploitation should proceed

### Temporal Orchestration Layer
Shannon uses Temporal for durable workflow orchestration with crash recovery:
- `src/temporal/shared.ts` - Types, interfaces, query definitions (PipelineState, getProgress handler)
- `src/temporal/workflows.ts` - Main workflow (pentestPipelineWorkflow) with pipelined parallel execution
- `src/temporal/activities.ts` - Activity implementations with heartbeats, agent execution wrappers
- `src/temporal/worker.ts` - Worker process entry point with ActivityManager
- `src/temporal/client.ts` - CLI client for starting workflows
- `src/temporal/query.ts` - Query tool for progress inspection

**Retry Configuration:**
- Production: 5min initial interval, 30min max, 50 attempts (for billing errors)
- Testing (PIPELINE_TESTING=true): 10s initial, 30s max, 5 attempts (fast iteration)
- Non-retryable errors: AuthenticationError, PermissionError, ConfigurationError, InvalidTargetError

**Key Features:**
- **Crash recovery** - Workflows resume automatically after worker restart via durable state
- **Queryable progress** - Real-time status via `./shannon query` or Temporal Web UI (http://localhost:8233)
- **Intelligent retry** - Distinguishes transient (billing) vs permanent (config) errors
- **Graceful failure** - Promise.allSettled ensures one pipeline failure doesn't stop others

### Five-Phase Testing Workflow

1. **Pre-Reconnaissance** (`pre-recon`) - External tool scans (nmap, subfinder, whatweb) + source code analysis
2. **Reconnaissance** (`recon`) - Analysis of initial findings and attack surface mapping
3. **Vulnerability Analysis** (5 agents in parallel) - Each hunts for specific vulnerability types
4. **Exploitation** (5 agents in parallel, conditional) - Only runs if vulnerabilities found in corresponding vuln agent
5. **Reporting** (`report`) - Executive-level security report with proven findings

**Agent Prerequisite Graph:**
```
pre-recon → recon → {injection-vuln, xss-vuln, auth-vuln, authz-vuln, ssrf-vuln}
                  → [queue validation]
                  → {injection-exploit, xss-exploit, auth-exploit, authz-exploit, ssrf-exploit}
                  → report
```

### Configuration System
YAML-based configuration with JSON Schema validation (`configs/config-schema.json`):
- Authentication: form, SSO, API, basic auth with TOTP/MFA support
- Login flows: Step-by-step instructions for AI to follow
- Rules: Paths to avoid or focus on during testing
- Application-specific parameters

**Key config files:**
- `configs/example-config.yaml` - Template with all options
- `configs/juice-shop-config.yaml` - OWASP Juice Shop example
- `configs/keygraph-config.yaml`, `chatwoot-config.yaml`, `metabase-config.yaml`, `cal-com-config.yaml` - App-specific

**Variable substitution in prompts:**
- `{{TARGET_URL}}` - Target application URL
- `{{CONFIG_CONTEXT}}` - YAML config as formatted string
- `{{LOGIN_INSTRUCTIONS}}` - Auth flow from config

### Prompt Templates
The `prompts/` directory contains specialized prompt templates for each testing phase:
- `pre-recon-code.txt` - Initial code analysis prompts
- `recon.txt` - Reconnaissance analysis prompts
- `vuln-*.txt` - Vulnerability assessment prompts (injection, XSS, auth, authz, SSRF)
- `exploit-*.txt` - Exploitation attempt prompts
- `report-executive.txt` - Executive report generation prompts
- `shared/login-instructions.txt` - Login flow template included via prompt-manager.ts

### Claude Agent SDK Integration
The agent uses `@anthropic-ai/claude-agent-sdk` with maximum autonomy:
- `maxTurns: 10_000` - Allows extensive autonomous analysis
- `permissionMode: 'bypassPermissions'` - Full system access for thorough testing
- Playwright MCP integration for web browser automation
- Working directory set to target local repository
- Configuration context injection for authenticated testing
- **Router mode support** - Can route to alternative LLM providers (OpenAI, OpenRouter) via claude-code-router

### MCP Server (In-Process Tools)
The `mcp-server/` directory provides custom MCP tools for agents:
- `save-deliverable` - Save analysis results to deliverables directory
- `generate-totp` - Generate TOTP tokens for 2FA authentication
- Validates deliverable format and queue data for exploitation decisions
- Built into the worker process, available to all agents

### Audit & Metrics System
Crash-safe audit system with comprehensive tracking:

**Output Structure:**
```
audit-logs/{hostname}_{sessionId}/
├── session.json          # Metrics, costs, timing (atomic writes)
├── agents/               # Turn-by-turn execution logs
├── prompts/              # Prompt snapshots for reproducibility
└── deliverables/         # Security reports and findings
```

**Crash Safety:**
- Append-only logging with immediate flush (survives kill -9)
- Atomic writes for session.json (no partial writes)
- Event-based logging (tool_start, tool_end, llm_response)
- SessionMutex prevents race conditions during parallel execution

**Metrics Tracked:**
- Phase-level and agent-level timing/cost aggregations
- Turn counts per agent
- Validation results
- Total cost USD, total duration, total turns


## Development Notes

### Understanding the Codebase

**Key Architectural Concepts:**

1. **Queue-Based Validation** (`src/queue-validation.ts`)
   - Vulnerability agents produce `*_exploitation_queue.json` files
   - Exploit agents check these queues before running
   - If queue is empty or has 0 vulnerabilities, exploit is skipped
   - This implements the "No Exploit, No Report" policy

2. **Agent Execution Flow** (`src/ai/claude-executor.ts`)
   - Git checkpoint → Run agent → Success or Rollback
   - Heartbeats sent every 3 minutes to Temporal
   - Audit logging captures all LLM interactions
   - Retry logic handles billing errors and transient failures

3. **Parallel Pipelined Execution** (`src/temporal/workflows.ts:runVulnExploitPipeline`)
   - Each vuln-exploit pair is independent
   - Uses Promise.allSettled for graceful failure handling
   - No synchronization barrier between pipelines
   - Exploitation starts immediately after its vuln agent completes

4. **MCP Integration Pattern**
   - In-process MCP server (`mcp-server/`) provides custom tools
   - Playwright MCP for browser automation (assigned per agent)
   - Tool responses validate and save deliverables
   - TOTP generation for 2FA authentication

**File Reading Order for Understanding:**
1. `src/session-manager.ts` - Agent definitions and prerequisites
2. `src/temporal/workflows.ts` - Main workflow orchestration
3. `src/temporal/activities.ts` - Activity implementations
4. `src/queue-validation.ts` - Queue validation logic
5. `src/ai/claude-executor.ts` - Agent execution wrapper
6. `mcp-server/src/index.ts` - Custom MCP tools

### Adding a New Agent
1. Define agent in `src/session-manager.ts` (add to AGENTS constant and AGENT_ORDER)
2. Create prompt template in `prompts/` (e.g., `vuln-newtype.txt` or `exploit-newtype.txt`)
3. Add activity function in `src/temporal/activities.ts` (wraps `runAgent` with specific prompt)
4. Register activity in `src/temporal/workflows.ts`:
   - Sequential agents: Add to phase sequence
   - Parallel vuln agents: Add to `runVulnExploitPipeline` array
   - Parallel exploit agents: Add to conditional execution in pipeline
5. Add queue configuration in `src/queue-validation.ts` (if vuln-exploit pair)
6. Update MCP agent mapping in `src/constants.ts` (if Playwright needed)
7. Build with `npm run build`

### Modifying Prompts
- Prompt templates use variable substitution: `{{TARGET_URL}}`, `{{CONFIG_CONTEXT}}`, `{{LOGIN_INSTRUCTIONS}}`
- Shared partials in `prompts/shared/` are included via `prompt-manager.ts`
- Test changes with `PIPELINE_TESTING=true` for faster iteration (10s retry vs 5min)

### Testing vs Production Mode

**Testing Mode** (`PIPELINE_TESTING=true`):
- 10s retry intervals vs 5min production
- 5 maximum attempts vs 50 production
- Minimal prompts for faster iteration
- Useful for prompt development and workflow testing

**Production Mode** (default):
- Long retry intervals for billing error recovery
- Maximum attempt count for long-running tests
- Full prompts for thorough analysis
- Optimized for actual penetration testing

### Security Focus
This is explicitly designed as a **defensive security tool** for:
- Vulnerability assessment of owned applications
- Security testing with explicit authorization
- Identifying exploitable vulnerabilities before production deployment

**Important:** Only use on systems you own or have explicit written permission to test.

## Key Files & Directories

**Entry Points:**
- `src/temporal/workflows.ts` - Main workflow orchestration with parallel pipelines
- `src/temporal/activities.ts` - Activity implementations with heartbeats
- `src/temporal/worker.ts` - Worker process with ActivityManager
- `src/temporal/client.ts` - CLI client for starting workflows

**Core Logic:**
- `src/session-manager.ts` - Agent definitions, prerequisites, execution order
- `src/ai/claude-executor.ts` - Claude Agent SDK wrapper with git checkpoints
- `src/config-parser.ts` - YAML config parsing with JSON Schema validation
- `src/queue-validation.ts` - Queue-based exploitation decision logic
- `src/audit/` - Crash-safe logging, metrics, session tracking

**Configuration:**
- `shannon` - Main CLI script for running pentests
- `docker/docker-compose.yml` - Temporal server + worker containers
- `configs/` - YAML configs and JSON schema
- `prompts/` - AI prompt templates for each agent

**External Tools:**
- External tools (nmap, subfinder, whatweb) are validated before use
- Can be skipped in testing mode for faster iteration

**Output:**
- `audit-logs/{hostname}_{sessionId}/` or custom OUTPUT path
- Contains session.json, agent logs, prompts, deliverables

## Router Mode (Multi-Model Support)

Shannon supports routing requests through alternative LLM providers via [claude-code-router](https://github.com/musistudio/claude-code-router). This is **experimental and unsupported**.

**Enable router mode:**
```bash
# In .env:
ROUTER_DEFAULT=openai,gpt-5.2  # or openrouter,google/gemini-3-flash-preview

# Run:
./shannon start URL=<url> REPO=<path> ROUTER=true
```

**Note:** Shannon is optimized for Anthropic's Claude models. Alternative providers may produce inconsistent results.

## Troubleshooting

### Common Issues
- **"Repository not found"**: Ensure target local directory exists and is accessible
- **Agent fails to start**: Check that `npm run build` has been run after code changes
- **Changes not picked up**: Use `REBUILD=true` to force Docker rebuild without cache

### Temporal & Docker Issues
- **"Temporal not ready"**: Wait for health check or run `docker compose logs temporal`
- **Worker not processing**: Ensure worker container is running with `docker compose ps`
- **Reset workflow state**: `./shannon stop CLEAN=true` removes all Temporal data and volumes
- **Local apps unreachable**: Use `host.docker.internal` instead of `localhost` for URLs
- **Container permissions**: On Linux, may need `sudo` for docker commands
- **View workflow history**: Temporal Web UI at http://localhost:8233

### External Tool Dependencies
Missing tools can be skipped using `PIPELINE_TESTING=true` mode during development:
- `nmap` - Network scanning
- `subfinder` - Subdomain discovery
- `whatweb` - Web technology detection

### Understanding Agent Failures

**Retryable Errors** (will retry up to 50 times in production):
- Billing errors (rate limits, quota issues)
- Transient network errors
- Temporary API unavailability

**Non-Retryable Errors** (fail immediately):
- AuthenticationError - Invalid API credentials
- ConfigurationError - Invalid YAML config
- InvalidTargetError - Target URL/repo not accessible
- ExecutionLimitError - Agent exceeded max turns (10,000)

Check agent logs in `audit-logs/{sessionId}/agents/` for detailed error context.
