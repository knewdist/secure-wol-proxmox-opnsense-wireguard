Secure Wake-on-LAN with Proxmox, OPNsense, and WireGuard

Incremental, security-first implementation

Overview
Secure Remote Management Homelab

A secure home lab environment with Proxmox, OPNsense firewall segmentation, and WireGuard VPN for controlled remote access and Wake-on-LAN workflows.

🔍 Overview

This project demonstrates a secure remote access and systems management setup using virtualization and network security tools. It was built as a learning platform to gain real-world experience in Linux systems, firewalls, networking, and secure remote administration — skills relevant for IT support, systems, and networking roles.

🛠 Key Features

Proxmox virtualization for hosting multiple VMs and services.

OPNsense firewall for secure network segmentation and traffic control.

WireGuard VPN to enable secure remote access to internal services.

Secure Wake-on-LAN (WoL) configurations that are only available through authenticated VPN sessions.

Designed with best practices for secure access and least-privilege network controls. 
GitHub
+1

🧠 What You’ll Learn

This project showcases hands-on experience with:

Linux systems and command-line tools

VPN setup and key-based authentication

Firewall rule creation and traffic segmentation

Virtual machine deployment and networking

Secure remote administration techniques

📦 Tech Stack

Proxmox VE – Virtualization platform

OPNsense – Firewall & router platform

WireGuard – VPN solution for secure remote access

Bash/Linux CLI – Scripting and system troubleshooting

VLAN and secure routing concepts

🚀 How It Works

Proxmox hosts virtual machines and provides virtualization resources.

OPNsense manages network segments (LAN, IoT, VPN) and firewall rules.

WireGuard provides a secure VPN tunnel to connect remotely.

Once connected, authorized users can send Wake-on-LAN packets to power systems on the internal network securely.





Environment

Hypervisor

Proxmox VE

Firewall / Router

OPNsense

Client OS

Arch Linux (host: omarchy)

Network Model

LAN: 192.168.1.0/24

WireGuard VPN: 10.10.10.0/24

Proxmox Host NIC

Physical interface: eno1

Phase 1 — Persistent Wake-on-LAN on Proxmox
Problem

Wake-on-LAN settings configured via ethtool do not persist across reboots by default.

This creates a silent failure mode:

WoL works initially

Stops working after reboot, update, or power loss

No obvious indication unless tested

Solution

Configure WoL at the NIC driver level and enforce persistence using systemd.

Key Steps

Identify the correct physical interface

eno1 (not vmbr0, which is virtual)

Verify WoL capability

ethtool eno1


Confirmed:

Supports Wake-on: umbg
Wake-on: g


Create a persistent systemd service

[Unit]
Description=Enable Wake-on-LAN on eno1
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/ethtool -s eno1 wol g

[Install]
WantedBy=multi-user.target


Enable and test

systemctl enable wol-eno1.service
systemctl start wol-eno1.service


Reboot to confirm persistence

Result

Wake-on-LAN survives reboots

Proxmox host can remain powered off when idle

Reduced power consumption and hardware wear

No dependency on Proxmox services being online

Phase 2 — Secure Remote Access Design

WireGuard via OPNsense

Design Goal

Ensure the Proxmox host can only be:

Accessed after authentication

Reached through an encrypted tunnel

Woken without exposing LAN services or ports

Key Architectural Decision

WireGuard runs on OPNsense — not on the Proxmox host.

This guarantees:

A single, always-on control plane

Clean trust boundaries

No reliance on the Proxmox server being powered on

Centralized firewall policy enforcement

WireGuard Architecture (OPNsense-Specific)

OPNsense intentionally separates WireGuard into two layers:

1. WireGuard Instance (Tunnel Layer)

Terminates encrypted tunnels

Owns the VPN IP

Defines the VPN subnet

Configured as:

Tunnel address: 10.10.10.1/24

This layer handles cryptography and routing only.

2. Assigned Interface (Firewall Policy Layer)

Has no IP address

Exists solely for firewall rules

Acts as a policy boundary

This separation prevents:

Accidental routing leaks

Implicit trust

VPN logic being mixed with firewall logic

Peer Identity Model
Key-Based Authentication

WireGuard uses public/private key pairs, similar to SSH:

Private key

Generated on the client

Never leaves the device

Public key

Registered on OPNsense

Used to identify the peer

There are:

No passwords

No shared secrets

No interactive authentication

Access is based purely on cryptographic identity.

Client Peer Configuration

Client

Hostname: omarchy

OS: Arch Linux

Tunnel Identity

10.10.10.2/32

Why /32?

Each peer receives exactly one IP:

Prevents IP spoofing

Enables precise firewall rules

Scales cleanly as peers are added

Makes intent explicit

Placeholder Peer (Implementation Detail)

OPNsense only creates the wg interface after at least one peer exists.

To satisfy this safely:

A placeholder peer was created

Assigned a non-routable /32

No endpoint

No routing

No access

This allowed:

Interface creation

Continued configuration

Zero exposure risk

Security Principles Applied

Default deny — no traffic allowed unless explicitly permitted

No exposed services — zero WAN ports opened

Authentication before access — VPN required for reachability

Clear trust boundaries — firewall controls all tunnel traffic

Immediate kill-switch — WireGuard can be disabled instantly in OPNsense

Current State
Component	Status
Proxmox WoL	Persistent and verified
WireGuard plugin	Installed and enabled
WireGuard instance	Configured
Tunnel IP	10.10.10.1/24
Client peer	omarchy (10.10.10.2/32)
Firewall rules	None (deny all)
WAN exposure	None

At this stage, the VPN exists but cannot pass traffic by design.

Next Steps (Planned)

Generate WireGuard client config on Arch Linux

Bring tunnel up and verify handshake only

Add minimal firewall rules (least privilege)

Implement VPN-only Wake-on-LAN trigger

Document power usage before vs after

Why This Matters

This project demonstrates:

Practical Zero-Trust thinking

Firewall-first design

Secure remote management

Real-world sysadmin decision-making

It avoids:

Exposed management ports

Always-on infrastructure

Implicit trust models

And it scales cleanly as the lab grows.

## Status

Actively implemented and documented as part of a security-focused homelab.



