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
- No LAN-based WoL abuse
- No guest or IoT access
- Authentication required
- Firewall-enforced control

---

## Planned Enhancements
- WireGuard-only WoL trigger
- Mobile VPN support
- Power usage comparison (before/after)

