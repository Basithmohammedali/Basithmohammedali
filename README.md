# Somnium Lab — HTB Community Machine

> "You mustn't be afraid to dream a little bigger, darling."

An Inception-themed HackTheBox-style vulnerable machine. A dream research portal with a file upload vulnerability as the entry point and a sudo misconfiguration for privilege escalation.

---

## Difficulty
**Easy**

## Attack Path
```
File Upload (webshell) → www-data shell → lateral move to user → sudo PrivEsc → root
```

## Setup Instructions

### Requirements
- Ubuntu 22.04 LTS (fresh install)
- VirtualBox / VMware
- Host-only network adapter

### Deploy

```bash
# Clone the repo
git clone https://github.com/yourusername/somnium-htb
cd somnium-htb

# Copy app to /opt
sudo cp -r . /opt/somnium

# Run setup
sudo bash setup.sh
```

### Manual Setup (step by step)

```bash
# 1. Install Python & Flask
sudo apt update
sudo apt install python3 python3-pip python3-venv -y

# 2. Set up app
sudo mkdir -p /opt/somnium
sudo cp -r . /opt/somnium
cd /opt/somnium
python3 -m venv venv
source venv/bin/activate
pip install flask

# 3. Set permissions
sudo chown -R www-data:www-data /opt/somnium/static/uploads

# 4. Install and start service
sudo cp somnium.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable somnium
sudo systemctl start somnium

# 5. Set up sudo misconfiguration (PrivEsc path)
echo "red ALL=(root) NOPASSWD: /usr/bin/find" | sudo tee -a /etc/sudoers

# 6. Plant flags
echo "HTB{dr3am_with1n_4_dr34m}" | sudo tee /home/red/user.txt
echo "HTB{y0u_4r3_th3_arch1t3ct}" | sudo tee /root/root.txt
sudo chmod 644 /home/red/user.txt
sudo chmod 600 /root/root.txt
```

---

## Vulnerabilities

| Step | Vulnerability | Details |
|------|--------------|---------|
| Entry | Unrestricted File Upload | Upload a `.php` webshell, execute via `/static/uploads/shell.php` |
| Lateral | Config file credentials | Password found in app config or `/opt/somnium/app.py` |
| PrivEsc | sudo misconfiguration | `red` can run `/usr/bin/find` as root — GTFOBins |

---

## Credentials (for setup only — do not include in OVA)

| User | Password | Note |
|------|----------|------|
| admin | inception2024 | Web admin |
| red | [set your own] | Low-priv SSH user |

---

## Intended Path (Spoiler)

1. Register on the portal
2. Upload a PHP webshell (`shell.php`) via the dream fragment uploader
3. Navigate to `/static/uploads/shell.php?cmd=id`
4. Get reverse shell as `www-data`
5. Read `/opt/somnium/app.py` — find `red`'s password in comments or config
6. `su red` → grab `user.txt`
7. `sudo find . -exec /bin/bash \; -quit` → root shell
8. Grab `root.txt`

---

## Notes
- No internet connection required on the VM
- All dependencies installed locally
- Service auto-restarts on crash (stable)
