# Setup Guide

## Phase 1 - Infrastructure

### Step 1: Network Planning

Open VirtualBox and navigate to **Tools → Network → NAT Networks**. Create a new NAT network with the following settings:

- **Name:** soc-net
- **CIDR:** 10.0.5.0/24
- **DHCP:** Enabled

Once the network is created, add the following port forwarding rules so services running inside the VM are accessible from your host browser:

| Name    | Protocol | Host IP   | Host Port | Guest Port |
|---------|----------|-----------|-----------|------------|
| Wazuh   | TCP      | 127.0.0.1 | 8443      | 443        |
| TheHive | TCP      | 127.0.0.1 | 9000      | 9000       |
| ssh-core    | TCP      | 127.0.0.1 | 2210      | 10.0.5.10  | 22         |
| ssh-linux   | TCP      | 127.0.0.1 | 2230      | 10.0.5.30  | 22         |

> Guest IPs are assigned by DHCP. Note the IP of the VM after it boots and update the Guest IP column accordingly.

---

### Step 2: Wazuh Manager Deployment

Create a new VM with the following specifications:

- **OS:** Ubuntu 22.04 LTS Server
- **RAM:** 10GB
- **Disk:** 80GB
- **vCPUs:** 6
- **Network:** soc-net

Boot the VM and run the following to update the system and install Wazuh using the official installation assistant:

```bash
sudo apt update && sudo apt upgrade -y

curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

The installer will output admin credentials on completion - save these immediately as they are only shown once.

Once installed, verify all services are running:

```bash
sudo systemctl status wazuh-dashboard wazuh-indexer wazuh-manager
```

The Wazuh dashboard is accessible at `https://127.0.0.1:8443` from your host machine.

---
