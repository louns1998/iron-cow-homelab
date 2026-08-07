# Secure Remote Access with Tailscale

This document covers how I set up secure remote access to my homelab using Tailscale, so I can reach `code-server` (and the rest of the NAS's local network) from anywhere — without exposing any ports to the public internet.

## Why Tailscale Instead of Port Forwarding

Opening port 8080 directly on the router would make `code-server` reachable by anyone on the internet — a common attack surface (password guessing, exploiting container vulnerabilities, etc.).

Tailscale instead creates a private encrypted network (a "tailnet") between my own devices. Nothing is exposed publicly; only devices logged into my account can reach each other, over an end-to-end encrypted tunnel. It's free for personal use (up to 100 devices).

## 1. Creating a Tailscale Account

Signed up at [tailscale.com](https://tailscale.com) using GitHub login.

## 2. Connecting the First Personal Device

Installed the Tailscale client on my home desktop and signed in with the same account. It immediately appeared in the admin panel (`console.tailscale.com/admin/machines`) as a connected device with a private `100.x.x.x` address.

## 3. Deploying Tailscale on the NAS

Pulled the official `tailscale/tailscale` image from the NAS's Docker registry (the `library/Tailscale` listing failed to load tags, so I used the vendor's own namespace instead).

**Container configuration:**

| Setting | Value |
|---|---|
| Privileges | **Run container with high privileges** (required — Tailscale needs to create a virtual network interface) |
| Environment variable | `TS_AUTHKEY=<auth key>` |
| Ports | None needed — Tailscale operates at the network level, not through a mapped port |

### Generating the Auth Key

Went to `console.tailscale.com/admin/settings/keys` → **Generate auth key**, with:
- **Reusable:** enabled (so the same key can be reused if the container is recreated)
- **Ephemeral:** disabled (the NAS is a permanent device, not a temporary one)
- **Expiration:** 90 days (doesn't affect the device connection once authenticated, only the key's own validity window)

This key authenticates the container automatically on startup, without requiring an interactive login inside the NAS.

## 4. Troubleshooting: Container Exiting Unexpectedly

After downloading the Tailscale image, `code-server`'s container briefly exited on its own (state: `Exited`). Restarted it and it stayed stable — likely a temporary resource spike on the NAS's 4GB of RAM while the new image was being pulled, not a persistent issue.

## 5. Enabling Access to the Whole LAN (Subnet Routing)

Initially, connecting to the Tailscale IP directly (`<tailscale-node-IP>:8080`) failed with `ERR_CONNECTION_REFUSED`. The reason: the Tailscale container only routes traffic to *itself* by default — it has no knowledge of other services (like `code-server`) running in separate containers on the same NAS.

**Fix — advertise the NAS's local subnet:**

Added a second environment variable to the Tailscale container:

```
TS_EXTRA_ARGS=--advertise-routes=<your-LAN-subnet>/24
```

This tells Tailscale: "the entire local subnet, including my own local IP, is reachable through me."

After restarting the container, a new device appeared in the admin panel tagged **"Subnets"**.

**Approving the route:** Tailscale doesn't activate advertised subnet routes automatically for security reasons. Had to manually approve it:

1. `console.tailscale.com/admin/machines`
2. Found the NAS device → **⋯** → **Edit route settings**
3. Approved the `<your-LAN-subnet>/24` route

## 6. Accessing code-server Remotely

Once the route was approved, connecting from any Tailscale-connected device — including from outside the home network — works by using the **same local IP** as before, not a special Tailscale address:

```
http://<NAS-local-IP>:8080
```

Tailscale transparently tunnels this traffic back to the home network through the encrypted connection.

## 7. Renaming Devices

Both the NAS Tailscale node and personal devices can be renamed in the admin panel (`console.tailscale.com/admin/machines` → **⋯** → **Edit machine name**, disabling "Auto-generate from OS hostname") for easier identification — purely cosmetic, doesn't affect routing or security.

## How the Two Security Layers Compare

| Service | How it's secured | Exposed to the internet? |
|---|---|---|
| NAS web portal / mobile app | Vendor-managed relay + username/password login | Yes, by design (vendor's infrastructure) |
| code-server (port 8080) | LAN-only, or via Tailscale's encrypted tunnel + subnet routing | No, never directly |

## Key Takeaways

- A VPN mesh (Tailscale) is a safer alternative to opening router ports for accessing self-hosted services remotely.
- A Tailscale node only routes to itself unless explicitly configured with subnet routing (`--advertise-routes`), which also requires manual approval in the admin console.
- Once a subnet is advertised and approved, devices on the tailnet reach services using their normal LAN IP — no need to memorize separate Tailscale addresses.
- Containers can be left running permanently (`restart: unless-stopped`); they don't need to be manually stopped/started for day-to-day use.
