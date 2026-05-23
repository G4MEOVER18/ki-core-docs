# CyberNode

Lokale Workstation für Claude Code Sessions, Git-Workflows und GitHub-Publishing.

---

## Übersicht

| Eigenschaft | Wert |
|-------------|------|
| **Hostname** | CYBER-NODE-NUC |
| **IP-Adresse** | 192.168.0.99 (zweite NIC: 192.168.0.73) |
| **OS** | Windows 11 Pro |
| **Formfaktor** | Intel NUC |
| **GPU** | Keine (CPU-basiert) |
| **Primär-Agent** | Claude Code (claude-sonnet-4-6) |
| **NAS-Mount** | K:\ (`\192.168.0.5\Ki-Daten`) ✅ aktiv |
| **Rolle** | Dateioperationen, Git, Shell-Skripte, GitHub-Publishing |

---

## Agent: Claude Code

Claude Code läuft als CLI-Tool auf CyberNode mit vollständigem Dateisystem-Zugriff.

- **Modell:** claude-sonnet-4-6 (via Claude.ai OAuth — kein API-Key nötig)
- **Workspace:** Lokales Dateisystem
- **Kernfähigkeiten:**
  - Dateien direkt lesen/schreiben
  - Shell-Befehle ausführen (PowerShell/Bash)
  - Git-Operationen und GitHub-Publishing via `gh` CLI
  - Hintergrundaufgaben starten und überwachen
  - NAS Chat-Protokoll lesen/schreiben

### CLAUDE.md

Die Claude Code Instanz liest `CLAUDE.md` im Workspace-Root. Wichtige Regeln:
- Immer auf Deutsch antworten
- Projektdateien unter `D:\Projekte\<kategorie>\<projektname>`
- Proaktiv handeln — nicht fragen ob etwas getan werden soll

---

## NAS-Anbindung

### Status: Aktiv ✅

K: Drive ist persistent gemountet und wird aktiv genutzt:

```
\192.168.0.5\Ki-Daten  →  K:\
```

- **Schreiben:** `K:\KI-CORE\chat\from-cybernode\`
- **Lesen:** `K:\KI-CORE\chat\from-ai-core\`, `K:\KI-CORE\chat\from-gamingnode\`

NAS-Chat ist vollständig eingerichtet und operativ.

### Direktzugriff auf AI-Core Services

Claude Code auf CyberNode kann AI-Core Services direkt erreichen:

```
OpenClaw Gateway: http://192.168.0.14:18789
LiteLLM API:      http://192.168.0.14:4000
Open-WebUI:       http://192.168.0.14:3000
Ollama (5060Ti):  http://192.168.0.14:11434
Ollama (3060):    http://192.168.0.14:11435
```

---

## Rolle: GitHub-Publishing Node

CyberNode ist der **git/GitHub Publishing Node** des Netzwerks:
- Repos von AI-Core via NAS empfangen
- Inhalte prüfen, Secrets entfernen, Lücken füllen
- Via `gh` CLI nach GitHub pushen

### Workflow

1. AI-Core schreibt NAS-Nachricht mit Repo-Pfad nach `from-ai-core\`
2. CyberNode liest die Nachricht, prüft Inhalt
3. CyberNode korrigiert, ergänzt und pusht via `gh repo create` + `git push`
4. Nachricht auf `STATUS: DONE` setzen

---

## Umgebungsvariablen

| Variable | Wert |
|----------|------|
| `CYBER_NODE_HOME` | `D:\KI-CORE` |
| `NPM_CONFIG_CACHE` | `D:\KI-CORE\cache\npm` |
| `PIP_CACHE_DIR` | `D:\KI-CORE\cache\pip` |

---

## Installierte Tools

| Tool | Zweck |
|------|-------|
| Claude Code (claude-cli) | Primärer AI-Agent |
| Git + GitHub CLI (`gh`) | Versionskontrolle + GitHub-Publishing |
| PlatformIO | Embedded-Firmware-Builds (STM32, ESP32) |
| Python 3.x | Security-Tool-Entwicklung |
| Node.js | OpenClaw-kompatible Tools |
| PuTTY / plink | SSH zu AI-Core und KI-Core Nodes |
