# Networking

---

## Network Topology

```
Internet
    │
    └── Router / Firewall
            │
            └── 192.168.0.0/24 (LAN)
                    │
        ┌───────────┼────────────┬───────────────────────────┐
        │           │            │                           │
   AI-Core      NAS Server  Gaming Node                 CyberNode
  .0.14          .0.5          .0.21                     .0.63
```

---

## Node Addresses

| Node | IP | Hostname | Notes |
|------|----|----------|-------|
| AI-Core | 192.168.0.14 | AI-CORE | Static/DHCP reservation |
| NAS | 192.168.0.5 | — | TrueNAS, static |
| Gaming Node | 192.168.0.21 | — | Static/DHCP reservation |
| CyberNode | 192.168.0.63 | — | Static/DHCP reservation |
| Proxmox Host | — | — | Internal, manages AI-Core VM |

---

## AI-Core Port Map

| Port | Protocol | Service | Access |
|------|----------|---------|--------|
| 3000 | TCP/HTTP | Open-WebUI | LAN |
| 4000 | TCP/HTTP | LiteLLM API | LAN |
| 5432 | TCP | PostgreSQL | localhost only |
| 6379 | TCP | Redis | localhost only |
| 8000 | TCP/HTTP | Portainer (HTTP) | LAN |
| 8188 | TCP/HTTP | ComfyUI | LAN |
| 8888 | TCP/HTTP | SearXNG | LAN |
| 9443 | TCP/HTTPS | Portainer (HTTPS) | LAN |
| 11434 | TCP/HTTP | Ollama (RTX 5060 Ti) | localhost |
| 11435 | TCP/HTTP | Ollama (RTX 3060) | localhost |
| 18789 | TCP/HTTP | OpenClaw Gateway | LAN |

---

## NAS Share Access

The NAS `Ki-Daten` share is accessible via SMB from all Windows nodes:

```
\\192.168.0.5\Ki-Daten
```

Mount as persistent drive:

```powershell
net use K: \\192.168.0.5\Ki-Daten /persistent:yes
```

---

## Security Notes

### What MUST NOT be internet-exposed

- Ollama ports (11434, 11435) — no authentication, full model access
- PostgreSQL (5432) — database, localhost only
- Redis (6379) — no auth by default, localhost only
- OpenClaw Gateway (18789) — password-protected, but only tested on LAN

### What CAN be internet-exposed (with care)

- Open-WebUI (3000) — has authentication
- LiteLLM (4000) — has master key auth
- Portainer (9443) — HTTPS, has admin auth

### Recommended Firewall Rules

```
# Allow from LAN only (block from WAN):
ALLOW 192.168.0.0/24 → AI-Core:3000   (WebUI)
ALLOW 192.168.0.0/24 → AI-Core:4000   (LiteLLM)
ALLOW 192.168.0.0/24 → AI-Core:8188   (ComfyUI)
ALLOW 192.168.0.0/24 → AI-Core:18789  (OpenClaw)

# Block all external access to:
BLOCK * → AI-Core:11434-11435  (Ollama — no auth)
BLOCK * → AI-Core:5432         (Postgres)
BLOCK * → AI-Core:6379         (Redis)
```

---

## Inter-Node Communication

### NAS Chat Protocol

File-based messaging system using the NAS share. See [NAS documentation](07-nas.md).

### Direct API Calls

Agents can call services on other nodes directly:

```powershell
# From CyberNode → AI-Core Ollama
Invoke-RestMethod "http://192.168.0.14:11434/api/generate" -Method POST ...

# From AI-Core → Gaming Node Ollama
Invoke-RestMethod "http://192.168.0.21:11434/api/chat" -Method POST ...
```

### OpenClaw Cross-Node Routing

The Gaming Node's Ollama is registered as `gaming-ollama` provider in `openclaw.json`:

```json
"gaming-ollama": {
  "baseUrl": "http://192.168.0.21:11434",
  "api": "ollama"
}
```

---

## DNS / Hostname Resolution

Nodes are referenced by IP in all configs. Optional: set up local DNS or `/etc/hosts` entries:

```
# /etc/hosts or Windows hosts file additions
192.168.0.14  ai-core ai-core.local
192.168.0.5   nas nas.local
192.168.0.21  gaming-node
192.168.0.63  cybernode
```
