# Phase 1 — Persistent Wake-on-LAN on Proxmox

## Objective
Ensure Wake-on-LAN (WoL) on the Proxmox host persists across reboots and power cycles.

By default, WoL settings applied via `ethtool` are not persistent and may reset after reboot or driver reload.

---

## Environment
- Proxmox VE
- Physical NIC: `eno1`
- Virtual bridge: `vmbr0` (not used for WoL)

---

## Key Concept
Wake-on-LAN operates at the **physical NIC level**, not at the virtual bridge level.

The physical interface must remain powered and listen for magic packets *before* Proxmox or virtual networking is available.

---

## Verification

```bash
ip link
ethtool eno1

Confirmed:
Supports Wake-on: umbg
Wake-on: g

---

Persistent via systemd
Create a oneshot systemd service to reapply WoL at boot:

[Unit]
Description=Enable Wake-on-LAN on eno1
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/ethtool -s eno1 wol g

[Install]
WantedBy=multi-user.target

---

Enable and test:
systemctl enable wol-eno1.service
reboot

---

Result:
WoL survives reboot
Proxmox can remain powered off when idle
Reliable, repeatable behavior

