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



