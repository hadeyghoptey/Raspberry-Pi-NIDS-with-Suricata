# Raspberry Pi NIDS with Suricata + GitHub Alert Bot

A lightweight **Network Intrusion Detection System (NIDS)** built on a **Raspberry Pi** using **Suricata**, with automated **GitHub-based alert notifications**.

This project is designed to be:
- Fully **CLI-based** (no GUI)
- **Lightweight** and stable
- Suitable for **learning blue-team / SOC fundamentals**
- Easy to reproduce on low-resource hardware

---

## 📌 System Overview

- **Hardware**: Raspberry Pi  
- **Operating System**: Raspberry Pi OS Lite (32-bit)  
- **NIDS Engine**: Suricata  
- **Alerting**: Custom Python bot → GitHub Issues  
- **Interface**: Ethernet or Wi-Fi (monitor mode not required)

---

## 🖥️ Operating System Details

This setup uses:

- **Raspberry Pi OS Lite (32-bit)**
- Downloaded from the **official Raspberry Pi website**
- Installed using **Raspberry Pi Imager**
- No desktop environment (CLI only)

Why 32-bit Lite:
- Better package compatibility on Raspberry Pi
- Lower RAM and storage usage
- More stable for IDS and security tooling

---

## 🔧 Initial OS Setup

After installing Raspberry Pi OS Lite and logging in:

### 1️⃣ Update the system
```bash
sudo apt update
sudo apt full-upgrade -y
sudo reboot
````

---

### 2️⃣ Basic configuration

```bash
sudo raspi-config
```

Recommended settings:

* Expand filesystem
* Set timezone and locale
* Enable SSH
* Change default password

Reboot after changes.

---

## 🔐 Basic Hardening (Recommended)

```bash
sudo apt install -y ufw fail2ban
```

Enable firewall:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw enable
```

---

## 🛠️ Installing Suricata (NIDS)

### 1️⃣ Install Suricata

```bash
sudo apt install -y suricata
```

Verify installation:

```bash
suricata --build-info
```

---

### 2️⃣ Identify network interface

```bash
ip link
```

Common interfaces:

* `eth0` → Ethernet
* `wlan0` → Wi-Fi

---

### 3️⃣ Configure Suricata interface

Edit the configuration file:

```bash
sudo nano /etc/suricata/suricata.yaml
```

Locate the `af-packet` section and configure it:

```yaml
af-packet:
  - interface: eth0
    cluster-id: 99
    cluster-type: cluster_flow
    defrag: yes
```

Replace `eth0` with your actual interface if needed.

---

### 4️⃣ Update Suricata rules

```bash
sudo apt install -y suricata-update
sudo suricata-update
```

---

### 5️⃣ Enable and start Suricata

```bash
sudo systemctl enable --now suricata
sudo systemctl status suricata --no-pager
```

---

### 6️⃣ Verify alerts

```bash
sudo tail -f /var/log/suricata/fast.log
```

From another machine, generate traffic:

```bash
nmap -sS <PI_IP_ADDRESS>
```

You should see scan-related alerts.

---

## 🚨 GitHub Alert Bot (Notifier)

Suricata alerts are automatically pushed to GitHub using a Python-based bot.

---

### 1️⃣ Create a GitHub repository

* Create a **private repository**
* Generate a **Personal Access Token**

  * Scope: `repo`

---

### 2️⃣ Install dependencies

```bash
sudo apt install -y git python3-pip
pip3 install requests
```

---

### 3️⃣ Clone your alert repository

```bash
git clone https://github.com/<YOUR_USERNAME>/suricata-alerts.git
cd suricata-alerts
```

---

### 4️⃣ Python notifier script

Create `notifier.py`:

````python
import requests
import time

GITHUB_TOKEN = "YOUR_GITHUB_TOKEN"
REPO = "YOUR_USERNAME/suricata-alerts"

HEADERS = {
    "Authorization": f"token {GITHUB_TOKEN}",
    "Accept": "application/vnd.github.v3+json"
}

SURICATA_LOG = "/var/log/suricata/fast.log"
SEEN = set()

def send_issue(alert):
    url = f"https://api.github.com/repos/{REPO}/issues"
    data = {
        "title": "🚨 Suricata Alert Detected",
        "body": f"```\n{alert}\n```"
    }
    requests.post(url, headers=HEADERS, json=data)

while True:
    with open(SURICATA_LOG, "r") as f:
        lines = f.readlines()[-10:]

    for line in lines:
        if line not in SEEN:
            SEEN.add(line)
            send_issue(line)

    time.sleep(15)
````

---

### 5️⃣ Run the notifier

```bash
sudo python3 notifier.py
```

Trigger traffic again and verify GitHub Issues are created automatically.

---

### 6️⃣ Run notifier as a service

Create service file:

```bash
sudo nano /etc/systemd/system/suricata-github.service
```

```ini
[Unit]
Description=Suricata GitHub Alert Bot
After=network.target suricata.service

[Service]
ExecStart=/usr/bin/python3 /home/<USER>/suricata-alerts/notifier.py
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now suricata-github
```

---

## 📊 What This Project Demonstrates

* Real-time **network intrusion detection**
* Practical **blue team / SOC workflow**
* Lightweight security monitoring on constrained hardware
* Automated alerting without paid services
* Reproducible and extensible design

---

## 🚀 Future Improvements

* Switch to `eve.json` for structured alerts
* Alert filtering by severity
* Add Discord / Slack notifications
* Combine Suricata with Zeek
* Apply ML-based anomaly detection (advanced)

---

## ⚠️ Disclaimer

This project is for **educational and defensive security purposes only**.
Do not deploy on networks you do not own or have permission to monitor.

---
