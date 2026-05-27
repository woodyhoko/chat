# NightConnect P2P

A **serverless peer-to-peer chat app** that runs entirely in your browser. No account, no server, no message storage — just two peers connecting directly via WebRTC.

**[▶ Live Demo](https://woodyhoko.github.io/chat)**

---

## How It Works

1. Open the app — you're assigned a **3-character peer ID** (e.g. `A4Z`)
2. Share your ID with the person you want to chat with
3. They enter your ID and hit **Connect**
4. Messages travel peer-to-peer via WebRTC data channels — nothing passes through a server

```
You (Browser A)  ──────── WebRTC ────────  Friend (Browser B)
     A4Z                                        Q9M
```

---

## Features

- 🔒 **Zero server** — messages never touch a backend
- 🌙 **Dark / Light theme** toggle
- 📱 **Responsive** — sidebar layout on desktop, single-column on mobile
- ⚡ **Instant** — connection established in seconds via PeerJS signaling

---

## Stack

| Layer | Tech |
|---|---|
| P2P Transport | [PeerJS](https://peerjs.com/) (WebRTC wrapper) |
| Styling | Vanilla CSS with CSS custom properties |
| Build | None — single self-contained HTML file |

---

## Run Locally

```bash
# No build needed — just open:
open index.html
```

> **Note:** Some browsers block WebRTC on `file://` URLs. Serve with `python -m http.server 8000` if the connection fails.

