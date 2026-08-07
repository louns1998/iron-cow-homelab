# Setting Up code-server on the Iron Cow Zero 1 NAS

This document walks through the full process of deploying a browser-based VS Code environment (`code-server`) on my Iron Cow Zero 1 NAS using Docker, including the issues I ran into and how I solved them.

## 1. Pulling the Image

Searched the NAS's built-in Docker image registry for `code-server` and selected the official image:

- **Image:** [`codercom/code-server`](https://hub.docker.com/r/codercom/code-server)
- **Tag:** `latest`
- **Architecture:** `arm64` (matches the NAS's Rockchip RK3568 CPU)

## 2. Creating the Container

Configured through the NAS's Docker UI:

| Setting | Value |
|---|---|
| Port mapping | `8080:8080` (TCP) |
| Volume | `[NAS storage]/codesvproject` → `/home/coder/project` (Read/Write) |
| Environment variable | `PASSWORD=<my password>` |

Container name: `codeservershadow`

## 3. Accessing code-server

Once running, accessed it from a browser on the same local network at:

```
http://<NAS-local-IP>:8080
```

**Note:** this only works on the same LAN. The NAS's web portal doesn't expose a reverse proxy for custom Docker ports, so remote access outside the home network isn't available without extra setup (e.g. a VPN like Tailscale — a possible next step).

## 4. Troubleshooting: Permission Denied

Trying to create a file inside `/home/coder/project` failed with:

```
Unable to write file 'vscode-remote://.../hello.py'
(NoPermissions (FileSystemError): Error: EACCES: permission denied)
```

**Diagnosis** — ran inside the container's integrated terminal:

```bash
whoami
id
ls -la /home/coder/
```

This showed the container runs as user `coder` (uid=1000), but the mounted `project` folder was owned by `root`:

```
drwxr-xr-x 2 root root    6 Aug  5 19:06 project
```

**Fix:**

```bash
sudo chown -R coder:coder /home/coder/project
```

After this, file creation worked normally.

## 5. Installing Python

The base image doesn't ship with Python. Installed it manually:

```bash
sudo apt update
sudo apt install python3
```

**Note:** anything installed with `apt` lives inside the container's filesystem, not on the NAS. If the container is ever deleted and recreated, Python would need to be reinstalled. Only the `/home/coder/project` folder is persisted on the NAS via the mounted volume.

## 6. Writing and Running the First Script

Created `hello.py` inside `~/project`:

```python
print("Hello, this is my first script on my own server")

name = "Luis"
print(f"Hello {name}, welcome to your homelab")
```

Ran it:

```bash
cd project
python3 hello.py
```

## 7. Version Control: Connecting to GitHub

Set up Git inside the `project` folder and pushed the first commit to the existing `iron-cow-homelab` repository.

**Issue 1 — Git initialized in the wrong directory.** Running `git init` from `~` (home) instead of `~/project` caused Git to track dozens of unrelated internal code-server system files. Fixed by removing the misplaced repo and reinitializing in the correct folder:

```bash
rm -rf ~/.git
cd ~/project
git init
```

**Issue 2 — Missing Git identity.** First commit attempt failed until configuring:

```bash
git config --global user.name "<your name>"
git config --global user.email "<my email>"
```

**Issue 3 — Diverged histories.** The remote repo already had commits (`README.md`, `docker-compose.yml`) unrelated to the local repo's history, so a direct push was rejected. Resolved with:

```bash
git config pull.rebase false
git pull origin main --allow-unrelated-histories
# resolved the merge commit message in the terminal editor (nano)
git push -u origin main
```

Result: `hello.py` merged successfully into the `iron-cow-homelab` repository alongside the existing README and Docker Compose reference file.

## Key Takeaways

- Docker volumes don't automatically match container user permissions — ownership often needs to be fixed manually with `chown`.
- Packages installed inside a container are ephemeral unless baked into a custom image; only mounted volumes persist.
- `git init` must be run in the exact folder meant to be version-controlled — running it too high in the directory tree pulls in unrelated files.
- Merging unrelated Git histories (local vs. remote repos created independently) requires `--allow-unrelated-histories` and a reconciliation strategy (`merge` vs `rebase`).
