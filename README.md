```bash
sudo apt update && sudo apt upgrade -y
```

```bash
sudo apt install docker -y
```

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

## TODO

- Add Fail2Ban
- Add Wireguard
- Add ufw-docker
- Add cloudflare tunnels
