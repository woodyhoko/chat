# NightConnect — Serverless P2P Chat

A **peer-to-peer browser chat** with zero server involvement in message delivery. Two peers connect directly via WebRTC; messages never touch a backend, are never stored, and disappear when the tab closes.

**[▶ Live Demo](https://woodyhoko.github.io/chat)**

---

## 1. Architecture

```
Browser A (peer A4Z)                         Browser B (peer Q9M)
     │                                              │
     │    WebRTC DataChannel (DTLS-SRTP)            │
     └──────────────────────────────────────────────┘
           ↑
     PeerJS signaling server
     (used only to exchange SDP offer/answer + ICE candidates)
     ← connection setup only; zero message traffic →
```

The only time a server is involved is during the **WebRTC handshake** (ICE negotiation). Once the `RTCDataChannel` is open, all message traffic flows peer-to-peer via DTLS-encrypted UDP/TCP. PeerJS abstracts the SDP and ICE exchange into a simple API.

---

## 2. WebRTC data channel internals

WebRTC `RTCDataChannel` provides a SCTP (Stream Control Transmission Protocol) channel tunnelled over DTLS 1.2:

```javascript
const peer = new Peer();                          // generates 3-char ID via PeerJS

// Initiating side
const conn = peer.connect(remoteId, {
  reliable: true,                                 // ordered SCTP delivery
  serialization: 'json'
});
conn.on('open', () => conn.send({ text: '...' }));

// Receiving side
peer.on('connection', conn => {
  conn.on('data', msg => renderMessage(msg));
});
```

Key properties of the channel:
- **Ordered delivery** (`reliable: true`) — SCTP retransmits lost packets, preserving message order
- **DTLS encryption** — all data is encrypted in transit; the signaling server never sees message content
- **NAT traversal** — PeerJS uses STUN (Google's public servers) for NAT hole-punching; for symmetric NAT, a TURN relay would be needed (not included)

---

## 3. Peer ID and discovery

On load, `new Peer()` connects to PeerJS's cloud signaling server, which assigns a random 3-character alphanumeric ID. The user shares this ID out-of-band (copy/paste, voice). The other peer calls `peer.connect(id)`, triggering the SDP handshake via the signaling server. No phone book, no account, no history.

---

## 4. Features

| Feature | Implementation |
|---|---|
| Zero-server messaging | WebRTC `RTCDataChannel` (SCTP over DTLS) |
| Dark / Light theme | CSS custom properties toggled by JS; persisted to `localStorage` |
| Responsive layout | CSS Grid — sidebar + message column on desktop, single column on mobile |
| Message timestamps | `Date.toLocaleTimeString()` appended client-side |
| Connection status | PeerJS `open` / `close` / `error` events update the UI |

---

## 5. Privacy properties

- Messages are **never written to any server** — they travel directly peer-to-peer
- The PeerJS signaling server sees only connection metadata (peer IDs, IP addresses during ICE), not message content
- **No persistence** — refreshing the tab destroys the connection and all messages
- **No account** — no email, no login, no tracking cookie

---

## 6. Limitations & production considerations

- **Single peer per session** — the app opens one `RTCDataChannel` to one remote peer; multi-party chat would require a mesh or SFU architecture
- **STUN only** — symmetric NAT (common on corporate networks) blocks peer-to-peer; add a TURN server for reliability
- **No end-to-end verification** — the peer ID is not cryptographically authenticated; a MITM could impersonate a peer at the signaling level
- **PeerJS cloud dependency** — signaling uses PeerJS's public server (rate-limited); replace with a self-hosted `peerjs-server` for production

---

## 7. Stack

| Layer | Technology |
|---|---|
| P2P transport | [PeerJS](https://peerjs.com/) (WebRTC `RTCDataChannel` + STUN/ICE) |
| Styling | Vanilla CSS with CSS custom properties |
| Deployment | Single self-contained `index.html` |

---

## 8. Run locally

```bash
# WebRTC on file:// is blocked in some browsers; use a local server:
python -m http.server 8000
# open http://localhost:8000
# Open in two tabs and connect using the displayed peer ID
```
