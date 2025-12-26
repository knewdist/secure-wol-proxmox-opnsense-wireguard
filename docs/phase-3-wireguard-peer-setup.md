# Phase 3 — WireGuard Peer Identity and Setup

## Objective
Register client identities securely using WireGuard public/private key pairs.

---

## Authentication Model
WireGuard uses key-based authentication:

- Private key: generated on the client, never shared
- Public key: registered on OPNsense

Authentication is based on possession of the private key.

---

## Client Environment
- OS: Arch Linux
- Client name: `omarchy`

---

## Key Generation (Client Side)

```bash
wg genkey | tee ~/wg-home.private | wg pubkey > ~/wg-home.public
chmod 600 ~/wg-home.private

---

Peer Registration:
Public key is pasted into OPNsense
Assigned VPN identity: 10.10.10.2/32

Why /32
Assigns exactly one IP to the peer
Prevents spoofing
Enables precise firewall control


