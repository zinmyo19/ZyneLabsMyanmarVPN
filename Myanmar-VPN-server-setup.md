# ZyneLabs Myanmar VPN — Server Setup Guide 🇲🇲

The app is a **client only** — without a server there is nothing to connect to.
To get through strict firewalls with deep packet inspection (DPI), use
**VLESS + REALITY** (plain WireGuard/OpenVPN gets fingerprinted and blocked).

## 1. Rent a VPS (Singapore — closest to Myanmar)

- ~$5/month (RackNerd, HostHatch, DigitalOcean, Vultr...)
- Ubuntu 22.04/24.04
- ⚠️ Needs a card — if you don't have one, ask a friend/relative abroad

## 2. Install Xray (on the VPS, as root)

```bash
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

Generate the keys:

```bash
xray x25519   # privateKey + publicKey (keep the publicKey)
xray uuid     # one UUID
openssl rand -hex 4  # shortId (e.g. a1b2c3d4)
```

## 3. Write the config — `/usr/local/etc/xray/config.json`

Replace `UUID`, `PRIVATE_KEY`, `SHORT_ID` with your own values:

```json
{
  "log": { "loglevel": "warning" },
  "inbounds": [{
    "port": 443,
    "protocol": "vless",
    "settings": {
      "clients": [{ "id": "UUID", "flow": "xtls-rprx-vision" }],
      "decryption": "none"
    },
    "streamSettings": {
      "network": "tcp",
      "security": "reality",
      "realitySettings": {
        "show": false,
        "dest": "www.microsoft.com:443",
        "xver": 0,
        "serverNames": ["www.microsoft.com"],
        "privateKey": "PRIVATE_KEY",
        "shortIds": ["SHORT_ID"]
      }
    },
    "sniffing": { "enabled": true, "destOverride": ["http", "tls", "quic"] }
  }],
  "outbounds": [
    { "protocol": "freedom", "tag": "direct" },
    { "protocol": "blackhole", "tag": "block" }
  ]
}
```

```bash
systemctl restart xray && systemctl enable xray
# firewall: open 443/tcp
ufw allow 443/tcp 2>/dev/null; iptables -I INPUT -p tcp --dport 443 -j ACCEPT 2>/dev/null
```

## 4. Generate the client link (to put on the family phone)

```
vless://UUID@SERVER_IP:443?encryption=none&flow=xtls-rprx-vision&security=reality&sni=www.microsoft.com&fp=chrome&pbk=PUBLIC_KEY&sid=SHORT_ID&type=tcp#ZyneLabs-MM
```

- `SERVER_IP` = your VPS IP, `PUBLIC_KEY` = publicKey from x25519
- Turn this link into a QR code (qrencode / online QR generator) and send it home
- In the app: **+ → Import from QR code** → scan → connect 🟢

## 5. Tips for clear voice calls

- Keep the server in Singapore (lowest ping)
- If Messenger calls are bad: in app settings → routing, voice-call domains must go
  through the VPN (they are blocked otherwise)
- Don't put more than 3–5 people on one server

## 6. What you can try right now (before the server is ready)

**WARP** inside the ZyneLabs Gaming VPN app — Cloudflare IPs are hard to block
completely, so it sometimes works. Connect WARP on the family phone and try
Messenger. If it fails, go the VLESS route above.
