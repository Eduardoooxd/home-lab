# Eduardo Couto Home Lab

## Architecture of the solution

![Eduardo Couto Architecuture Diagram](./assets/architecture.svg)

### Update the server

```bash
sudo apt update && sudo apt upgrade -y
```

### Setup Message of Day

```bash
sudo tee /etc/motd < assets/frank_ocean.txt
```

```bash
sudo tee /etc/motd < assets/die_lit.txt
```

### Install Essential Pacakages

```bash
sudo apt install git btop -y
```

#### Install Docker

```bash
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker eduardo
```

Enable Docker API to allow portainer to access containers info

```bash

# Create directory for daemon configuration
sudo mkdir -p /etc/systemd/system/docker.service.d

# Create override file
sudo nano /etc/systemd/system/docker.service.d/override.conf

```

Add this content to override.conf file

```bash
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:2375
```

Then

```bash
# Reload daemon and restart Docker
sudo systemctl daemon-reload
sudo systemctl restart docker

# Verify API is listening
curl http://localhost:2375/version
```

#### Create External Docker Network

```bash
docker network create proxy
```

### Configure Git

```bash
  git config --global user.email "you@example.com"
  git config --global user.name "Eduardo Couto"
  git config --global credential.helper store
```

### Change Key SSH Configurations

#### On Local Machine

#### Generate SSH Key Pair

```bash
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/ed25519_rpi5-1
```

#### Copy the Public Key to the Raspberry Pi

```bash
ssh-copy-id -i ~/.ssh/ed25519_rpi5-1.pub username@pi_ip_address
```

#### Login into RPI

```bash
ssh username@pi_ip_address
```

#### Edit SSH configuration to disable password authentication

```bash
sudo nano /etc/ssh/sshd_config
```

```
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```

### Define Static IP

```bash
sudo nano /etc/dhcpcd.conf
```

#### Add static IP Configuration

```bash
interface eth0  # or wlan0 if using Wi-Fi
static ip_address=192.168.1.X/24
static routers=192.168.1.1  # Your router's IP address
static domain_name_servers=192.168.1.1 8.8.8.8  # Router or public DNS like Google's
```

#### Reboot System

```bash
sudo reboot
```

### Firewall Setup

```bash
sudo apt install ufw -y

# Allow outgoing traffic
sudo ufw default allow outgoing

# Deny All Incoming Trafic
sudo ufw default deny incoming

## SSH Port
sudo ufw allow ssh

# Allow WireGuard port
# sudo ufw allow 51820/udp comment 'WireGuard VPN'

## Allow Docker API
sudo ufw allow 2375

# Enable Firewall
sudo ufw enable

sudo ufw status
```

### Set up Fail2Ban

## TODO

- Add Fail2Ban
- Add Wireguard
- Add ufw-docker
- Add cloudflare tunnels
