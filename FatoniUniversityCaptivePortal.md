# Autonomous Network & Internet-Auth Background Daemon Setup Guide

This guide details how to set up an automated background service on Debian to monitor internet connectivity and authenticate via an interactive captive portal login script using `curl`, `w3m`, and `expect`.

---

## Prerequisites

Ensure the required system tools are installed:

```bash
sudo apt update
sudo apt install -y curl expect w3m

```

---

## Step 1: Create the Connectivity Check Script

1. Create the check script in `/usr/local/bin/`:
```bash
sudo nano /usr/local/bin/check_internet.sh

```


2. Add the following code:
```bash
#!/bin/bash

if ! curl -s --max-time 5 [https://www.google.com](https://www.google.com) > /dev/null && \
   ! curl -s --max-time 5 [https://www.cloudflare.com](https://www.cloudflare.com) > /dev/null; then
    /usr/bin/expect /home/moha/run_w3m.exp
fi

```


3. Save, exit, and make the script executable:
```bash
sudo chmod +x /usr/local/bin/check_internet.sh

```



---

## Step 2: Create the Login Automation Script (`run_w3m.exp`)

1. Navigate to your user home directory and create the expect script:
```bash
cd /home/moha
nano run_w3m.exp

```


2. Add the following Tcl/Expect script:
```tcl
#!/usr/bin/expect -f

set timeout 10

# Spawn interactive w3m session
spawn w3m -cookie "[http://172.30.200.1:3990/prelogin](http://172.30.200.1:3990/prelogin)"

# Wait for page to load, then press Tab to focus Username
sleep 3
send "\t"

# Open text field, type username, press Enter
send "\r"
send "Write Your Student Username Given to you\r"

# Press Tab to jump to Password field
send "\t"

# Open text field, type password, press Enter
send "\r"
send "Write Here the Password\r"

# Press Tab to focus Submit button and press Enter
send "\t"
send "\t"
send "\r"

# Wait for form processing
sleep 2
send "\t"
send "\r"

# Quit w3m (send 'q' then confirm with 'y')
sleep 2
send "q"
send "y"

expect eof

```


3. Save, exit, and make it executable:
```bash
chmod +x /home/moha/run_w3m.exp

```



---

## Step 3: Configure Systemd Service

1. Create the systemd service file:
```bash
sudo nano /etc/systemd/system/check-net.service

```


2. Add the service definition:
```ini
[Unit]
Description=Check Internet and Run Login Script
After=network.target network-online.target

[Service]
Type=oneshot
User=moha
ExecStart=/usr/local/bin/check_internet.sh

[Install]
WantedBy=multi-user.target

```


3. Save and exit.

---

## Step 4: Configure Systemd Timer (Periodic Trigger)

1. Create the systemd timer file to run the check every 30 seconds:
```bash
sudo nano /etc/systemd/system/check-net.timer

```


2. Add the timer configuration:
```ini
[Unit]
Description=Run check-net every 30 seconds
Requires=check-net.service

[Timer]
OnBootSec=30sec
OnUnitActiveSec=30sec
Persistent=true
AccuracySec=1sec

[Install]
WantedBy=timers.target

```


3. Save and exit.

---

## Step 5: Enable and Start the Background Daemon

Apply the configurations and start the background timer:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now check-net.timer

```

---

## Monitoring and Maintenance

* **Check timer status:**
```bash
sudo systemctl status check-net.timer

```


* **View live service logs:**
```bash
sudo journalctl -u check-net.service -f

```


* **Manually test the script:**
```bash
/usr/local/bin/check_internet.sh

```
