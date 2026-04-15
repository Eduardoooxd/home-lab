# Raspberry Pi Zero 2W — Pi-hole Setup Guide

This guide covers setting up a Raspberry Pi Zero 2W as a Pi-hole DNS sinkhole on the local network.

## Prerequisites

- Raspberry Pi Zero 2W with a microSD card (8GB+)
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/) installed on the machine
- SSH key pair on the machine (generated in step 4 below)

---

## 1. Flash the OS

1. Open Raspberry Pi Imager.
2. Choose **Raspberry Pi OS Lite (32-bit)** — no desktop needed.
3. Select the microSD card.
4. Click the **gear icon (⚙️)** to open advanced options:
   - Set **hostname**: `pihole`
   - Enable **SSH** → select **Use password authentication** (key-based auth is set up after first boot)
   - Set **username/password** (e.g. user: `pi`)
   - Configure **Wi-Fi** (SSID + password + country code)
5. Write the image.

---

## 2. First Boot & Connect via Password

Insert the card, power on the Pi, wait ~60 seconds, then connect:

```bash
# Option A — check router's DHCP leases for the assigned IP
# Option B — try mDNS hostname
ssh eduardo@rpi-zero2w
```

---

## 3. Initial System Update

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt autoremove -y
sudo reboot
```

Reconnect after reboot:

```bash
ssh eduardo@rpi-zero2w
```

---

## 4. Set Up SSH Key Authentication

### Generate a key pair on the local machine (not on the Pi)

```bash
ssh-keygen -t ed25519 -C "eduardo@rpi-zero2w" -f ~/.ssh/pi-zero2w
```

- Setting a passphrase adds extra security (optional but recommended)

### Copy the public key to the Pi

```bash
ssh-copy-id -i ~/.ssh/pi-zero2w.pub eduardo@rpi-zero2w
```

This appends the public key to `~/.ssh/authorized_keys` on the Pi. The password will be prompted one last time.

**Alternatively**, if `ssh-copy-id` is not available:

```bash
cat ~/.ssh/pi-zero2w.pub | ssh eduardo@rpi-zero2w "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

### Test key-based login

Open a new terminal and verify login works without a password:

```bash
ssh -i ~/.ssh/pi-zero2w eduardo@rpi-zero2w
```

If it connects without prompting for a password, key auth is working. After adding the SSH config shortcut (see end of this guide), the `-i` flag is no longer needed.

### Disable password authentication on the Pi

```bash
sudo nano /etc/ssh/sshd_config
```

Set these lines (uncomment if needed):

```
PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin no
```

Restart SSH:

```bash
sudo systemctl restart ssh
```

> **Important:** Confirm key login works in a separate terminal _before_ closing the current session, to avoid getting locked out.

### (Optional) Change the default SSH port

In `/etc/ssh/sshd_config`, change:

```
Port 2222
```

Restart SSH and reconnect with `ssh -p 2222 eduardo@rpi-zero2w`.

---

## 5. Static IP

Raspberry Pi OS Bookworm uses **NetworkManager** instead of `dhcpcd`. Use `nmcli` to configure the static IP.

First, find the Wi-Fi connection name:

```bash
nmcli con show
```

It is typically named after the SSID or `preconfigured`. Then apply the static IP (replace `"preconfigured"` with the actual connection name):

```bash
sudo nmcli con mod "preconfigured" ipv4.addresses 192.168.1.105/24
sudo nmcli con mod "preconfigured" ipv4.gateway 192.168.1.1
sudo nmcli con mod "preconfigured" ipv4.dns "127.0.0.1 1.1.1.1"
sudo nmcli con mod "preconfigured" ipv4.method manual
sudo nmcli con up "preconfigured"
```

> **Note:** Using `127.0.0.1` as the primary DNS points the Pi itself to Pi-hole after installation. `1.1.1.1` is kept as a fallback for bootstrap resolution.

Verify the new address:

```bash
ip addr show wlan0
```

Reconnect using the new static IP:

```bash
ssh -i ~/.ssh/pi-zero2w eduardo@192.168.1.105
```

---

## 6. Enable Firewall (UFW)

Install and configure UFW:

```bash
sudo apt install ufw -y

# Allow SSH (use the port configured — default 22)
sudo ufw allow 22/tcp

# Allow Pi-hole web UI (HTTP)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow DNS (TCP + UDP)
sudo ufw allow 53/tcp
sudo ufw allow 53/udp

# Allow DHCP if Pi-hole will be used as DHCP server (optional)
# sudo ufw allow 67/udp

# Enable the firewall
sudo ufw enable

# Verify rules
sudo ufw status verbose
```

---

## 7. Install Pi-hole

```bash
curl -sSL https://install.pi-hole.net | bash
```

Walk through the interactive installer:

| Prompt                | Recommended choice                  |
| --------------------- | ----------------------------------- |
| Interface             | `wlan0`                             |
| Upstream DNS          | Cloudflare (`1.1.1.1`) or preferred |
| Blocklists            | Keep defaults, add more later       |
| Admin web interface   | Yes                                 |
| Web server (lighttpd) | Yes                                 |
| Log queries           | Yes                                 |
| Privacy mode          | 0 — Show everything                 |

At the end the installer prints a **random admin password**. Save it or reset it:

```bash
pihole -a -p
```

---

## 8. Access the Admin Dashboard

Open in a browser:

```
http://192.168.1.105/admin
```

Log in with the password set during installation.

---

## 9. Point the Router to Pi-hole

Log into the router admin panel and set the **primary DNS** to `192.168.1.105`. This routes all DNS queries on the network through Pi-hole.

Alternatively, set it per device to avoid changing router settings.

---

## 10. Verify Everything Works

```bash
# On the Pi — check Pi-hole status
pihole status

# On any device using Pi-hole DNS
nslookup doubleclick.net 192.168.1.105
# Should return 0.0.0.0 (blocked)

nslookup google.com 192.168.1.105
# Should resolve normally
```

---

## Useful Pi-hole Commands

```bash
pihole status          # Show overall status
pihole -up             # Update Pi-hole
pihole -g              # Update gravity (blocklists)
pihole tail            # Live query log
pihole -a -p           # Change admin password
pihole restartdns      # Restart DNS resolver
```

---

## SSH Config Shortcut

Add to `~/.ssh/config` on the local machine for easy access:

```
Host rpi-zero2w
    HostName 192.168.1.105
    User eduardo
    IdentityFile ~/.ssh/pi-zero2w
```

Then connect with: `ssh rpi-zero2w`
