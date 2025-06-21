
# Note: You need to have a domain name and have it connected to Cloudflare before using this.
# Raspberry Pi Cloudflare Tunnel Setup

This guide explains how to set up a Cloudflare Tunnel (Argo Tunnel) on a Raspberry Pi to expose a local web server running on port 80 securely to the internet using a custom domain managed by Cloudflare.

---

## Prerequisites

- Raspberry Pi with SSH access
- A domain managed by Cloudflare
- A web server running on Raspberry Pi on port 80 (e.g., nginx, Apache)
- Basic familiarity with terminal commands

---

## Steps

### 1. Install `cloudflared`

Download and install the latest `cloudflared` binary for ARM64:

```bash
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64 -o cloudflared
chmod +x cloudflared
sudo mv cloudflared /usr/local/bin/
```
### test intsallation
```bash
cloudflared --version
```
### 2.Authenticate with Cloudflare

Login and authorize cloudflared to access your Cloudflare account:
```bash
cloudflared tunnel login
```
This command will open a browser window for you to complete the login.

### 3.create a tuunel
Create a new tunnel (replace mytunnel with a name of your choice):
```bash
cloudflared tunnel create mytunnel
```
### 4. Config
```bash
tunnel: <Tunnel-ID>
credentials-file: ~/.cloudflared/<Tunnel-ID>.json # if path not found change it manualy naka

ingress:
  - hostname: your.domain.com
    service: http://localhost:80
  - service: http_status:404
```
### 5. Route DNS to Tunnel
```
cloudflared tunnel route dns mytunnel your.domain.com
```
### 6. Run the Tunnel
```
cloudflared tunnel run mytunnel
```
### 7. Final try to access your domain 
example xxx.yourdomain.com 
