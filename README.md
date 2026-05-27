# OPNsense + Active Directory Home Lab

A virtualized home lab simulating a small office network: an **OPNsense firewall**, a **Windows Server 2022 Active Directory domain controller**, and a **Windows 10/11 client** joined to the domain — all running in VirtualBox.

Built as part of my hands-on networking practice while pursuing my Master's in Computer Engineering for IoT Systems at Hochschule Nordhausen.

> 🚧 **Status:** Work in progress. This README will be updated with full configuration steps, screenshots, and test results as the lab is completed.

## Goal

Design and configure a virtual office network where:

- A dedicated **firewall (OPNsense)** controls all traffic between the simulated internet (WAN) and the internal LAN.
- An internal **Active Directory domain** (`corp.local`) provides centralized authentication, DNS, and DHCP services.
- A **domain-joined client workstation** can log in with domain credentials and access internal resources.
- Firewall rules enforce that the LAN can reach the internet, but the internet cannot initiate connections into the LAN.

## Topology

```
   [Host Machine — running VirtualBox]
                  |
   ┌──────────────┴──────────────┐
   │  Virtual Network "WAN-Net"  │  ── NAT to host's internet
   └──────────────┬──────────────┘
                  |
            [OPNsense VM]
            WAN: NAT
            LAN: 192.168.50.1/24
                  |
   ┌──────────────┴──────────────┐
   │ Virtual Network "LAN-Net"  │
   │      192.168.50.0/24       │
   └──┬──────────────────────┬──┘
      │                      │
[Windows Server 2022]   [Windows Client]
192.168.50.10           DHCP from server
DC + AD + DNS + DHCP    Joined to corp.local
```

## Components

| Role | OS | Hostname | IP |
|---|---|---|---|
| Firewall / Router | OPNsense | `opnsense-fw` | WAN: NAT · LAN: 192.168.50.1 |
| Domain Controller | Windows Server 2022 | `DC01` | 192.168.50.10 |
| Client Workstation | Windows 10 / 11 | `CLIENT01` | DHCP |

## Planned configuration

### OPNsense
- WAN interface using VirtualBox NAT (provides upstream internet)
- LAN interface on internal network with IP 192.168.50.1/24
- Firewall rules: LAN → WAN allowed, WAN → LAN blocked by default
- (DHCP disabled on OPNsense — handled by the Windows DC instead)

### Windows Server 2022 (DC01)
- Static IP 192.168.50.10, gateway 192.168.50.1, DNS pointing to itself
- Active Directory Domain Services role installed
- New forest created: `corp.local`
- DNS service running on the DC
- DHCP role installed, scope `192.168.50.100 – 192.168.50.200`, options pushing DC as DNS and OPNsense as gateway

### Windows Client (CLIENT01)
- Configured for DHCP
- Joined to `corp.local` domain
- Login tested with a domain user account

## Verification (planned)

| Test | Expected |
|---|---|
| OPNsense web UI reachable from LAN | ✅ |
| Client receives DHCP lease from DC01 | ✅ |
| Client resolves `corp.local` via DC01 DNS | ✅ |
| Client successfully joins the domain | ✅ |
| Domain user can log in on the client | ✅ |
| Client can ping a public host (e.g., 1.1.1.1) | ✅ |
| Public host cannot initiate connection into LAN | ✅ |

## Skills demonstrated (when complete)

`OPNsense` · `Firewall Configuration` · `Active Directory` · `Windows Server` · `DNS` · `DHCP` · `VirtualBox` · `Network Design` · `Domain Join` · `Network Segmentation`

---

**Author:** Wasif Gull · Master's student in Computer Engineering for IoT Systems @ HS Nordhausen
[LinkedIn](https://www.linkedin.com/in/wasif-gull-257124403/) · [Project 1: Multi-VLAN LAN in Packet Tracer](https://github.com/WasifGull/multi-vlan-lan-packet-tracer)
