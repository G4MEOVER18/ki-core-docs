# Hardware Overview

## Node Comparison

| Node | CPU | RAM | GPU(s) | Role |
|------|-----|-----|--------|------|
| **AI-Core** | Xeon E5-2699 v3 (18c/36t) | 128 GB | RTX 5060 Ti (16GB) + RTX 3060 (12GB) | Primary AI server |
| **Gaming Node** | — | — | RTX 3080 (10GB) | Secondary inference |
| **CyberNode** | — | — | None (CPU) | Shell/coding tasks |
| **Proxmox Host** | Bare metal (same CPU as VM) | — | Hosts AI-Core VM | Hypervisor |
| **NAS** | — | — | — | Shared storage |

---

## AI-Core (Proxmox VM)

The AI-Core runs as a KVM/QEMU virtual machine with full hardware passthrough for the GPUs.

### CPU
- **Model:** Intel Xeon E5-2699 v3
- **Architecture:** Haswell-EP (2014)
- **Cores/Threads:** 18 cores / 36 threads (full socket passed through)
- **Base clock:** 2.30 GHz, Turbo up to 3.60 GHz
- **Cache:** 45 MB L3
- **ISA extensions:** AVX2, SSE4.2, AES-NI — all passed through via `cpu: host` in Proxmox

### RAM
- **Total:** 128 GB
- **Type:** DDR4 ECC (Xeon platform)
- **Allocation:** Full 128 GB assigned to VM (no ballooning — required for GPU passthrough)

### GPUs

#### RTX 5060 Ti — Primary Inference (Jarvis)
| Property | Value |
|----------|-------|
| VRAM | 16 GB GDDR7 |
| CUDA cores | ~4608 |
| Architecture | Blackwell |
| Ollama port | 11434 |
| Agent | Jarvis (qwen3:14b) |
| Model capacity | Up to ~16 GB (qwen3:14b = 8.6 GB + overhead) |

#### RTX 3060 — Secondary Inference (Nexus)
| Property | Value |
|----------|-------|
| VRAM | 12 GB GDDR6 |
| CUDA cores | 3584 |
| Architecture | Ampere |
| Ollama port | 11435 |
| Agent | Nexus (qwen2.5-coder:7b) |
| Model capacity | Up to ~12 GB |

### VRAM Budget

```
RTX 5060 Ti (16 GB):
  jarvis:latest  = 8.6 GB  (loaded continuously)
  OS overhead    = ~0.5 GB
  Available      = ~6.9 GB  (for additional models / context)

RTX 3060 (12 GB):
  nexus:latest   = 4.4 GB  (keep_alive=-1, permanent)
  OS overhead    = ~0.5 GB
  Available      = ~7.1 GB  (context window, additional models)
```

---

## Gaming Node (192.168.0.21)

Secondary AI node, primarily used for ClaudeBot (high-context reasoning).

| Property | Value |
|----------|-------|
| GPU | NVIDIA RTX 3080 (10 GB GDDR6X) |
| Ollama | Port 11434 (`http://192.168.0.21:11434`) |
| Agent | ClaudeBot (claude-sonnet-4-6) |
| Models | qwen2.5:7b, qwen2.5-coder:7b, qwen2.5-coder:14b, gpt-oss:20b, llava:7b |

---

## CyberNode (192.168.0.99)

Lightweight workstation for local shell tasks and Claude Code.

| Property | Value |
|----------|-------|
| GPU | None |
| Agent | Claude Code (claude-sonnet-4-6) |
| Role | File operations, git, shell scripts |
| Connectivity | LAN + NAS (pending) |

---

## NAS (192.168.0.5)

| Property | Value |
|----------|-------|
| OS | TrueNAS SCALE |
| Share | `Ki-Daten` (SMB + NFS) |
| Role | Shared storage, inter-node communication, backups |
| ZFS | Dataset snapshots, monthly scrub |

---

## Upgrade Path

| Component | Current | Next upgrade | Reason |
|-----------|---------|-------------|--------|
| AI-Core VRAM | 28 GB total (16+12) | RTX 5090 (32GB) | Fit 32B+ models |
| AI-Core CPU | Xeon E5-2699 v3 | Modern EPYC/Xeon | PCIe gen 3 bottleneck |
| Gaming Node GPU | RTX 3080 (10GB) | RTX 5060 Ti (16GB) | Larger local models |
| NAS storage | — | Additional HDD array | Long-term model/data storage |
