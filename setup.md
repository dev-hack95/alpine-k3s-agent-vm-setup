This README is designed specifically for your **Alpine Linux VM** running inside **QEMU/Termux**, connecting as an agent to a **Raspberry Pi** Master.

---

# 🚀 Alpine K3s Agent Setup Guide

This guide details the configuration of a lightweight Alpine Linux VM as a K3s Agent node.

## 🛠 Prerequisites

* **Alpine Linux** (v3.18+ recommended)
* **Master Node IP:** `192.168.1.2`
* **Node IP:** `10.0.2.15` (QEMU Default)
* **Node External IP:** `192.168.1.8`

---

## 1. System Preparation

Before installing K3s, Alpine requires specific dependencies and cgroup configurations.

```bash
# Update and install dependencies
apk update
apk add cgroup-tools iptables socat findutils dbus

# Fix Machine-ID (Required for Node Registration)
dbus-uuidgen > /etc/machine-id

# Enable and start Cgroups
rc-update add cgroups default
rc-service cgroups start

```

---

## 2. K3s Installation

Download the K3s binary and set execution permissions.

```bash
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.2:6443 \
K3S_TOKEN=K109008e4da73637304b234295e92b9a9dbe4115b54b4691526e695ee76ce98687d::server:VasMe9uwbwMXaZFUcovXt4zcorA= \
sh -s - agent

```

---

## 3. Persistent Configuration (OpenRC)

To ensure the agent starts correctly on boot with the specific network flags required for an emulated environment, configure the OpenRC environment file.

**File:** `/etc/conf.d/k3s-agent`

```bash
command_args="agent \
  --server https://192.168.1.2:6443 \
  --token K109008e4da73637304b234295e92b9a9dbe4115b54b4691526e695ee76ce98687d::server:VasMe9uwbwMXaZFUcovXt4zcorA= \
  --node-ip 10.0.2.15 \
  --node-external-ip 192.168.1.8"

```

**Enable the Service:**

```bash
rc-update add k3s-agent default
rc-service k3s-agent start

```

---

## 4. Docker & Docker-Compose

Optional: Install Docker for side-car container tasks.

```bash
# Install
apk add docker docker-compose

# Enable & Start
rc-update add docker boot
rc-service docker start

# Add user to group
addgroup <username> docker

```

---

## 5. Troubleshooting & Logs

Since K3s is supervised by `supervise-daemon`, logs are redirected to a file.

| Task | Command |
| --- | --- |
| **Check Live Logs** | `tail -f /var/log/k3s-agent.log` |
| **Check Service Status** | `rc-service k3s-agent status` |
| **Check Kubelet Health** | `curl http://127.0.0.1:10248/healthz` |
| **Verify on Master** | `kubectl get nodes` (Run on Raspberry Pi) |

### Common Fixes:

* **Duplicate IP Error:** Ensure you don't have overlapping configs in `/etc/rancher/k3s/config.yaml` and `/etc/conf.d/k3s-agent`.
* **Port 10250 Busy:** Run `killall -9 k3s` and restart the service.

---

**Would you like me to help you create a custom `docker-compose.yaml` to test your new Docker installation?**
