# System Architecture

## Overview

KI-CORE is a distributed, self-hosted AI infrastructure spanning multiple physical nodes connected via a local network. The system is designed for 24/7 autonomous operation with automatic failure recovery, multi-agent coordination, and efficient GPU utilization across heterogeneous hardware.

---

## Network Topology

```
                        ┌─────────────────────────────────────────┐
                        │         Local Network: 192.168.0.0/24   │
                        └─────────────────────────────────────────┘
                                           │
          ┌────────────────────┬───────────┴──────────┬──────────────────┐
          │                    │                       │                  │
    ┌─────▼──────┐    ┌────────▼───────┐    ┌─────────▼─────┐  ┌────────▼───────┐
    │  AI-Core   │    │  Gaming Node   │    │   CyberNode   │  │      NAS       │
    │ .0.14      │    │    .0.21       │    │    .0.63      │  │    .0.5        │
    │ Proxmox VM │    │ Workstation    │    │  Workstation  │  │  TrueNAS       │
    └────────────┘    └────────────────┘    └───────────────┘  └────────────────┘
```

### Node Roles

| Node | IP | Role | Hardware |
|------|----|------|----------|
| **AI-Core** | 192.168.0.14 | Primary AI inference server | Xeon E5-2699 v3, 128GB RAM, RTX 5060 Ti + RTX 3060 |
| **Gaming Node** | 192.168.0.21 | Secondary AI node | RTX 3080 |
| **CyberNode** | 192.168.0.63 | Coding assistant, shell tasks | CPU-based workstation |
| **NAS** | 192.168.0.5 | Shared storage, inter-node chat | TrueNAS |
| **Proxmox Host** | — | Hypervisor for AI-Core VM | Bare metal server |

---

## AI-Core Architecture

```
┌─────────────────────────────── AI-Core (192.168.0.14) ──────────────────────────────┐
│                                                                                      │
│  ┌────────────────────────────────── Docker Stack ────────────────────────────────┐  │
│  │  open-webui :3000   litellm :4000   comfyui :8188   searxng :8888             │  │
│  │  portainer :9443    postgres :5432  redis :6379      watchtower               │  │
│  └────────────────────────────────────────────────────────────────────────────────┘  │
│                                                                                      │
│  ┌──────────────────────────── OpenClaw Gateway :18789 ────────────────────────────┐ │
│  │                                                                                  │ │
│  │   Agent: main/Jarvis        Agent: nexus/Nexus        Agent: clio/Clio         │ │
│  │   qwen3:14b (5060Ti)        qwen2.5-coder:7b (3060)   claude-opus-4-7 (cloud)  │ │
│  └─────────────────┬───────────────────┬────────────────────────────────────────┘  │
│                    │                   │                                             │
│  ┌─────────────────▼──┐   ┌────────────▼──────────┐                                │
│  │  Ollama :11434     │   │  Ollama :11435          │                                │
│  │  RTX 5060 Ti       │   │  RTX 3060               │                                │
│  │  - jarvis:latest   │   │  - nexus:latest         │                                │
│  │  - qwen3:14b       │   │  - qwen2.5-coder:7b     │                                │
│  │  - qwen3:8b        │   │                         │                                │
│  │  - gemma3:12b      │   │                         │                                │
│  └────────────────────┘   └─────────────────────────┘                                │
│                                                                                      │
└──────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Agent Hierarchy

```
                        ┌─────────────────────────────┐
                        │     Claude Code (Human UI)  │
                        │     CyberNode 192.168.0.63  │
                        └──────────────┬──────────────┘
                                       │ (direct session)
                        ┌──────────────▼──────────────┐
                        │   OpenClaw Gateway :18789   │
                        │      on AI-Core             │
                        └────────┬──────┬─────────────┘
                                 │      │
                    ┌────────────▼──┐ ┌─▼──────────────┐
                    │ Jarvis (main) │ │ Nexus (coder)  │
                    │ qwen3:14b     │ │ qwen2.5-coder  │
                    │ :11434        │ │ :11435          │
                    └───────┬───────┘ └────────────────┘
                            │ (spawn subagent)
                    ┌───────▼────────┐
                    │ Clio (premium) │
                    │ claude-opus-4-7│
                    │ (cloud)        │
                    └────────────────┘
```

---

## Data Flow

### Inference Request Flow

```
User → OpenClaw Gateway → Agent Session (bootstrap) → Ollama/Cloud → Response
                              ↕
                    Session State (C:\Users\..\.openclaw\agents\<name>\sessions\)
                              ↕
                    Workspace Files (AGENTS.md, MEMORY.md, SOUL.md, ...)
```

### Inter-Node Communication

```
AI-Core (Jarvis) → writes MSG to K:\KI-CORE\chat\from-ai-core\
CyberNode (Claude Code) → reads MSG, writes REPLY to K:\KI-CORE\chat\from-cybernode\
Gaming Node (ClaudeBot) → reads/writes K:\KI-CORE\chat\from-gaming-node\
```

---

## Model Routing

The OpenClaw gateway routes requests based on `openclaw.json` provider configuration:

```
Request: "ollama-3060/nexus"
  → Provider lookup: ollama-3060 → baseUrl: http://127.0.0.1:11435
  → POST /api/chat with model: nexus
  → RTX 3060 processes inference

Request: "ollama/jarvis"  
  → Provider lookup: ollama → baseUrl: http://127.0.0.1:11434
  → POST /api/chat with model: jarvis
  → RTX 5060 Ti processes inference
```

Fallback chain triggers automatically on timeout or model unavailability.

---

## Reliability

- **Auto-Recovery:** Windows Task Scheduler runs every 5 minutes, detects gateway crashes, restarts via `\OpenClaw Gateway` task
- **Session Cleanup:** Removes stale lock files (>10min), old sessions (>24h), junk files
- **VRAM Warmup:** Keeps `nexus:latest` loaded in RTX 3060 VRAM with `keep_alive=-1`
- **Elevated Commands:** Separate Task Scheduler task (RunLevel=Highest) allows non-admin processes to execute privileged operations
