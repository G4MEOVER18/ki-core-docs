# KI-CORE Distributed AI System Setup Guide  
**Documentation Version 1.0**  
**Last Updated: 2023-10-15**  

---

## 1. Prerequisites  

### **Hardware Requirements**  
- **Host System**:  
  - x86_64 architecture with support for IOMMU (Intel VT-d or AMD-Vi)  
  - Minimum 16 GB RAM (32 GB recommended for dual GPU workloads)  
  - Dual GPU support (e.g., NVIDIA A100, RTX 3090, or equivalent)  
  - 1 TB+ NVMe SSD for VM storage  
- **Network**:  
  - Gigabit Ethernet for VM host and guest communication  
  - SMB-enabled NAS for inter-node sync  

### **Software Requirements**  
- **Proxmox VE**: 7.x or later  
- **Windows 11 Enterprise**: 22H2 or later  
- **NVIDIA Drivers**: 535.xx or later (compatible with CUDA 12.x)  
- **Docker Desktop for Windows**: 4.12 or later  
- **Node.js**: 18.x (for OpenClaw)  
- **TrueNAS SMB Server**: 12.0+ with shared directory  

---

## 2. Proxmox Host Setup  

### **Enable IOMMU and VFIO**  
1. **BIOS/UEFI Settings**:  
   - Enable **Intel VT-d** (Intel) or **AMD-Vi** (AMD)  
   - Disable **Secure Boot**  
   - Set **CPU Mode** to **"Host CPU"** (for GPU passthrough)  

2. **Kernel Parameters**:  
   Edit `/etc/default/grub`:  
   ```bash
   GRUB_CMDLINE_LINUX="intel_iommu=on iommu=pt"
   ```  
   Update GRUB:  
   ```bash
   update-grub
   update-initramfs -u
   ```  

3. **PCIe Passthrough Configuration**:  
   - Identify GPU PCIe IDs via `lspci -v`  
   - Add GPUs to VM in Proxmox:  
     ```bash
     virsh nodedev-list
     virsh nodedev-detach <GPU_ID>
     ```  
   - Assign GPUs to the Windows VM in Proxmox VM settings.  

---

## 3. Windows 11 VM Configuration  

### **Resource Allocation**  
- **CPU**: 8 cores (ensure CPU model supports GPU passthrough)  
- **RAM**: 16 GB (adjust based on workload)  
- **Storage**: 50 GB SSD (expandable)  
- **Network**: Bridged mode for direct IP assignment  

### **VM Settings**  
- Enable **PCI Passthrough** for assigned GPUs  
- Disable **Hyper-V** and **Windows Sandbox** in BIOS  
- Install **Windows 11 Enterprise** via ISO  

---

## 4. NVIDIA Driver Installation in VM  

