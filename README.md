#  Tony's Home Tunnel

# HomeTunnel — Complete Setup & Usage Guide

A reverse HTTPS proxy tunnel that lets you browse the web through
your home connection from work or your phone — no router changes,
no static IP, no paid services.

---

## How It Works

```
[Work PC / Phone]  →  HTTPS  →  [Cloudflare Edge]
                                       ↓ tunnel
                              [Your Mac at Home]
                                       ↓
                              [Python HTTP Proxy]
                                       ↓
                              [The Open Internet]
```

- Your Mac runs a local HTTP proxy (port 8888)
- Cloudflare Tunnel punches a hole out through your firewall
  and gives you a free `*.trycloudflare.com` HTTPS URL
- You point your work browser / phone proxy settings at that URL
- All your traffic routes through your home internet

No port-forwarding needed. No static IP needed. Free.

---

## One-Time Setup on Your Home Mac

### 1. Download the HomeTunnel folder
Put the `HomeTunnel` folder anywhere convenient — Desktop is fine.

### 2. Make scripts executable
Open Terminal and run:
```bash


chmod +x ~/Desktop/HomeTunnel/HomeTunnel.command
```

### 3. Allow the script in macOS Security
The first time you double-click `HomeTunnel.command`, macOS will
warn you. Do this:
- Go to **System Settings → Privacy & Security**
- Scroll down and click **"Open Anyway"** next to the blocked file
- Click **Open** in the dialog

### 4. Let Homebrew install (first run only)
If you don't have Homebrew or `cloudflared`, the script installs
them automatically. This takes ~2 minutes on first run.

---

## Running the Tunnel

1. Double-click **HomeTunnel.command** in Finder
   (or run `bash home_tunnel.sh` in Terminal)

2. Wait ~10 seconds for the green box to appear:

```
╔══════════════════════════════════════════════════════╗
║  ✅  TUNNEL IS LIVE!                                 ║
╠══════════════════════════════════════════════════════╣
║  Proxy URL:                                          ║
║  https://sunny-bird-1234.trycloudflare.com           ║
╚══════════════════════════════════════════════════════╝
```

3. **Copy the URL** — you'll need the hostname part
   (e.g. `sunny-bird-1234.trycloudflare.com`)

4. Leave the Terminal window open. Closing it stops the tunnel.

> ⚠️ The URL changes every time you restart the script.
> Bookmark the hostname or paste it into a notes app.

---

## Connecting from Your Work Computer

### macOS (work Mac)
1. **Apple menu → System Settings → Network**
2. Click your active connection (Wi-Fi or Ethernet) → **Details**
3. Click the **Proxies** tab
4. Tick **Secure Web Proxy (HTTPS)**
5. Server: `sunny-bird-1234.trycloudflare.com`  Port: `443`
6. Click **OK** → **Apply**
7. Open any browser — your traffic now exits through home

To turn off: uncheck **Secure Web Proxy** and Apply.

### Windows 10/11 (work PC)
1. **Start → Settings → Network & Internet → Proxy**
2. Under **Manual proxy setup** click **Set up**
3. Toggle **Use a proxy server** → ON
4. Address: `sunny-bird-1234.trycloudflare.com`  Port: `443`
5. Click **Save**

To turn off: toggle **Use a proxy server** → OFF.

### Chrome / Firefox (browser-only, no system change)
**Chrome:**
- Install the **Proxy SwitchyOmega** extension
- New profile → Protocol: HTTPS
- Server: `sunny-bird-1234.trycloudflare.com`, Port: `443`
- Switch to that profile

**Firefox:**
- Settings → General → scroll to **Network Settings** → Settings
- Select **Manual proxy configuration**
- HTTPS Proxy: `sunny-bird-1234.trycloudflare.com`  Port: `443`
- Tick **Also use this proxy for HTTPS**
- OK

---

## Connecting from Your iPhone

1. Open **Settings → Wi-Fi**
2. Tap the **(i)** next to your current Wi-Fi network
3. Scroll to **HTTP Proxy** → tap **Configure Proxy**
4. Choose **Manual**
5. Server: `sunny-bird-1234.trycloudflare.com`
6. Port: `443`
7. Authentication: OFF (leave blank)
8. Tap **Save** (top right)

Open Safari — all traffic now goes through your home connection.

To turn off: go back and select **Off**.

> ✅ This works on any Wi-Fi network, including your work Wi-Fi.
> It also works on mobile data (4G/5G) if your carrier allows
> custom proxy ports (most do).

---

## Connecting from Your Android Phone

1. **Settings → Wi-Fi**
2. Long-press your network → **Modify network** (or tap the gear icon)
3. Expand **Advanced options**
4. Proxy: **Manual**
5. Proxy hostname: `sunny-bird-1234.trycloudflare.com`
6. Proxy port: `443`
7. Tap **Save**

---

## Do You Need to Change Your Home Router Settings?

**No.** This is the beauty of Cloudflare Tunnel — it makes an
*outbound* connection from your Mac to Cloudflare's servers, so
your router firewall doesn't need any inbound rules or
port-forwarding.

**You do NOT need to:**
- Forward any ports on your router
- Set up a static IP or DDNS
- Open any firewall rules

**The only requirement:** your Mac must be on and the
HomeTunnel.command window must be open and running.

---

## Optional: Keep Your Mac Awake

macOS may sleep and kill the tunnel. To prevent this:

```bash
# Prevent sleep while tunnel is running (auto-stops when you Ctrl+C)
caffeinate -dims &
```

Or in **System Settings → Battery (or Energy Saver)**:
- Prevent Mac from sleeping automatically: **ON**
- Wake for network access: **ON**

---

## Optional: Use a Permanent URL (Free Cloudflare Account)

The free quick-tunnel URL changes on every restart. If you want
a permanent subdomain (e.g. `home-proxy.yourdomain.com`):

1. Sign up free at https://dash.cloudflare.com
2. Add your domain to Cloudflare (or use their free subdomain)
3. Run: `cloudflared tunnel login`
4. Run: `cloudflared tunnel create home-proxy`
5. Edit `home_tunnel.sh` — replace the cloudflared line with:
   ```bash
   cloudflared tunnel run --url http://127.0.0.1:$PROXY_PORT home-proxy
   ```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Script says "port in use" | Run `PROXY_PORT=9999 bash home_tunnel.sh` |
| No URL after 30 sec | Check internet connection; try again |
| Proxy works but slow | Normal — traffic routes via Cloudflare US servers |
| Site says "proxy error" | The site may block proxy traffic (try a VPN instead) |
| iPhone proxy not working | Ensure port is 443, not 8888 |
| Work blocks trycloudflare.com | Use a permanent named tunnel (see above) |

**Full logs:** `~/Library/Logs/HomeTunnel/tunnel.log`

---

## Security Notes

- The tunnel is open to anyone who knows the URL. Don't share it
  publicly. The random URL provides obscurity by default.
- For extra security, add HTTP Basic Auth by running:
  `cloudflared tunnel --credentials-file ... --http-host-header ...`
- Traffic between you and Cloudflare is encrypted (HTTPS/TLS 1.3)
- Traffic from Cloudflare to your Mac is inside the secure tunnel

---

*HomeTunnel uses Cloudflare Tunnel (free tier) and a Python
stdlib HTTP proxy — no third-party Python packages required.*
