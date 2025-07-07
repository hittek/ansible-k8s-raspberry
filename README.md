# Automated k3s Cluster Deployment on Raspberry Pi with Ansible

## Overview

This repository provides an Ansible-based solution for automating the deployment of a lightweight Kubernetes (k3s) cluster on Raspberry Pi devices. The playbook is designed for modern Raspberry Pi OS and Ubuntu images that use `/boot/firmware/cmdline.txt` for kernel parameters, and is suitable for both single-node and multi-node clusters.

## Prerequisites

- **Raspberry Pi OS or Ubuntu** installed on all Raspberry Pi devices.  
  Ensure your OS uses `/boot/firmware/cmdline.txt` for kernel parameters (default for recent images).
- **Ansible** installed on your control machine (Linux, macOS, or WSL).
- **SSH access** from the control machine to all Raspberry Pi nodes, with passwordless sudo privileges.
- **Network connectivity** between all nodes.

## Quick Start

### 1. Clone the Repository

```bash
git clone <this-repo-url>
cd ansible-k8s-raspberry
```

### 2. Configure Inventory

Edit the `hosts` file to list your master and worker nodes. For a single-node cluster:

```ini
[masternode]
raspberrypi4

[workers]
# No workers needed for single node setup
```

For a multi-node cluster, add additional hostnames or IPs under `[workers]`.

### 3. Prepare SSH Access

Ensure you can SSH into each Raspberry Pi as a user with passwordless sudo. For example:

```bash
ssh-copy-id pi@raspberrypi4
```

Test connectivity:

```bash
ansible -i hosts all -m ping --user pi
```

### 4. Run the Playbook

```bash
ansible-playbook -i hosts install_k3s.yaml --user <your-ssh-username>
```

- The playbook will:
  - Ensure required cgroup kernel parameters are set in `/boot/firmware/cmdline.txt`
  - Remove any conflicting parameters (e.g., `cgroup_disable=memory`)
  - Reboot nodes if kernel parameters are changed
  - Install k3s on the master node
  - Install k3s agents on worker nodes (if any)

### 5. Verify the Cluster

After the playbook completes and all nodes have rebooted, verify your cluster:

```bash
ssh <your-ssh-username>@raspberrypi4
sudo kubectl get nodes
```

You should see all nodes in the `Ready` state.

## File Structure

- `install_k3s.yaml` — Main Ansible playbook for cluster setup.
- `hosts` — Inventory file listing your Raspberry Pi nodes.

## Troubleshooting

- **Kernel parameters not set:**  
  The playbook will fail if the required cgroup parameters are not present in `/proc/cmdline` after a reboot. Manually check `/boot/firmware/cmdline.txt` and ensure all parameters are on a single line.
- **SSH or sudo issues:**  
  Ensure your user can SSH and run sudo commands without a password prompt.
- **iptables not found:**  
  The playbook installs `iptables` automatically if needed.

## References

- [k3s Documentation](https://rancher.com/docs/k3s/latest/en/)
- [Ansible Documentation](https://docs.ansible.com/)
