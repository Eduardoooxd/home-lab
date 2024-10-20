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
