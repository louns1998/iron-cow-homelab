# How to Connect to My Homelab (code-server)

A simple guide for connecting to the development environment remotely and securely.

## For Yourself (any new device)

1. **Install Tailscale**
   Download from [tailscale.com/download](https://tailscale.com/download) for your device (Windows, Mac, Linux, iOS, Android).

2. **Sign in**
   Use the same account every time (`<your-account>`). The new device will appear automatically in your tailnet.

3. **Open code-server**
   Once connected, go to:
   ```
   http://<NAS-local-IP>:8080
   ```
4. **Enter the password** when prompted.

That's it — works the same whether you're on home WiFi, mobile data, or another network.

## Sharing Access with Someone Else

⚠️ Don't just hand out the IP and password to anyone outside your Tailscale network — without being connected to the tailnet, that address isn't reachable at all (which is the whole point of using Tailscale).

To actually let someone else in, use Tailscale's own sharing feature instead of adding them as a full member of your network:

1. Go to `console.tailscale.com/admin/machines`
2. Find the NAS device (the one tagged **Subnets**) → click **⋯** → **Share**
3. Enter the other person's email
4. They'll get an invite link — once accepted, Tailscale installs on their device and they can reach `http://<NAS-local-IP>:8080` just like you do, without seeing your other devices or being able to browse your whole network

This keeps access scoped: they can reach only what you've shared, and you can revoke it anytime from the same admin panel.

## Quick Troubleshooting

| Problem | Likely cause |
|---|---|
| Connection refused | Tailscale app not running/connected, or the `tailscale-nas` container is stopped |
| Times out / loads forever | The subnet route (`<your-LAN-subnet>/24`) isn't approved yet in the admin panel |
| Wrong password prompt won't accept | Double-check the password set in the `code-server` container's environment variables |
