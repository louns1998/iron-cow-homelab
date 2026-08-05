# iron-cow-homelab
# My Homelab: Personal NAS with Docker and a Remote Dev Environment

## 🎯 Project Goal

Build a personal alternative to cloud storage services (Google Drive, iCloud, etc.) using my own hardware, giving me full control over my files, photos, and videos — while also setting up a programming environment accessible from any device on my network.

## 🖥️ Hardware

**NAS:** Iron Cow Zero 1

| Component | Spec |
|---|---|
| CPU | ARM Cortex-A55, Rockchip RK3568 @ 2.0GHz |
| GPU | Mali-G52 |
| NPU | 1 TOPS |
| RAM | 4GB DDR4 |
| System storage | 32GB eMMC |
| Network | 2.5GbE LAN |
| Ports | USB 3.0, USB-C, HDMI |

## 📦 Base NAS Features

Before touching Docker, I set up the NAS's native web portal features:
- File manager with remote access via web and mobile app
- Photo backup and viewing
- Movie/series playback
- File sharing across devices

## 🐳 Docker: Extending the NAS

The NAS ships with Docker built in, which lets me install open-source services beyond the vendor's stock apps. The first service I deployed:

### code-server (VS Code in the browser)

A full development environment (Visual Studio Code) running as a container on the NAS, accessible from any browser on my local network — no local install required on each device.

**Image used:** [`codercom/code-server`](https://hub.docker.com/r/codercom/code-server) (official image, `arm64` architecture)

**Container configuration:**

| Parameter | Value |
|---|---|
| Port | `8080:8080` (TCP) |
| Volume | `[NAS folder]/codesvproject` → `/home/coder/project` (Read/Write) |
| Environment variable | `PASSWORD` (access password) |

Equivalent `docker-compose.yml`:

```yaml
version: "3"
services:
  code-server:
    image: codercom/code-server:latest
    container_name: codeservershadow
    ports:
      - "8080:8080"
    volumes:
      - ./codesvproject:/home/coder/project
    environment:
      - PASSWORD=your_password_here
    restart: unless-stopped
```

**Access:** `http://[NAS-local-IP]:8080` (within the home LAN)

## 📚 What I Learned

- How lightweight virtualization with Docker containers compares to installing software directly on the host system
- Network storage (NAS) management as an alternative to third-party cloud services
- Configuring volumes for data persistence in containers
- The difference between local (LAN) access and remote access, and why a service exposed on a port isn't reachable outside the network without additional setup (reverse proxy or VPN)

## 🔜 Next Steps

- [ ] Explore secure remote access (VPN, e.g. Tailscale) to use code-server outside the home network
- [ ] Add more Docker services (monitoring, password manager, etc.)
- [ ] Document the first code projects built in this environment

---

*Personal homelab project — Luis Suarez*
