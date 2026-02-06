# Shadow-Net

**Whispers in the dark. No traces. No masters.**

Shadow-Net is a lightweight, **ephemeral encrypted mesh communication layer** built on top of the [Yggdrasil](https://yggdrasil-network.github.io/) overlay network. It enables secure, peer-to-peer messaging and file transfer with **zero persistent storage**, **manual web-of-trust**, and **automatic self-destruction** — ideal for privacy-critical scenarios like protests, off-grid groups, or high-risk environments.

Everything runs in RAM (`/dev/shm`), keys expire quickly, logs/metadata are memory-only, and a single wipe command (or reboot) leaves no forensic traces.

## Features

- **End-to-end encrypted** (Yggdrasil transport + GPG application layer)
- **RAM-resident only** — no disk writes, auto-cleanup on exit/timer
- **Manual trust via GPG fingerprints** (handshake protocol)
- **UDP-based** (port 9999) for low-footprint messaging
- **Dual CLI + CGI/web support** — send messages/files via terminal or simple web portal
- **Inbox viewer** — JSON export of decrypted messages + metadata (timestamp, sender FP, trust status)
- **Nuclear wipe** — instant purge of keys, logs, caches, and listener process
- **Self-destruct mode** — optional timer-based shutdown

**Warning**: This is alpha-stage software (v0.1–1.0 scaffold). Use on trusted hardware only. UDP is lossy (no built-in retries/ACKs). Peer discovery is basic/local-only — not plug-and-play on the global Yggdrasil mesh.

## Requirements

- Linux (tested on Debian/Ubuntu derivatives)
- [Yggdrasil](https://github.com/yggdrasil-network/yggdrasil-go) installed and running (provides the encrypted IPv6 mesh)
- Standard tools: `gpg`, `netcat` (`nc`), `ping6`, `jq` (optional for JSON), `screen`/`at` (for background/timer)

## Installation & Quick Start

### 1. Install Yggdrasil (one-time):
```bash
curl -s https://ygg.sh | bash   # or follow official docs
systemctl enable --now yggdrasil
```

### 2. Clone / Download Shadow-Net:
```bash
git clone https://github.com/vanj900/shadow-net.git
cd shadow-net
chmod +x *.sh
```

### 3. Launch (basic mode):
```bash
./shadow-launcher.sh
```
- This bootstraps: checks Yggdrasil, generates temp GPG key, starts listener, scans peers, enables self-destruct trap.
- Output ends with: `[*] [SHADOWNET] Shadow-Net active. Shadows can now speak in silence.`

### 4. Handshake with a peer (one-time per contact):
```bash
./shadow-utils.sh handshake fd00::abcd   # replace with real peer IPv6
```
- Sends your fingerprint → waits for their pubkey → imports & caches trust.

### 5. Send a message:
```bash
./shadow-api.sh send fd00::abcd "Meet at dusk. Bring the keys."
```

### 6. View inbox (CLI or web polling):
```bash
./shadow-inbox.sh
```

### 7. Wipe everything (when done):
```bash
./shadow-wipe.sh
```
or just reboot — traces vanish.

## Project Structure

```
shadow-net/
├── shadow-launcher.sh      # Bootstrap: key gen, listener, discovery, self-destruct
├── shadow-utils.sh         # Peer scan + manual handshake + list trusted peers
├── shadow-core.sh          # Encrypted send/listen loop (trust-enforced)
├── shadow-api.sh           # CLI/CGI bridge for sending messages/files
├── shadow-inbox.sh         # JSON exporter for inbox logs + metadata
├── shadow-wipe.sh          # Instant purge of all RAM artifacts
├── README.md               # This file
├── LICENSE                 # MIT License
└── .gitignore              # Git ignore rules
```

(Optional: Add `shadow-portal/` folder with `index.html` + JS to poll `/shadow-inbox.sh` as a simple web UI.)

## How It Works (High-Level)

1. Yggdrasil provides encrypted IPv6 overlay routing (E2EE transport).
2. Launcher creates 1-day ephemeral GPG key in RAM.
3. Peers are discovered (basic local ping) or manually added.
4. **Handshake** exchanges pubkeys via UDP → trust cached.
5. **Send**: GPG-encrypt → UDP send to trusted peer only.
6. **Listen**: Decrypt inbound → log plaintext + JSON metadata (sender FP, trust, timestamp) to RAM.
7. **Wipe**: Kills listener + rm -rf keys/logs/caches.

## Security & Privacy Notes

- **Strengths** — No central servers, no persistent identity, double encryption, RAM-only, manual trust.
- **Limitations** — UDP lossy (messages can drop), discovery is local-only (fd00:: range), manual key exchange required.
- **Improvements needed** — Replace ping discovery with `yggdrasilctl getPeers`, add retries/ACKs, QR onboarding.
- **Threat model** — Designed against device seizure, network surveillance, centralized backdoors. Not bulletproof against advanced forensics or compromised peers.

## Legal & Ethical Disclaimer

Shadow-Net is a privacy & communication tool. Use responsibly and legally. The authors are not responsible for misuse. Do not use to facilitate illegal activities.

## Contributing / Next Steps

- Fix global-mesh discovery (`yggdrasilctl` integration)
- Add QR code key/onboarding generator
- Build simple web portal (HTML/JS polling inbox JSON)
- Package as single tar.gz or Raspberry Pi image

Pull requests welcome. Shadows grow stronger together.

## Version & License

**Version**: 0.1 (Rebranded Alpha)  
**License**: MIT (Open Source)  
**Author**: O1 (original scaffold), vanj900 (current maintainer)  
**Project Home**: https://github.com/vanj900/shadow-net

---

*Stay dark. Speak free.*