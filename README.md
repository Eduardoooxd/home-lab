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
sudo apt install git docker -y
```

### Change Key SSH Configurations

### Firewall Setup

```bash
sudo apt install ufw -y

## SSH Port
sudo ufw allow 22

# Allow outgoing traffic
sudo ufw default allow outgoing

# Deny All Incoming Trafic
sudo ufw default deny incoming

# Enable Firewall
sudo ufw enable
```

### Set up Fail2Ban

## TODO

- Add Fail2Ban
- Add Wireguard
- Add ufw-docker
- Add cloudflare tunnels
