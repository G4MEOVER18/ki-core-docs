# AI-Core Node

The AI-Core is the central AI inference server — the computational heart of the KI-CORE system.

---

## Overview

| Property | Value |
|----------|-------|
| **Hostname** | AI-CORE |
| **IP Address** | 192.168.0.14 |
| **OS** | Windows 11 Enterprise |
| **Platform** | Proxmox VM (KVM/QEMU) |
| **CPU** | Intel Xeon E5-2699 v3, 18 cores / 36 threads @ 2.30 GHz |
| **RAM** | 128 GB |
| **GPU 1** | NVIDIA RTX 5060 Ti (16 GB VRAM) |
| **GPU 2** | NVIDIA RTX 3060 (12 GB VRAM) |
| **Storage** | C: (OS), D: (KI-CORE workloads), K: (NAS mount) |

---

## GPU Configuration

Both GPUs are PCIe passthrough from the Proxmox host into the Windows VM.

| GPU | VRAM | Ollama Port | Agent | Primary Model |
|-----|------|------------|-------|---------------|
| RTX 5060 Ti | 16 GB | 11434 | Jarvis (main) | qwen3:14b (8.6 GB) |
| RTX 3060 | 12 GB | 11435 | Nexus (coder) | qwen2.5-coder:7b (4.4 GB) |

**Dual Ollama instances** run simultaneously, each bound to one GPU:

```powershell
# Instance 1 — RTX 5060 Ti (GPU index 0 or device 0)
$env:CUDA_VISIBLE_DEVICES = "0"
ollama serve  # listens on :11434

# Instance 2 — RTX 3060 (GPU index 1 or device 1)
$env:CUDA_VISIBLE_DEVICES = "1"
$env:OLLAMA_HOST = "127.0.0.1:11435"
ollama serve  # listens on :11435
```

Both instances share the same model storage directory, so models are downloaded once and usable by either GPU.

---

## Available Models

Both Ollama instances have access to all models (shared storage):

| Model | Size | Context | Best For |
|-------|------|---------|----------|
| `jarvis:latest` | 8.6 GB | 65k | Generalist tasks (custom Jarvis persona) |
| `nexus:latest` | 4.4 GB | 32k | Coding (custom Nexus persona) |
| `qwen3:14b` | 8.6 GB | 65k | Reasoning, complex tasks |
| `qwen3:8b` | 4.9 GB | 32k | Balanced speed/quality |
| `qwen3:4b` | 2.3 GB | 32k | Fast, lightweight tasks |
| `qwen2.5-coder:14b` | 8.4 GB | 65k | Advanced code generation |
| `qwen2.5-coder:7b` | 4.4 GB | 32k | Code tasks |
| `gemma3:12b` | 7.6 GB | 32k | Alternative generalist |
| `gemma3:27b` | 16.2 GB | 32k | Large reasoning (5060 Ti only) |
| `deepseek-r1:8b` | 4.9 GB | 32k | Chain-of-thought reasoning |
| `qwen3-vl:8b` | 5.7 GB | 32k | Vision + text |
| `nomic-embed-text` | 0.3 GB | 8k | Text embeddings |

---

## Running Services

### Ollama

Two independent instances, each serving one GPU:

```
http://127.0.0.1:11434  →  RTX 5060 Ti (Jarvis)
http://127.0.0.1:11435  →  RTX 3060   (Nexus)
```

### OpenClaw Gateway

Multi-agent routing layer sitting in front of Ollama and cloud providers:

```
http://0.0.0.0:18789  →  All agents (Jarvis, Nexus, Clio)
```

Config: `C:\Users\Yanis\.openclaw\openclaw.json`

### Docker Services

| Service | Port | Purpose |
|---------|------|---------|
| open-webui | 3000 | Web chat UI for all models |
| litellm | 4000 | Unified API proxy (OpenAI-compatible) |
| comfyui | 8188 | Stable Diffusion image generation |
| searxng | 8888 | Self-hosted web search |
| portainer | 9443 | Docker management |
| postgres | 5432 | Database (local only) |
| redis | 6379 | Cache (local only) |
| watchtower | — | Auto-updates containers |

---

## Directory Structure

```
D:\KI-CORE\
├── apps\
│   ├── claude\workspace\       ← Claude Code workspace
│   ├── openclaw\
│   │   ├── workspace\          ← Jarvis workspace
│   │   ├── workspace-nexus\    ← Nexus workspace
│   │   └── workspace-clio\     ← Clio workspace
│   ├── codex\workspace\
│   ├── opencode\workspace\
│   └── droid\workspace\
├── projects\                   ← Project files
│   ├── coding\
│   ├── automation\
│   ├── research\
│   └── testing\
├── models\                     ← AI model cache
├── services\                   ← Service configs
├── cache\                      ← Package manager caches
├── scripts\                    ← System PowerShell scripts
├── launchers\                  ← .cmd shortcuts
└── docker\                     ← Docker configs
```

### Environment Variables

```
KI_CORE_HOME=D:\KI-CORE
NPM_CONFIG_CACHE=D:\KI-CORE\cache\npm
PIP_CACHE_DIR=D:\KI-CORE\cache\pip
HF_HOME=D:\KI-CORE\cache\huggingface
```

---

## Auto-Recovery

Windows Task Scheduler runs two scheduled tasks:

| Task | RunLevel | Schedule | Purpose |
|------|----------|----------|---------|
| `\OpenClaw Gateway` | Limited | On system startup | Start gateway process |
| `\OpenClaw Auto-Recovery` | Highest | Every 5 minutes | Monitor + restart gateway, cleanup sessions, warm VRAM |

The Auto-Recovery task also serves as an **elevated command runner**: Claude Code can write a PowerShell script to `elevated-runner-input.ps1` and the task executes it with admin privileges, writing output to `elevated-runner-output.txt`.