### **Install Drivers**  
1. Download NVIDIA drivers from [NVIDIA官网](https://www.nvidia.com/Download/index.aspx) (select "Windows Server" or "Windows 11" version).  
2. Install drivers via **Device Manager** (use "Custom Installation" to avoid conflicts).  
3. Verify GPU detection:  
   ```powershell
   Get-WmiObject Win32_VideoController | Select Name, DriverVersion
   ```  

---

## 5. Dual Ollama Setup  

### **Install Ollama**  
1. Download Ollama from [ollama.com](https://ollama.com/download) and install on the Windows VM.  
2. **Instance 1 (GPU 0)**:  
   ```bash
   CUDA_VISIBLE_DEVICES=0 ollama serve --host 0.0.0.0 --port 11434
   ```  
3. **Instance 2 (GPU 1)**:  
   ```bash
   CUDA_VISIBLE_DEVICES=1 ollama serve --host 0.0.0.0 --port 11435
   ```  
4. Configure `OLLAMA_HOST` in environment variables for each instance.  

---

## 6. Docker Desktop + Docker Compose Deployment  

### **Install Docker Desktop**  
- Download from [docker.com](https://www.docker.com/products/docker-desktop/) and install.  
- Enable **Use Windows containers** and **Enable experimental features**.  

### **Docker Compose Configuration**  
Create `docker-compose.yml` in a dedicated folder:  
```yaml
version: '3.8'
services:
  open-webui:
    image: openwebui/open-webui:latest
    ports: ["3000:3000"]
    environment:
      - OLLAMA_HOST=http://host.docker.internal:11434
  litellm:
    image: litellm/litellm:latest
    ports: ["4000:4000"]
  comfyui:
    image: commodore/comfyui:latest
    ports: ["8188:8188"]
  searxng:
    image: searxng/searxng:latest
    ports: ["8888:8888"]
  portainer:
    image: portainer/portainer-ce:latest
    ports: ["9000:9000"]
  postgres:
    image: postgres:14
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
  redis:
    image: redis:alpine
  watchtower:
    image: watchtower:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```  

### **Start Services**  
```powershell
docker-compose up -d
```

---

## 7. OpenClaw Installation and Configuration  

### **Install OpenClaw**  
```bash
npm install -g openclaw
```  

### **Configure Gateway**  
1. Create `config.json` in the OpenClaw directory:  
   ```json
   {
     "agents": ["agent1", "agent2"],
     "llm": {
       "host": "http://localhost:11434",
       "model": "llama2"
     },
     "storage": {
       "type": "smb",
       "path": "\\\\NAS_IP\\shared\\chat"
     }
   }
   ```  
2. Run OpenClaw:  
   ```bash
   openclaw --config config.json
   ```  

---

## 8. Agent Workspace Setup  

### **Create Agent Files**  
- **AGENTS.md**:  
  ```markdown
  # Agents
  - **Agent 1**: Role: Data Analyst, Skills: SQL, Python
  - **Agent 2**: Role: Content Creator, Skills: Markdown, SEO
  ```  
- **IDENTITY.md**:  
  ```markdown
  # Identity
  - Organization: KI-CORE
  - Purpose: Collaborative AI Research
  ```  
- **SOUL.md**:  
  ```markdown
  # Soul
  - Core Values: Innovation, Transparency, Ethics
  - Mission: Democratize AI Access
  ```  

---

## 9. Windows Task Scheduler Tasks  

### **Auto-Recovery Tasks**  
1. **Gateway Start**:  
   - Trigger: At logon  
   - Action: Start `openclaw` via batch file:  
     ```batch
     @echo off
     cd C:\path\to\openclaw
     openclaw --config config.json
     ```  
2. **Watchdog Task**:  
   - Trigger: Every 5 minutes  
   - Action: Run PowerShell to check process:  
     ```powershell
     if (!(Get-Process -Name "openclaw" -ErrorAction SilentlyContinue)) {
         Start-Process "C:\path\to\openclaw\openclaw.exe" -ArgumentList "--config config.json"
     }
     ```  

---

## 10. NAS Integration  

### **Mount SMB Share**  
1. Open **File Explorer** > **This PC** > **Map network drive**.  
2. Enter SMB path: `\\NAS_IP\shared`  
3. Set username/password for TrueNAS share.  

### **Configure Chat Directory**  
Update OpenClaw config to use mounted SMB path:  
```json
"storage": {
  "type": "smb",
  "path": "\\\\NAS_IP\\shared\\chat"
}
```  

---

## 11. Verification Checklist  

| Component               | Verification Steps                              |  
|------------------------|------------------------------------------------|  
| GPU Passthrough        | `nvidia-smi` shows GPU utilization             |  
| Docker Services        | `docker ps` lists all containers               |  
| OpenClaw Logs          | Check `logs/` directory for errors             |  
| NAS Sync              | Verify SMB mount and chat directory permissions|  
| Agent Workflows        | Test agent interactions via OpenClaw UI       |  

---

**Note**: Replace placeholder values (e.g., `NAS_IP`, `CUDA_VISIBLE_DEVICES`) with your environment-specific configurations. Always test in a non-production environment before deployment.
