# Phase 2 — Secure Remote Access Architecture (WireGuard)

## Objective
Design a secure remote access model where the Proxmox host can only be accessed or woken after authenticated VPN access.

---

## Key Decision
WireGuard runs on **OPNsense**, not on the Proxmox host.

This ensures:
- A single always-on control plane
- No dependency on the server being powered on
- Clear trust boundaries

---

## Network Design

| Network | CIDR |
|---|---|
| LAN | 192.168.1.0/24 |
| WireGuard VPN | 10.10.10.0/24 |

Gateway convention is preserved:
- LAN gateway: `192.168.1.1`
- VPN gateway: `10.10.10.1`

---

## OPNsense WireGuard Model

OPNsense separates WireGuard into two layers:

### WireGuard Instance (Tunnel Layer)
- Terminates the encrypted tunnel
- Owns the tunnel IP
- Defines the WireGuard network

Configured with:
- `10.10.10.1/24`

### Assigned Interface (Firewall Policy Layer)
- Has no IP address
- Exists solely for firewall rule application
- Acts as a policy boundary

---

## Security Rationale
- Default deny behavior
- No WAN exposure
- Firewall rules explicitly control access
- VPN can be disabled instantly

