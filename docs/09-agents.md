# AI Agents

KI-CORE runs multiple AI agents, each with a distinct role, model, and personality. All agents are managed by the OpenClaw framework.

---

## Agent Overview

| Agent | Node | IP | Model | GPU | Role |
|-------|------|----|-------|-----|------|
| **Jarvis** (main) | AI-Core | 192.168.0.14 | qwen3:14b | RTX 5060 Ti | Generalist coordinator |
| **Nexus** | AI-Core | 192.168.0.14 | qwen2.5-coder:7b | RTX 3060 | Code specialist |
| **Clio** | AI-Core | 192.168.0.14 | claude-opus-4-7 | Cloud | Premium reasoning |
| **ClaudeBot** | Gaming Node | 192.168.0.21 | claude-sonnet-4-6 | Cloud | High-context tasks |
| **Claude Code** | CyberNode | 192.168.0.99 | claude-sonnet-4-6 | Cloud | Shell/file operations |

---

## Jarvis — Main Agent

**Workspace:** `D:\KI-CORE\apps\openclaw\workspace`  
**Model:** `ollama/jarvis` (qwen3:14b fine-tuned, port 11434)  
**GPU:** RTX 5060 Ti  

Jarvis is the system's generalist coordinator and primary interface. He handles:
- Direct conversations with the user
- Coordinating other agents via `openclaw agent --agent <name>`
- Long-horizon planning and task tracking
- Memory management (daily logs + MEMORY.md + Open-WebUI memories)
- NAS chat for cross-node communication

**Startup sequence:**
1. Read `SOUL.md` (identity)
2. Read `USER.md` (user profile)
3. Read today's + yesterday's `memory/YYYY-MM-DD.md`
4. Read `MEMORY.md` (main sessions only)
5. Check `.openclaw/task-state.json` for interrupted tasks
6. Query Open-WebUI memories for session-relevant context
7. Heartbeat check (`HEARTBEAT.md`)

**Fallback chain:** `qwen3:14b` → `gemma3:12b` → `qwen3:8b` → `minimax-m2.5:cloud` → `claude-sonnet-4-6`

---

## Nexus — Code Specialist

**Workspace:** `D:\KI-CORE\apps\openclaw\workspace-nexus`  
**Model:** `ollama-3060/nexus` (qwen2.5-coder:7b on RTX 3060, port 11435)  
**GPU:** RTX 3060  

Nexus is purpose-built for code tasks on the dedicated RTX 3060. He specializes in:
- Code generation and debugging
- Technical analysis
- Script writing (PowerShell, Python, JS)

**Key config:**
```json
{
  "id": "nexus",
  "workspace": "D:\\KI-CORE\\apps\\openclaw\\workspace-nexus",
  "model": {
    "primary": "ollama-3060/nexus",
    "fallbacks": ["ollama/qwen2.5-coder:7b"]
  }
}
```

**Invoke:**
```powershell
openclaw agent --agent nexus --message "Write a Python function that..." --json
```

**Known issue:** Gateway routing to `ollama-3060` provider (port 11435) is currently unreliable — falls back to port 11434 (5060 Ti). See [known issues](12-known-issues.md).

---

## Clio — Premium Reasoning

**Workspace:** `D:\KI-CORE\apps\openclaw\workspace-clio`  
**Model:** `claude-cli/claude-opus-4-7` (Anthropic cloud)  
**Backend:** Claude CLI (`claude.exe`)

Clio handles tasks requiring top-tier reasoning quality:
- Complex analysis and planning
- Tasks exceeding Jarvis's capability
- Large codebase reviews

**Invoke from Jarvis:**
```powershell
openclaw agent --agent clio --message "Analyze this architecture..." --json
```

---

## ClaudeBot — Gaming Node

**Node:** Gaming Node (192.168.0.21)  
**Model:** claude-sonnet-4-6 (200k context)

Handles tasks that need a very large context window. Communicates with AI-Core via NAS chat protocol.

---

## Claude Code — CyberNode

**Node:** CyberNode (192.168.0.99)  
**Model:** claude-sonnet-4-6

Runs as the Claude Code CLI tool. Primary capabilities:
- File system operations
- Git workflows and GitHub publishing
- Shell script execution
- Direct tool use (no gateway overhead)

---

## OpenClaw Framework

All agents (except Claude Code) run on the **OpenClaw** framework:

- **Gateway:** Port 18789, provides multi-agent routing and session management
- **Config:** `C:\Users\Yanis\.openclaw\openclaw.json`
- **Session storage:** `C:\Users\Yanis\.openclaw\agents\<name>\sessions\`
- **Compaction:** `reserveTokensFloor: 1000` — minimal context reservation to prevent premature compaction
- **Timeout:** 600 seconds per session

### Session Lifecycle

```
1. Request arrives at Gateway (:18789)
2. Gateway loads/creates session (UUID-based .jsonl file)
3. Bootstrap: load workspace docs + skills into context
4. LLM call to Ollama or cloud
5. Response streamed back
6. Session state persisted to .jsonl
```

### Memory System

Each agent maintains persistent memory across sessions:

```
workspace/
├── SOUL.md          # Fixed identity, never changes
├── IDENTITY.md      # Current self-definition
├── USER.md          # Who the user is
├── MEMORY.md        # Curated long-term memory (main sessions only)
├── AGENTS.md        # Behavioral rules, red lines
├── TOOLS.md         # Tool references and configs
├── HEARTBEAT.md     # Periodic check tasks
└── memory/
    ├── YYYY-MM-DD.md        # Daily raw logs
    └── dreaming/
        ├── light/YYYY-MM-DD.md   # Light sleep processing
        ├── rem/YYYY-MM-DD.md     # REM processing
        └── deep/YYYY-MM-DD.md   # Deep dream synthesis
```

Open-WebUI memories provide cross-session retrieval via `POST /api/v1/memories/query`.

---

## Calling Agents from Code

```powershell
# Call a specific agent via OpenClaw CLI
openclaw agent --agent nexus --message "Review this code: ..." --json

# Via gateway HTTP API (for programmatic use)
$body = @{
    agentId = "nexus"
    message = "Your prompt here"
} | ConvertTo-Json
Invoke-RestMethod -Uri "http://127.0.0.1:18789/api/agent/chat" -Method POST -Body $body -ContentType "application/json"

# Direct Ollama (bypass gateway — fastest, no session context)
$body = @{
    model = "qwen3:14b"
    prompt = "Your prompt here"
    stream = $false
} | ConvertTo-Json
Invoke-RestMethod -Uri "http://127.0.0.1:11434/api/generate" -Method POST -Body $body -ContentType "application/json"
```
