# Home Server Ansible Setup

This repository contains an Ansible setup for configuring your home server.

---

## Requirements

- Debian Server
- Ansible installed on your control machine
- SSH access to the server

---

## SSH Setup

Install Open SSH Server on Debian Machine. It can can be installed durin Debian setup.

On yor host machine run

```bash
ssh-keygen
ssh-copy-id username@client-ip
```

And then connect 

```bash
ssh username@client-ip
```

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

Before running any playbooks, copy or rename 'all.yaml.example' into 'all.yaml' file. After that you need to set your network configuration in `group_vars/all.yaml`:

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

### 4. Install Docker

Once your server has a static IP, you can install Docker using Ansible:

```bash
ansible-playbook home-server.yaml -K --tags docker
```

This sets up Docker so you can run applications in isolated containers.

**Optional: Install Portainer**

Portainer is a web interface for managing Docker. It lets you:

- See running containers
- Start, stop, or restart containers
- View logs and settings
- Create new containers without the command line

Install it with:

```bash
ansible-playbook home-server.yaml -K --tags portainer
```

After installation, open a browser and go to:

```bash
https://<your-server-ip>:9443
```

Create an admin account, and you can manage Docker visually. Portainer is optional but convenient.

### 5. Install Home Assistant

After docker is installed, deploy Home Assistant with:

```bash
ansible-playbook home-server.yaml -K --tags ha
```

Default config path is /opt/homeassistant/config (can be changed in group_vars/all.yaml).

You can configure USB devices and Bluetooth in group_vars/all.yaml:

```yaml
usb_devices:
  [
    '/dev/serial/by-id/usb-ITead_Sonoff_Zigbee-if00-port0',
    '/dev/serial/by-id/other_device',
  ]
use_bluetooth: false # set to true if you want Bluetooth support
```

- Finding your Zigbee USB: run ls /dev/serial/by-id/ — look for a device like usb-manufacturer-if00-port0.

- Enabling Bluetooth: set use_bluetooth: true. The playbook will install and start BlueZ automatically if you have a dongle.

Access Home Assistant at:

```bash
http://<your-server-ip>:8123
```

and create your smart home. You can find official documentation on how to set it up [here](https://www.home-assistant.io/getting-started/onboarding/).

To update Home Assistant use:

```bash
ansible-playbook home-server.yaml -K --tags ha -e ha_action=update
```

### 6. Setup Remote Connection

To enable remote access, choose your VPN provider in group_vars and run the VPN tag:

```bash
ansible-playbook home-server.yaml -K --tags vpn
```

Provider-specific setup instructions are listed below.

#### 6.1 Cloudflare

To enable remote access through Cloudflare Tunnel, set the VPN provider and add your Cloudflare Tunnel token:

```yaml
vpn_provider: "cloudflare"
cloudflare_token: "YOUR_TUNNEL_TOKEN"
```

#### 6.2 Tailscale

To enable remote access with Tailscale, set the VPN provider and add your Tailscale Auth key. Key can be found in Tailscale dashboard Settings > Personal Settings > Keys > Auth keys

```yaml
vpn_provider: "tailscale"
tailscale_auth_key: "tskey-auth-key"
```

Setup your other devices to use Tailscale as VPN and access your services through Tailscale VPN.

#### 6.3 Wireguard

### 7. Continue with Other Tasks

In development
