# Mac mini: Remote Access with Tailscale

## 1. What this guide covers

This document explains how to reach your **Mac mini from outside your home or office network** using **Tailscale**. You install Tailscale on the Mac mini and on every device you use to connect (laptop, phone, another Mac). Traffic travels over an encrypted mesh VPN; you do **not** need to open ports on your router, set up dynamic DNS, or configure traditional VPN port forwarding.

**Typical uses after setup:**

- SSH into the Mac mini from anywhere.
- macOS **Screen Sharing** (VNC) over the Tailscale IP.
- Web UIs and APIs you run on the Mac (e.g. `http://100.x.x.x:port`) bound to the right interface (see notes below).

---

## 2. Concepts (short)

| Term | Meaning |
|------|--------|
| **Tailscale** | A VPN mesh: each device gets a stable IP on Tailscale’s `100.x.x.x` range and can reach other devices in your tailnet if policy allows. |
| **Tailnet** | Your private network of devices tied to one Tailscale account (or team). |
| **Tailscale IP** | The `100.x.x.x` address assigned to each machine; use this to connect instead of the LAN IP when you are off-network. |
| **MagicDNS** | Optional: resolve hostnames like `your-mac.tailnet-name.ts.net` instead of memorizing IPs (enabled in the admin console). |

---

## 3. Prerequisites

