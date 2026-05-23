# Proxmox Setup

The AI-Core AI server runs as a Proxmox Virtual Machine (KVM/QEMU) on a physical host. This enables hardware isolation, snapshots, and clean PCIe passthrough for both GPUs.

---

## Host Configuration

The Proxmox physical host is the bare-metal machine that owns the hardware. Key configuration areas:

### PCIe Passthrough

Both GPUs are passed through to the AI-Core VM exclusively:

```
IOMMU groups:
  RTX 5060 Ti  → passed to AI-Core VM (GPU slot 1)
  RTX 3060     → passed to AI-Core VM (GPU slot 2)
```

**Requirements for GPU passthrough:**
1. BIOS: Enable VT-d (Intel) / AMD-Vi (AMD), enable SR-IOV if available
2. GRUB: Add `intel_iommu=on iommu=pt` (or `amd_iommu=on`) to kernel cmdline
3. Proxmox: Enable IOMMU in `/etc/default/grub` and `/etc/modules`
4. VM: Machine type `q35`, OVMF/UEFI firmware
5. Blacklist GPU drivers on host: `nouveau`, `nvidia` must NOT load on host

**Proxmox modules** (`/etc/modules`):
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

**Blacklist on host** (`/etc/modprobe.d/blacklist.conf`):
```
blacklist nouveau
blacklist nvidia
blacklist nvidiafb
```

### VM Configuration (AI-Core)

```ini
# /etc/pve/qemu-server/<vmid>.conf (sanitized)
name: AI-CORE
ostype: win11
machine: pc-q35-<version>
bios: ovmf
cpu: host,hidden=1,flags=+pcid
cores: 36
sockets: 1
memory: 131072          # 128 GB RAM
balloon: 0              # Disable memory ballooning (GPU passthrough requirement)

# GPU 1: RTX 5060 Ti
hostpci0: <pcie-addr-5060ti>,pcie=1,x-vga=1,rombar=0

# GPU 2: RTX 3060
hostpci1: <pcie-addr-3060>,pcie=1,rombar=0

# Storage
scsi0: <storage>:<vmid>/vm-<vmid>-disk-0.raw,cache=writeback,discard=on,size=500G
scsi1: <storage>:<vmid>/vm-<vmid>-disk-1.raw,cache=writeback,discard=on,size=2000G  # Model storage

# Network
net0: virtio=<mac>,bridge=vmbr0
```

### Key Proxmox Settings for Stability

- **CPU type:** `host` (pass all CPU flags through for AVX2/AVX512 support)
- **Machine type:** `q35` required for PCIe passthrough
- **Memory ballooning:** Disabled (required for GPU passthrough)
- **NUMA topology:** Match physical host NUMA if present
- **Hugepages:** Configure on host for improved memory performance

---

## VM Snapshots & Backups

Proxmox allows point-in-time snapshots. Note: GPU passthrough VMs cannot be snapshot while running on most platforms — shut down first.

**Recommended backup schedule:**
```bash
# Via Proxmox Backup Server (PBS) or vzdump
vzdump <vmid> --storage <backup-storage> --compress zstd --mode suspend
```

---

## Networking

The AI-Core VM connects via `virtio` NIC bridged to `vmbr0` (physical NIC bridge). The VM gets a static IP (`192.168.0.14`) via DHCP reservation or static assignment.

```
Physical Host
  └── vmbr0 (bridge to physical NIC)
        └── AI-Core VM eth0 → 192.168.0.14
```

---

## Additional VMs / Containers

The Proxmox host may run additional LXC containers or VMs. Suggested dedicated workloads:
- **LXC: NAS services** (if TrueNAS not on separate hardware)
- **LXC: Monitoring** (Prometheus + Grafana for system metrics)
- **VM: Dev environment** (isolated testing)

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| GPU not visible in VM | Verify IOMMU groups, check `dmesg` for VFIO errors |
| VM won't start after adding GPU | Check if host still has GPU driver loaded (`lsmod \| grep nvidia`) |
| Performance degradation | Disable memory ballooning, pin CPU cores |
| Network very slow | Use `virtio` NIC, not `e1000` |
| After Proxmox update: GPU passthrough broken | Re-check VFIO modules, rebuild initramfs (`update-initramfs -u`) |
