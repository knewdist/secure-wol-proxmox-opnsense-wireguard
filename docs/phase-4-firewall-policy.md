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

## Validation and Debugging (Lessons Learned)

Objective

Validate VPN-only management access to the Proxmox server by enforcing strict firewall policy across WireGuard and the SERVERS VLAN, while documenting real-world misconfigurations and how they were identified and corrected.

This phase intentionally avoided “allow any” shortcuts to preserve a Zero Trust security model.

Environment Context

Client: Arch Linux (omarchy)

VPN: WireGuard (wg-home)

Firewall: OPNsense

Server VLAN: 192.168.30.0/24

VPN Network: 10.10.10.0/24

Proxmox Host: 192.168.30.203

Management Port: TCP 8006

What Was Expected

Proxmox Web UI (:8006) reachable only when:

WireGuard VPN is connected

Firewall rules explicitly permit access

No access from:

LAN

Wireless VLANs

Unauthenticated hosts

What Actually Happened (Symptoms)

After WireGuard was configured and firewall rules were added:

WireGuard handshake succeeded

Routing appeared correct

HTTPS to Proxmox hung indefinitely

ping to the SERVERS gateway sometimes worked

nc to port 8006 timed out

This initially suggested a firewall or routing issue.

Root Causes (There Were Two)
1. WireGuard Interface Was Not Brought Up

The primary failure was simple but subtle:

The VPN interface (wg-home) was not up on the client.

Without running:

sudo wg-quick up wg-home


No routes to 192.168.30.0/24 existed

Traffic never entered the tunnel

Firewall debugging was misleading because packets never arrived

Once the interface was brought up, routes were installed automatically:

ip route add 192.168.30.0/24 dev wg-home


This immediately resolved connectivity.

Lesson:
A correct WireGuard configuration is useless if the interface is down. Always verify routes, not just configuration files.

2. Firewall Rule Placement and Interface Direction

Before the VPN interface issue was identified, several firewall mistakes were uncovered and corrected:

Common Misconceptions Corrected

Allowing traffic on the WG_HOME interface alone is insufficient

Each interface enforces policy independently

Traffic must be explicitly allowed on the interface it enters

Correct Traffic Flow
VPN Client (10.10.10.2)
  → WG_HOME (WireGuard interface)
  → OPNsense routing
  → SERVERS interface
  → Proxmox (192.168.30.203:8006)


This required explicit rules on both interfaces.

Final Working Firewall Rules
WG_HOME Interface

Allow VPN clients to reach:

SERVERS gateway (192.168.30.1)

Proxmox management port (192.168.30.203:8006)

Purpose:

Permit routing and service access

Nothing else

SERVERS Interface

Allow incoming TCP 8006 from WG_HOME net

Default deny for all other sources

Purpose:

Enforce VPN-only management

Prevent LAN / wireless access even if routing exists

Verification Commands Used

These commands were critical in isolating each failure:

wg show wg-home


Confirmed handshake and peer state.

ip route get 192.168.30.203


Confirmed traffic was using wg-home.

nc -vz 192.168.30.203 8006


Confirmed TCP connectivity to Proxmox.

ss -tlnp | grep 8006


Confirmed Proxmox was listening on all interfaces.

Key Lessons Learned
1. VPN Connectivity ≠ VPN Usability

A WireGuard handshake does not guarantee routing or access. Routes must exist, and the interface must be up.

2. Firewall Rules Are Interface-Scoped

Rules must exist on every interface a packet traverses, not just the VPN interface.

3. “Hanging” Connections Usually Mean Silent Drops

TCP hangs (not rejects) almost always indicate firewall policy, not service failure.

4. Logs Tell the Truth

OPNsense Live View logs were essential in proving where packets stopped.

Security Outcome

After corrections:

✅ Proxmox management is VPN-only

✅ No LAN or wireless access path exists

✅ VLAN isolation is enforced

✅ No exposed management services

✅ Zero Trust principles upheld

### Administration Anti-Lockout Rule

By default, OPNsense injects an automatic anti-lockout rule that allows
administrative access from the LAN interface regardless of firewall rules
or service listening interfaces.

This rule was explicitly disabled to enforce VPN-only administrative access.

Behavior verified:
- VPN down → Web GUI inaccessible
- VPN up → Web GUI accessible over WG_HOME only

This confirms proper enforcement of remote-only administration.

### Configuration Backups and Recovery

Before disabling the OPNsense administration anti-lockout rule and enforcing
VPN-only management access, full configuration backups were exported.

Two snapshots are maintained:

- `opnsense-pre-vpn-admin-lockdown.xml`
  Baseline configuration prior to management-plane hardening

- `opnsense-vpn-only-admin-working.xml`
  Current hardened configuration with VPN-only GUI and SSH access

This ensures immediate recovery via console or VM access if required.