1. **Tailscale account** — Sign up at [https://tailscale.com](https://tailscale.com) (free tier is enough for personal use).
2. **Admin access** on the Mac mini (install apps / approve system extensions).
3. **Each client device** — Ability to install Tailscale (macOS, Windows, Linux, iOS, Android).

---

## 4. Step-by-step: Mac mini (the machine you want to reach)

### 4.1 Install Tailscale on the Mac mini

**Option A — App Store (simplest for desktop use)**

1. Open the **App Store** on the Mac mini.
2. Search for **Tailscale** and install the official app.
3. Open **Tailscale** from Applications.

**Option B — Homebrew**

1. Install Homebrew if needed: see [https://brew.sh](https://brew.sh).
2. In Terminal on the Mac mini, run:

   ```bash
   brew install --cask tailscale
   ```

3. Open **Tailscale** from Applications (or start it from Spotlight).

**Option C — Installer from Tailscale**

1. Download the macOS package from [https://tailscale.com/download](https://tailscale.com/download).
2. Open the `.pkg` and complete the installer.
3. Launch **Tailscale**.

### 4.2 Sign in and join your tailnet

1. When Tailscale prompts you to **Log in**, choose your provider (Google, Microsoft, GitHub, or email) and complete sign-in in the browser.
2. Approve the device in the browser if asked.
3. macOS may ask for permission to add **VPN configurations** and **network extensions** — allow these so Tailscale can create the virtual interface.

### 4.3 Confirm the Mac mini is online in Tailscale

1. Open [https://login.tailscale.com/admin/machines](https://login.tailscale.com/admin/machines) in a browser (from any device).
2. Find your Mac mini in the device list. Status should show **Connected**.
3. Note the **Tailscale IP** (starts with `100.`). You will use this address when connecting from outside your LAN.

### 4.4 (Recommended) Set a clear machine name

1. In the Tailscale app on the Mac mini, or in the admin console, set a **device name** you will recognize (e.g. `mac-mini-home`).
2. If you enable **MagicDNS** in the admin console (DNS tab), you can later use names like `mac-mini-home.your-tailnet.ts.net` instead of the raw IP.

---

## 5. Step-by-step: Your other devices (phone, laptop, etc.)

Repeat for **each** device that should reach the Mac mini.

### 5.1 Install Tailscale

- **macOS** — Same as section 4.1 (App Store, Homebrew, or download).
- **Windows** — Installer from [https://tailscale.com/download](https://tailscale.com/download); install and open the Tailscale app.
- **Linux** — Follow the Linux instructions on the same download page for your distribution (often `apt`, `dnf`, or an AppImage).
- **iOS / Android** — Install **Tailscale** from the App Store or Google Play.

### 5.2 Sign in with the **same** account

1. Open Tailscale on the client device.
2. Log in using the **same** Tailscale identity you used on the Mac mini.
3. Wait until the app shows **Connected** (or equivalent).

### 5.3 Verify you can see the Mac mini

1. On the client, open [https://login.tailscale.com/admin/machines](https://login.tailscale.com/admin/machines) or use the Tailscale app’s device list.
2. Confirm the Mac mini appears and is **Connected**.
3. From a terminal on another computer (with Tailscale connected), try **ping** the Mac mini’s Tailscale IP:

   ```bash
   ping 100.x.x.x
   ```

   Replace `100.x.x.x` with the IP from the admin console. If ping succeeds, the path is working.

---

## 6. Optional: Use MagicDNS (friendly hostname instead of IP)

If you prefer a readable URL-like name over `100.x.x.x`, enable **MagicDNS** once in your tailnet and then connect using the device hostname.

### 6.1 Enable MagicDNS in admin console

1. Open [https://login.tailscale.com/admin/dns](https://login.tailscale.com/admin/dns).
2. In the **DNS** page, turn on **MagicDNS**.
3. Save/apply changes if prompted.

### 6.2 Confirm your Mac mini hostname

1. Open [https://login.tailscale.com/admin/machines](https://login.tailscale.com/admin/machines).
2. Select your Mac mini device.
3. Note the full MagicDNS name (for example `mac-mini-home.your-tailnet.ts.net`).

### 6.3 Connect using hostname (examples)

- **SSH**
  ```bash
  ssh your_username@mac-mini-home.your-tailnet.ts.net
  ```
- **Screen Sharing (VNC)**
  ```text
  vnc://mac-mini-home.your-tailnet.ts.net
  ```
- **Web service**
  ```text
  http://mac-mini-home.your-tailnet.ts.net:3000
  ```

If hostname does not resolve immediately, disconnect/reconnect Tailscale on the client and try again.

---

## 7. What to connect to (common cases)

Use the **Tailscale IP** (`100.x.x.x`) or **MagicDNS name** when you are off your home network—not the Mac’s LAN address (e.g. `192.168.x.x`).

### 7.1 SSH (Remote Login)

1. On the Mac mini: **System Settings → General → Sharing → Remote Login** — turn **On**, and restrict users if you prefer.
2. From another device with Tailscale connected:

   ```bash
   ssh your_username@100.x.x.x
   ```

   Or with MagicDNS:

   ```bash
   ssh your_username@mac-mini-home.your-tailnet.ts.net
   ```

### 7.2 Screen Sharing (VNC)

1. On the Mac mini: **System Settings → General → Sharing → Screen Sharing** — turn **On**.
2. From another Mac on Tailscale: **Finder → Go → Connect to Server** (`⌘K`), enter:

   ```text
   vnc://100.x.x.x
   ```

   Or use the Screen Sharing app with the same address.

### 7.3 Web services and APIs

- If you run a server on the Mac mini, connect to `http://100.x.x.x:PORT` or `https://100.x.x.x:PORT` from a client on Tailscale.
- The app must **listen** on an interface that accepts those connections (often `0.0.0.0` or all interfaces). Binding only to `127.0.0.1` will **not** be reachable from other Tailscale devices.

### 7.4 macOS firewall

If **Firewall** is enabled (**System Settings → Network → Firewall**), you may need to **allow incoming connections** for **sshd**, **Screen Sharing**, or your specific app. If something works on LAN but not via Tailscale IP, check firewall rules first.

---

## 8. Optional: Subnet routing and exit nodes (brief)

- **Subnet router** — Lets Tailscale clients route to **other** devices on your home LAN (e.g. a NAS that does not run Tailscale). This is advanced; see Tailscale docs for “subnet router” if you need it.
- **Exit node** — Routes **all** internet traffic from a client through the Mac mini (like a traditional VPN exit). Enable only if you understand the implications; configure in the admin console and on the client.

For “I only need to reach my Mac mini,” you usually **do not** need either feature.

---

## 9. Security practices (recommended)

1. **Keep macOS and Tailscale updated** on the Mac mini and clients.
2. **Use strong passwords** for macOS accounts and SSH keys (`ssh-ed25519` keys are a good default).
3. **Restrict SSH** to your user and disable password login in favor of keys if you expose SSH broadly (edit `sshd_config` carefully).
4. In the Tailscale **admin console**, review **Access Controls (ACLs)** if you use a team or want to limit which devices can talk to the Mac mini.
5. **Do not** share your Tailscale login with untrusted people; anyone in the tailnet can reach machines according to your ACLs.

---

## 10. Troubleshooting

| Problem | What to try |
|--------|----------------|
| Client cannot ping Mac mini’s `100.x` address | Ensure both show **Connected** in the admin console; restart Tailscale on both sides; check Mac mini is not sleeping with networking suspended (adjust Energy settings if needed). |
| Works on Wi-Fi at home but not elsewhere | You should use the **Tailscale IP**, not `192.168.x.x`; confirm the client has Tailscale **connected** when off-network. |
| SSH refused | Enable **Remote Login** on the Mac mini; check firewall; use `ssh user@100.x.x.x`. |
| Web page does not load | Confirm the service listens on `0.0.0.0` or the Tailscale interface, not only `localhost`; check port and firewall. |
| “Needs key” or auth errors | Re-authenticate Tailscale on the device; for SSH, verify username and keys. |

Official help: [https://tailscale.com/kb](https://tailscale.com/kb).

---

## 11. Quick checklist

- [ ] Tailscale installed and signed in on the **Mac mini**.
- [ ] Tailscale installed and signed in on **each client** (same account).
- [ ] Mac mini shows **Connected** in [admin/machines](https://login.tailscale.com/admin/machines).
- [ ] **Tailscale IP** (`100.x.x.x`) noted for the Mac mini.
- [ ] (Optional) **MagicDNS** enabled in [admin/dns](https://login.tailscale.com/admin/dns) and hostname tested.
- [ ] **Remote Login** / **Screen Sharing** (or your services) enabled as needed.
- [ ] Test from a client: `ping` and `ssh` (or browser) to the Tailscale IP.

After this, you can connect to your Mac mini from coffee shops, mobile data, or another country without configuring your home router for inbound access.
