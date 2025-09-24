# Home Server Ansible Setup

This repository contains an Ansible setup for configuring your home server.

---

## Requirements

- Ansible installed on your control machine
- SSH access to the server

---

## Setup Instructions

### 1. Create hosts file

Copy or rename 'hosts.example' into 'hosts' file. Change server-name to whatever you want, also change ansible-host and user to you server IP and user

```ini
# Example
my-home-server ansible_host=10.0.0.115 ansible_user=ubuntu

[home_server]
my-home-server
```

### 2. Configure Variables

Before running any playbooks, you need to set your network configuration in `group_vars/all.yaml`:

```yaml
iface_name: 'enp2s0' # your network interface name
iface_ip: '10.0.0.20' # desired static IP
iface_gateway: '10.0.0.1' # gateway
iface_mask: '255.255.255.0' # Optional
```

Important: Make sure these values are correct. Running the playbook will change the network configuration and restart networking, which will disconnect your SSH session.

### 2. Run Initial Setup

Run the initial setup with the init tag:

```bash
ansible-playbook home-server.yaml -K --tags init
```

-K prompts for your sudo password.

Only the init task will run. This will change the IP to the static value you set and restart networking.

### 3. Update Hosts File

After the network change, your server’s IP will be changed. Update your hosts inventory file to use the new IP:

```ini
# Example
my-home-server ansible_host=10.0.0.20 ansible_user=ubuntu

[home_server]
my-home-server
```

Replace 10.0.0.115 (or whatever IP you had) with the new static IP you configured in variables.

### 4. Continue with Other Tasks

In development
