# Phase 5 — VPN-Only Wake-on-LAN (Planned)

## Objective
Allow the Proxmox host to be woken **only** after VPN authentication.

---

## Final Flow

Client (VPN authenticated)  
↓  
OPNsense firewall  
↓  
Wake-on-LAN packet  
↓  
Proxmox host powers on  

--- 

## Security Properties

- No routed LAN-based Wake-on-LAN abuse
- No guest or IoT VLAN access to Wake-on-LAN
- Authentication required for off-LAN Wake-on-LAN
- Firewall-enforced control at Layer 3
- Layer-2 behavior documented with mitigation options
Security controls are enforced at Layer 3 and above; Layer-2 broadcast behavior
is addressed through documented architectural mitigations rather than firewall rules.

## Wireless Layer-2 Limitation

Testing confirmed that raw Ethernet Wake-on-LAN frames can traverse the
wireless infrastructure when access-point-level broadcast isolation is
unavailable. Because these frames operate at Layer 2, they bypass
Layer-3 routing, firewall rules, and VPN controls.

As a result, Wake-on-LAN packets originating from wireless clients may
reach the Proxmox NIC directly via bridged broadcast forwarding, even
when VPN authentication is not present.

Full enforcement of VPN-only Wake-on-LAN requires one of the following:
- Access-point-level client or broadcast isolation, or
- Placement of servers in a dedicated VLAN with no shared broadcast domain

This behavior is inherent to Ethernet broadcast handling and is not a
firewall or WireGuard misconfiguration.


---

## Planned Enhancements
- WireGuard-only WoL trigger
- Mobile VPN support
- Power usage comparison (before/after)

