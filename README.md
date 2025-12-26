# Secure Wake-on-LAN with Proxmox, OPNsense, and WireGuard

## Overview

This project implements a **security-first, on-demand access model** for a Proxmox server using:

- Wake-on-LAN (WoL)
- WireGuard VPN
- OPNsense as a centralized firewall and control plane

The goal is to reduce idle power usage while ensuring the Proxmox host can **only be woken and accessed after authenticated, encrypted VPN access**.

The design follows:
- Least-privilege access
- Default-deny firewalling
- Clear trust boundaries
- No exposed management services

---

## Architecture Summary

- **Hypervisor:** Proxmox VE
- **Firewall / Router:** OPNsense
- **VPN:** WireGuard (terminated on OPNsense)
- **Client OS:** Arch Linux (`omarchy`)
- **LAN:** `192.168.1.0/24`
- **VPN:** `10.10.10.0/24`

WireGuard runs on OPNsense (not Proxmox) to ensure a single always-on control plane and avoid dependency on the host being powered on.

---

## Documentation (Phased Implementation)

Detailed implementation notes are broken out by phase:

- [Phase 1 — Persistent Wake-on-LAN on Proxmox](docs/phase-1-persistent-wol.md)
- [Phase 2 — Secure Remote Access Architecture (WireGuard)](docs/phase-2-wireguard-architecture.md)
- [Phase 3 — WireGuard Peer Identity and Setup](docs/phase-3-wireguard-peer-setup.md)
- [Phase 4 — Firewall Policy Design](docs/phase-4-firewall-policy.md)
- [Phase 5 — VPN-Only Wake-on-LAN (Planned)](docs/phase-5-vpn-only-wol.md)

Each phase documents design decisions, security rationale, and implementation details.

---

## Status

Active homelab project.  
Firewall rules and VPN-only Wake-on-LAN trigger are intentionally introduced incrementally.

---

## Why This Project Matters

This repository demonstrates:
- Firewall-first infrastructure design
- Secure remote management without exposed services
- Practical Zero Trust concepts in a homelab environment
- Real-world sysadmin decision-making and documentation







Actively implemented and documented as part of a security-focused homelab.




