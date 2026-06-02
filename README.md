# OPNsense + Active Directory Home Lab

A virtualized home lab simulating a small office network: an **OPNsense firewall**, a **Windows Server 2022 Active Directory domain controller**, and a **Windows Server client** joined to the domain — all running in VirtualBox.

Built as part of my hands-on networking practice while pursuing my Master's in Computer Engineering for IoT Systems at Hochschule Nordhausen.

## What this lab demonstrates

- Designing a segmented network with a dedicated firewall between the simulated WAN and an internal LAN
- Building an Active Directory forest from scratch and creating a new domain (`crop.local`)
- Configuring authoritative DNS for the internal domain
- Configuring DHCP with scope options that push the correct gateway and DNS to clients
- Joining a Windows machine to the domain and authenticating as a domain user
- Diagnosing and resolving DNS/firewall issues that prevent domain join

## Topology

```
   [Host Machine — running VirtualBox]
                  |
   ┌──────────────┴──────────────┐
   │  VirtualBox NAT (simulated  │  ── upstream internet
   │  WAN for OPNsense)          │
   └──────────────┬──────────────┘
                  |
            [OPNsense VM]
            WAN: 10.0.2.x (DHCP from VirtualBox NAT)
            LAN: 192.168.50.1/24
                  |
   ┌──────────────┴──────────────┐
   │ VirtualBox NAT Network       │
   │      'LAN-Net'              │
   │      192.168.50.0/24        │
   └──┬──────────────────────┬──┘
      │                      │
[Windows Server 2022]   [Windows Server 2022]
   DC01                   CLIENT01
   192.168.50.10          192.168.50.100 (DHCP)
   AD DS · DNS · DHCP     Joined to crop.local
```

## Components

| Role | OS | Hostname | IP |
|---|---|---|---|
| Firewall / Router | OPNsense 26.1 | `opnsense-fw` | WAN: NAT · LAN: 192.168.50.1 |
| Domain Controller | Windows Server 2022 (Eval) | `DC01` | 192.168.50.10 (static) |
| Client | Windows Server 2022 (Eval) | `CLIENT01` | 192.168.50.100 (DHCP) |

VirtualBox network: a NAT Network named `LAN-Net` (`192.168.50.0/24`, VirtualBox DHCP disabled — DHCP handled by DC01).

## OPNsense configuration

- WAN interface (`em0`) on VirtualBox NAT, gets its address via DHCP from the host
- LAN interface (`em1`) statically set to `192.168.50.1/24`
- DHCP server on LAN is **disabled** — DHCP is handled by DC01 in Active Directory
- Web GUI accessible from LAN at `https://192.168.50.1`

## Domain Controller configuration (DC01)

- Static IP `192.168.50.10/24`, gateway `192.168.50.1`, DNS pointing to itself (`127.0.0.1`)
- Roles installed: **Active Directory Domain Services**, **DNS Server**, **DHCP Server**
- New forest created: **`crop.local`**
- DHCP scope: `192.168.50.100 – 192.168.50.200`
  - Router option: `192.168.50.1` (OPNsense)
  - DNS server option: `192.168.50.10` (the DC itself)
  - Parent domain: `crop.local`

## Domain Join (CLIENT01)

- Fresh Windows Server 2022 install on LAN-Net
- Received DHCP lease from DC01: `192.168.50.100/24`, gateway `192.168.50.1`, DNS `192.168.50.10`
- Renamed to `CLIENT01` and joined to the `crop.local` domain
- Confirmed `whoami` returns `crop\administrator` after domain login

## Verification

| Test | Result |
|---|---|
| OPNsense LAN interface responds to ping from DC01 | ✅ |
| DC01 promoted to Domain Controller with healthy AD DS + DNS | ✅ |
| DHCP scope active and authorized in AD | ✅ |
| CLIENT01 receives DHCP lease automatically from DC01 | ✅ |
| CLIENT01 can `nslookup crop.local` → resolves to 192.168.50.10 | ✅ |
| CLIENT01 successfully joins the `crop.local` domain | ✅ |
| Logging in as `crop\Administrator` on CLIENT01 succeeds | ✅ |
| `whoami` on CLIENT01 returns `crop\administrator` | ✅ |

## Troubleshooting notes

The lab surfaced a couple of real-world issues worth recording:

- **CLIENT01 initially failed DHCP**, getting an APIPA (`169.254.x.x`) address. Resolved by running `ipconfig /release` and `ipconfig /renew`, which forced a fresh DHCP discovery once all three VMs were running.
- **First domain join attempt failed** with *"An Active Directory Domain Controller for the domain crop.local could not be contacted."* The cause was the Windows Defender Firewall on DC01 blocking inbound DNS queries on UDP/53 — `nslookup crop.local` from CLIENT01 timed out even though ICMP (ping) worked. Resolved by disabling the firewall on DC01 for the lab (production fix would be to add explicit allow rules for DNS instead).

## Skills demonstrated

`OPNsense` · `Firewall Configuration` · `Active Directory Domain Services` · `Windows Server 2022` · `DNS` · `DHCP` · `Domain Join` · `VirtualBox` · `Network Design` · `Troubleshooting`

---

**Author:** Wasif Gull · Master's student in Computer Engineering for IoT Systems @ HS Nordhausen
[LinkedIn](https://www.linkedin.com/in/wasif-gull-257124403/) · [Project 1: Multi-VLAN LAN in Packet Tracer](https://github.com/WasifGull/multi-vlan-lan-packet-tracer)
