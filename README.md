# My Homelab: Personal NAS with Docker, Remote Dev Environment & Secure Remote Access

## 🎯 Project Goal

Build a personal alternative to cloud storage services (Google Drive, iCloud, etc.) using my own hardware, giving me full control over my files, photos, and videos — while also setting up a programming environment accessible securely from anywhere.

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

The NAS ships with Docker built in, letting me install open-source services beyond the vendor's stock apps.

### code-server (VS Code in the browser)

A full development environment (Visual Studio Code) running as a container on the NAS, accessible from the browser — no local install required.

**Image:** [`codercom/code-server`](https://hub.docker.com/r/codercom/code-server) (official, `arm64`)

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

📄 Full setup log, including the permission and Git issues I ran into and how I fixed them: **[docs/code-server-setup.md](docs/code-server-setup.md)**

### Tailscale (secure remote access)

Instead of opening router ports to reach `code-server` from outside the house, I deployed Tailscale on the NAS to create a private encrypted VPN mesh between my devices — nothing exposed to the public internet.

With subnet routing configured, any device on my tailnet can reach the NAS's local network (`<your-LAN-subnet>/24`) securely from anywhere, using the same local IP as always.

📄 Full setup log, including the subnet routing configuration and troubleshooting: **[docs/tailscale-setup.md](docs/tailscale-setup.md)**

📄 Simple connect/share guide: **[how-to-connect.md](how-to-connect.md)**

## 💻 First Script

Wrote and ran my first script inside code-server, connected to the NAS's persistent storage via a mounted volume:

```python
print("Hello, this is my first script on my own server")

name = "Luis"
print(f"Hello {name}, welcome to your homelab")
```

See [`hello.py`](hello.py).

## 📚 What I Learned

- How lightweight virtualization with Docker containers compares to installing software directly on the host system
- Network storage (NAS) management as an alternative to third-party cloud services
- Configuring volumes for data persistence and fixing container/host permission mismatches (`chown`)
- Setting up Git correctly inside a project folder, resolving diverged histories, and protecting commit author privacy (GitHub's noreply email)
- Building a secure remote-access setup with a VPN mesh (Tailscale) instead of exposing ports directly to the internet, including subnet routing and route approval

## ✅ Project Status

- [x] NAS configured (files, photos, media, remote access via vendor portal)
- [x] Docker deployed with a self-hosted dev environment (`code-server`)
- [x] First script written, run, and version-controlled with Git
- [x] Project documented and pushed to GitHub
- [x] Secure remote access configured (Tailscale + subnet routing)

## 🔜 Next Steps

- [ ] Add more Docker services (monitoring, password manager, etc.)
- [ ] Build and document a slightly more advanced coding project
- [ ] Explore Tailscale ACLs for more granular access control when sharing

---

*Personal homelab project — < Shadow >*
