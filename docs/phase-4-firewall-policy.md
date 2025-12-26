# Phase 4 — Firewall Policy Design

## Objective
Ensure zero trust by default for VPN traffic.

---

## Default State
- WireGuard interface exists
- No firewall rules defined
- All traffic blocked by default

This ensures:
- No accidental access
- No lateral movement
- Explicit allow-only behavior

---

## Firewall Interface Role
The assigned WireGuard interface acts as a policy boundary:

- No IP ownership
- No routing logic
- Firewall rules determine access

Traffic must explicitly pass through this interface to reach internal networks.

---

## Design Principle
Access is granted incrementally and intentionally.
Nothing is allowed by default.

## Initial Firewall Rule (VPN → Firewall Only)

The first allow rule permits ICMP traffic from the WireGuard VPN subnet
to the firewall itself.

Source: WG_HOME address (10.10.10.0/24)  
Destination: This Firewall  
Protocol: ICMP  

This confirms tunnel functionality while maintaining a default-deny posture
for all other traffic.


