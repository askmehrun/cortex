# GPU Passthrough vs Container GPU Access

Expose a GPU to a workload two ways: pass the whole device to a VM (vfio-pci), or share the host driver with a container (nvidia-container-toolkit). Choose by isolation and performance needs.

## When to Use
- Full isolation, a different guest OS, or maximum GPU performance → VM passthrough.
- Fast access on the same host, or time-sliced sharing between users → container access.

## How
### VM GPU Passthrough (vfio-pci)
- Passes the entire PCIe device to the VM; the VM sees raw hardware and installs its own driver.
- Best isolation, best performance. The device is unavailable to the host while passed through.
1. Identify the IOMMU group: `find /sys/kernel/iommu_groups/*/devices/* -type l | grep nvidia`
2. Bind it in the Proxmox VM config: `hostpci0: 0000:01:00.0,mdev=none`
3. Install the NVIDIA driver inside the VM

### Container GPU Access (NVIDIA Container Toolkit)
- Shares the host kernel and uses the host's NVIDIA driver.
- No isolation — the container sees the same driver as the host.
- `--gpus all` exposes the GPU; setup needs no reboot.
```bash
apt install nvidia-container-toolkit
nvidia-ctk runtime configure --runtime=docker
systemctl restart docker
docker run --gpus all nvidia/cuda:12.0-base nvidia-smi
```

## Gotchas
- vfio-pci and container access are mutually exclusive: pass a device to a VM and the host loses it.
- Containers cannot get the device via passthrough — they have no separate kernel context.
- Passthrough requires IOMMU grouping and typically a reboot.

## Rule
Containers share the host kernel, so they use the host driver; passthrough gives the device to a separate kernel context that containers can't reach.
