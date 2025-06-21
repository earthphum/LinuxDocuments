# How to Set Up Cloudflared Tunnel as a Service on Raspberry Pi

This guide explains how to run your Cloudflare Tunnel automatically as a systemd service on your Raspberry Pi. This way, the tunnel will start automatically whenever the Pi boots up.

---

## Prerequisites

- Cloudflared installed and configured on your Raspberry Pi
- A working Cloudflare Tunnel (e.g., named `mytunnel`)
- Access to Raspberry Pi terminal with sudo privileges

---

## Steps

### 1. Create a systemd Service File

Open a new service file with your preferred text editor (nano is used here):

```bash
sudo nano /etc/systemd/system/cloudflared.service
```

### 2. Add Following
```bash
[Unit]
Description=Cloudflare Tunnel
After=network.target

[Service]
ExecStart=/usr/local/bin/cloudflared tunnel run mytunnel
Restart=on-failure
User=your
WorkingDirectory=~/.cloudflared # or /home/your/.cloudflared Maybe 555 i dont know bro but i use a second choice

[Install]
WantedBy=multi-user.target
```
### 3. Then run a command below
```bash
sudo systemctl daemon-reload
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

### My tip
If you change any config at ~/.cloudflared or etc 
```bash
sudo systemctl restart cloudflared
``` 
