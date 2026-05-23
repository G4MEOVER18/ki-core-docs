# ki-core

**Distributed Self-Hosted AI Infrastructure**

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Windows%2011-blue.svg)](docs/03-ai-core.md)
[![Agents](https://img.shields.io/badge/Agents-5-green.svg)](docs/09-agents.md)
[![GPUs](https://img.shields.io/badge/GPUs-RTX%205060Ti%20%2B%203060%20%2B%203080-76b900.svg)](docs/02-hardware.md)

---

## Overview

KI-CORE is a multi-node distributed AI infrastructure for home/office LAN environments. A Proxmox-based AI-Core server with dual GPUs runs the primary agent stack (Jarvis, Nexus, Clio), a gaming workstation hosts ClaudeBot for long-context tasks, and a CPU workstation (CyberNode) handles Claude Code sessions, git workflows, and GitHub publishing. All nodes share storage and communicate via a file-based protocol on a TrueNAS NAS.

```
+─────────────────+     +─────────────────+     +─────────────────+
│    AI-Core      │────▶│   Gaming Node   │     │   CyberNode     │
│  192.168.0.14   │     │  192.168.0.21   │     │  192.168.0.99   │
│  Proxmox VM     │     │  RTX 3080       │     │  Intel NUC      │
│  RTX 5060Ti+3060│     │  ClaudeBot      │     │  Claude Code    │
+─────────────────+     +─────────────────+     +─────────────────+
         │                       │                       │
         └───────────────────────┴───────────────────────┘
                                 │
                    +────────────▼────────────+
                    │          NAS            │
                    │      192.168.0.5        │
                    │  TrueNAS / Ki-Daten     │
                    +─────────────────────────+
```

---

## Key Features

- **Dual Ollama instances** — RTX 5060 Ti (:11434) + RTX 3060 (:11435) für parallele Inferenz
- **OpenClaw Gateway** — Multi-Agent-Routing, Session-Management, Fallback-Chains
- **5 spezialisierte Agenten** — Jarvis (Koordinator), Nexus (Code), Clio (Premium), ClaudeBot (langer Kontext), Claude Code (Shell/Git)
- **Docker-Stack** — Open-WebUI, LiteLLM, ComfyUI, SearXNG, Portainer, Watchtower
- **NAS Chat-Protokoll** — dateibasierte Inter-Node-Kommunikation über SMB-Share
- **Auto-Recovery** — Windows Task Scheduler überwacht Gateway alle 5 Minuten
- **Persistentes Memory-System** — SOUL.md, MEMORY.md, tägliche Logs, Dream-Processing

---

## Node Overview

| Node | IP | Rolle | Hardware |
|------|----|-------|----------|
| **AI-Core** | 192.168.0.14 | Primärer Inferenz-Server | Xeon E5-2699 v3, 128 GB RAM, RTX 5060 Ti + RTX 3060, Proxmox VM |
| **Gaming Node** | 192.168.0.21 | Sekundäre Inferenz, langer Kontext | RTX 3080, Windows 11 |
| **CyberNode** | 192.168.0.99 | Shell-Tasks, Git, GitHub-Publishing | Intel NUC, Windows 11 Pro |
| **NAS** | 192.168.0.5 | Shared Storage, Inter-Node-Chat | TrueNAS SCALE |

---

## Quick Start

```powershell
# 1. OpenClaw installieren (Node.js 18+ vorausgesetzt)
npm install -g openclaw

# 2. Config aus Template kopieren und anpassen
cp config/templates/openclaw-template.json C:\Users\<USER>\.openclaw\openclaw.json
# → Placeholders ersetzen: <REPLACE_WITH_STRONG_PASSWORD>, <ANTHROPIC_API_KEY>

# 3. Gateway starten
openclaw gateway start

# 4. Agent testen
openclaw agent --agent main --message "Hallo, bist du aktiv?" --json
```

---

## Documentation

| Dokument | Inhalt |
|----------|--------|
| [01 — Architektur](docs/01-architecture.md) | System-Übersicht, Netzwerk-Topologie, Datenfluss |
| [02 — Hardware](docs/02-hardware.md) | Alle Nodes mit CPU/GPU/RAM-Specs |
| [03 — AI-Core](docs/03-ai-core.md) | Proxmox VM, Ollama, Docker, Verzeichnisstruktur |
| [04 — Gaming Node](docs/04-gaming-node.md) | ClaudeBot, Ollama, Modelle |
| [05 — CyberNode](docs/05-cybernode.md) | Claude Code, NAS-Anbindung, Git-Workflows |
| [06 — Proxmox](docs/06-proxmox.md) | GPU-Passthrough, VM-Konfiguration, Backups |
| [07 — NAS](docs/07-nas.md) | TrueNAS, SMB, Chat-Protokoll, Backup-Jobs |
| [08 — Services](docs/08-services.md) | Docker-Stack, alle Container |
| [09 — Agenten](docs/09-agents.md) | Alle 5 Agenten, Modelle, Startup-Sequenz |
| [10 — Netzwerk](docs/10-networking.md) | Port-Map, Firewall-Regeln, DNS |
| [11 — Setup-Guide](docs/11-setup-guide.md) | Schritt-für-Schritt Nachbau |
| [12 — Known Issues](docs/12-known-issues.md) | Offene Probleme, Workarounds |

---

## Config Templates

Fertige Templates mit `<PLACEHOLDER>`-Syntax — nie echte Keys eintragen:

- [`config/templates/openclaw-template.json`](config/templates/openclaw-template.json)
- [`config/templates/litellm-config-template.yaml`](config/templates/litellm-config-template.yaml)
- [`config/templates/docker-compose-template.yml`](config/templates/docker-compose-template.yml)

---

## License

MIT — see [LICENSE](LICENSE)

© 2026 G4MEOVER18
