# NAS — TrueNAS Storage

The KI-CORE system uses a TrueNAS server as shared storage for all nodes.

---

## Overview

| Property | Value |
|----------|-------|
| **IP Address** | 192.168.0.5 |
| **OS** | TrueNAS SCALE (or CORE) |
| **Primary Share** | `Ki-Daten` |
| **Protocols** | SMB (Windows) / NFS (Linux) |
| **Mount Point (AI-Core)** | `K:\` drive (`\\192.168.0.5\Ki-Daten`) |
| **Mount Point (other nodes)** | `/mnt/ki-daten` (NFS) or `K:\` (SMB) |

---

## Share Structure

```
\\192.168.0.5\Ki-Daten\
└── KI-CORE\
    ├── chat\                     # Inter-node communication channel
    │   ├── from-ai-core\         # Jarvis writes here
    │   ├── from-gaming-node\     # ClaudeBot writes here
    │   ├── from-cybernode\       # Claude Code writes here
    │   └── from-nas\             # System notifications
    ├── backups\                  # Database and config backups
    ├── models\                   # Shared model cache (optional)
    ├── data\                     # Shared datasets and exports
    └── sync\                     # Cross-node file sync
```

---

## NAS Chat Protocol

All AI nodes communicate via a simple file-based protocol on the NAS share.

### Message Format

```
From: Jarvis
To: ClaudeBot
Date: 2026-05-22 14:30:00
Subject: Task delegation

STATUS: WAITING

Bitte analysiere folgenden Code und gib Feedback: [...]
```

### Status Tags

| Tag | Meaning |
|-----|---------|
| `STATUS: WAITING` | Unread, pending processing |
| `STATUS: IN_PROGRESS` | Currently being processed |
| `STATUS: DONE` | Processed, can be archived |

### File Naming

```
YYYY-MM-DD_HHMMSS_from-<node>_<subject>.txt
```

---

## Windows SMB Mount (AI-Core / CyberNode)

```powershell
# Persistent drive mapping (run once, persists after reboot)
net use K: \\192.168.0.5\Ki-Daten /persistent:yes

# Or via PowerShell
New-PSDrive -Name "K" -PSProvider FileSystem -Root "\\192.168.0.5\Ki-Daten" -Persist
```

## Linux NFS Mount (if applicable)

```bash
# /etc/fstab entry
192.168.0.5:/mnt/pool/Ki-Daten  /mnt/ki-daten  nfs  defaults,nofail  0  0

# Mount immediately
mount -a
```

---

## Backups to NAS

Key services back up to NAS automatically:

| Service | Backup Target | Schedule |
|---------|--------------|----------|
| PostgreSQL | `K:\KI-CORE\backups\postgres\` | Daily |
| OpenWebUI data | `K:\KI-CORE\backups\webui\` | Weekly |
| OpenClaw sessions | `K:\KI-CORE\backups\openclaw\` | Daily |
| System config | `K:\KI-CORE\backups\config\` | On change |

---

## TrueNAS Configuration Notes

- **Dataset quotas:** Set per-service quotas to prevent runaway log growth
- **Snapshots:** Enable ZFS snapshots on `Ki-Daten` dataset (daily, retain 30 days)
- **Scrub:** Enable monthly ZFS scrub for data integrity
- **SMB settings:** Enable `SMB2`, disable `SMB1`; set case sensitivity to `insensitive` for Windows compatibility
- **NFS settings:** If using NFS, restrict exports to `192.168.0.0/24` only

---

## Sync Status

| Node | NAS Chat Access | Backup Configured |
|------|----------------|-------------------|
| AI-Core | ✅ K: drive | ✅ |
| Gaming Node | ✅ | ⚠️ Partial |
| CyberNode | ❌ Not yet mounted | ❌ Pending |
