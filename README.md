```markdown
# ki-core  
**Distributed Self-Hosted AI Infrastructure for Home/Office LANs**

[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)  
[![Platform](https://img.shields.io/badge/Platform-Linux-orange.svg)](https://github.com/yourusername/ki-core)  
[![Agents](https://img.shields.io/badge/Agents-5-green.svg)](https://github.com/yourusername/ki-core)  
[![GPUs](https://img.shields.io/badge/GPUs-2%20RTX-blue.svg)](https://github.com/yourusername/ki-core)

---

### Overview  
ki-core is a multi-node distributed AI infrastructure designed for home/office LAN environments. It leverages a Proxmox-based AI-Core, GPU-accelerated gaming nodes, and CPU-focused workstations to run five specialized AI agents (Jarvis, Nexus, Clio, ClaudeBot, Claude Code) alongside a TrueNAS storage backbone. The system integrates Docker services, dual Ollama instances, and an OpenClaw gateway for seamless agent routing and resource management.

---

### Architecture Diagram  
```
+----------------+       +----------------+       +----------------+
|   AI-Core      |<----->|  Gaming Node   |<----->|   CyberNode    |
| (Proxmox VM)   |       | (RTX 3080)     |       | (CPU-only)     |
+----------------+       +----------------+       +----------------+
        |                           |                           |
        v                           v                           v
+----------------+       +----------------+       +----------------+
|     NAS        |<----->|  Docker Services |<----->|  OpenClaw Gateway |
| (TrueNAS)      |       | (Open-WebUI, etc) |       | (Agent Routing)  |
+----------------+       +----------------+       +----------------+
```

---

### Key Features  
- **Auto-recovery** with 5-minute watchdog for node stability  
- **Session management** and persistent memory system across agents  
- **VRAM warmup** optimization for GPU workloads  
- **NAS chat protocol** for inter-node communication and storage  
- **Elevated runner** for secure admin task execution  
- Dual **Ollama instances** (RTX 5060 Ti + RTX 3060) with GPU passthrough  
- **OpenClaw gateway** for dynamic agent routing and load balancing  

---

### Node Overview  
| Node Name   | Role                  | Hardware                                      | Notes                                  |
|-------------|-----------------------|-----------------------------------------------|----------------------------------------|
| AI-Core     | Main Server           | Intel Xeon E5-2699 v3, 128GB RAM, RTX 5060 Ti + RTX 3060 | Proxmox VM with GPU passthrough        |
| Gaming Node | Secondary Inference   | RTX 3080                                      | ClaudeBot agent, gaming workload       |
| CyberNode   | CPU-Only Workstation  | CPU-focused (no GPU)                          | Claude Code agent, code generation     |
| NAS         | Storage & Networking  | TrueNAS                                       | Shared storage, inter-node communication |

---

### Quick Start  
1. **Install OpenClaw**  
   ```bash
   git clone https://github.com/yourusername/ki-core.git
   cd ki-core
   docker-compose up -d
   ```

2. **Configure Nodes**  
   Edit `config.yaml` to define agent roles, GPU allocation, and NAS paths.

3. **Start OpenClaw Gateway**  
   ```bash
   ./openclaw start --watchdog 300
   ```

---

### Documentation  
- [Architecture Details](docs/architecture.md)  
- [Configuration Guide](docs/configuration.md)  
- [Agent Setup](docs/agents.md)  
- [Troubleshooting](docs/troubleshooting.md)  

---

### License & Contributing  
This project is licensed under the **MIT License** – see [LICENSE](LICENSE) for details.  
Contributions are welcome! Please open an issue or submit a pull request via GitHub.  
```
