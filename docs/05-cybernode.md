# CyberNode

Local coding and shell workstation, used primarily for Claude Code sessions.

---

## Overview

| Property | Value |
|----------|-------|
| **IP Address** | 192.168.0.63 |
| **GPU** | None (CPU-based inference only) |
| **Primary Agent** | Claude Code (claude-sonnet-4-6) |
| **Role** | File operations, git workflows, shell scripts, local builds |

---

## Agent: Claude Code

Claude Code runs as a CLI tool on CyberNode with full filesystem access.

- **Model:** claude-sonnet-4-6 (via Claude.ai OAuth subscription — no API key needed)
- **Workspace:** Local filesystem
- **Key capabilities:**
  - Read/write files directly
  - Run shell commands (PowerShell/Bash)
  - Git operations
  - Spawn background tasks and monitor output
  - Elevated runner pattern (via NAS + shared task system)

### CLAUDE.md

The CyberNode's Claude Code instance reads from `CLAUDE.md` in the workspace root. Key rules:
- Always respond in German
- Project files go under `D:\KI-CORE\projects\<kategorie>\<projektname>`
- Prefer Ollama (local models) for heavy work, Claude API for tool-use tasks
- All AI agent API traffic goes through AI-Core's LiteLLM/OpenClaw gateway

---

## Connectivity

### NAS Integration (Pending)

CyberNode has not yet been connected to the TrueNAS share. **TODO:**

```powershell
# Mount NAS share persistently
net use K: \\192.168.0.5\Ki-Daten /persistent:yes

# Verify chat channel
Test-Path "K:\KI-CORE\chat\from-cybernode\"
```

Once mounted:
- **Write to:** `K:\KI-CORE\chat\from-cybernode\`
- **Read from:** `K:\KI-CORE\chat\from-ai-core\`, `K:\KI-CORE\chat\from-gaming-node\`

### Direct Access to AI-Core

Claude Code on CyberNode can reach AI-Core services directly:

```
OpenClaw Gateway: http://192.168.0.14:18789
LiteLLM API:      http://192.168.0.14:4000
Open-WebUI:       http://192.168.0.14:3000
```

---

## Repo Publishing

CyberNode is designated as the **git/GitHub publishing node** — it handles:
- Reviewing documentation repos pushed from AI-Core
- Running `git push` to GitHub
- Code review before publishing

### Workflow for Receiving Repos from AI-Core

1. AI-Core writes a NAS chat message with the repo path
2. CyberNode Claude Code reads the message from `K:\KI-CORE\chat\from-ai-core\`
3. CyberNode reviews, adjusts, then pushes to GitHub via `gh` CLI

---

## Environment

| Variable | Value |
|----------|-------|
| `KI_CORE_HOME` | `D:\KI-CORE` |
| `NPM_CONFIG_CACHE` | `D:\KI-CORE\cache\npm` |
| `PIP_CACHE_DIR` | `D:\KI-CORE\cache\pip` |
| `HF_HOME` | `D:\KI-CORE\cache\huggingface` |
