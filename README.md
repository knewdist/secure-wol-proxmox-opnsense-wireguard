Secure Wake-on-LAN with Proxmox, OPNsense, and WireGuard
Overview

This project implements a security-first, on-demand access model for a Proxmox server using:

Wake-on-LAN (WoL)

WireGuard VPN

OPNsense as a centralized firewall and control plane

The goal is to reduce idle power usage while ensuring the Proxmox host is not routably accessible or remotely woken without authenticated, encrypted VPN access, while clearly documenting the limits of Layer-2 behavior.

The design emphasizes:

Least-privilege access

Default-deny firewalling

Explicit trust boundaries

No exposed management services

Architecture Summary

Hypervisor: Proxmox VE

Firewall / Router: OPNsense

VPN: WireGuard (terminated on OPNsense)

Client OS: Arch Linux (omarchy)

LAN: 192.168.1.0/24

VPN: 10.10.10.0/24

WireGuard runs on OPNsense (not Proxmox) to ensure a single always-on control plane and avoid dependency on the host being powered on.

Security Model & Properties

This project enforces security controls at clearly defined network layers, with behavior validated through hands-on testing rather than assumptions.

Enforced Controls

No routed Wake-on-LAN from untrusted networks
Directed or routed Wake-on-LAN traffic from non-LAN networks is blocked by firewall policy unless explicitly permitted.

Authentication required for off-LAN Wake-on-LAN
Wake-on-LAN attempts originating outside the local broadcast domain require WireGuard authentication.

Firewall-enforced control at Layer 3
All routed access to Wake-on-LAN and management services is explicitly governed by OPNsense firewall rules.

No direct WAN exposure
No Proxmox management services or Wake-on-LAN mechanisms are exposed to the public internet.

Layer-2 Wake-on-LAN Reality (Documented Finding)

Testing confirmed that Wake-on-LAN does function from wireless VLANs under certain conditions.

This occurs because:

Wake-on-LAN fundamentally operates at Layer 2

Some WoL utilities transmit raw Ethernet broadcast frames

Wireless access points may bridge these frames to the wired LAN

Layer-3 firewalls and VPNs cannot intercept or block Layer-2 broadcasts by design

This behavior is not a misconfiguration, but an inherent property of Ethernet networking.

Mitigation Options

Full enforcement of VPN-only Wake-on-LAN requires one or more of the following:

Access-point–level client or broadcast isolation

Placement of servers in a dedicated server VLAN with no shared broadcast domain

Switch-level enforcement preventing Layer-2 broadcast forwarding

These mitigations are architectural controls that operate below the firewall layer.

Documentation (Phased Implementation)

Detailed implementation notes are broken out by phase:

Phase 1 — Persistent Wake-on-LAN on Proxmox

Phase 2 — Secure Remote Access Architecture (WireGuard)

Phase 3 — WireGuard Peer Identity and Setup

Phase 4 — Firewall Policy Design

Phase 5 — Wake-on-LAN Security Testing and Findings

Each phase documents design decisions, testing outcomes, and security rationale.

## Status

Active homelab project.  
Firewall rules and VPN-only Wake-on-LAN trigger are intentionally introduced incrementally.

### Verified Controls

✔ WireGuard VPN routing verified  
✔ VPN-only HTTPS access to Proxmox confirmed  
✔ Server VLAN isolation enforced  
✔ Wireless and LAN Wake-on-LAN blocked  
✔ End-to-end firewall policy validation complete

Why This Project Matters

This repository demonstrates:

Firewall-first infrastructure design

Secure remote management without exposed services

Practical Zero Trust concepts in a homelab environment

Real-world sysadmin decision-making based on observed behavior, not assumptions

Clear documentation of security boundaries across OSI layers







Actively implemented and documented as part of a security-focused homelab.




