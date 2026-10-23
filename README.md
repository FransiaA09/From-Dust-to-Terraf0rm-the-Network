<div align="center">

# From Dust to Terraf0rm the Network

**A firewall in front of an Active Directory network, split into security zones — then attacked and defended, one milestone at a time.**

![Stack](https://img.shields.io/badge/stack-OPNsense_+_Active_Directory_+_Kali-1f6feb)
![Environment](https://img.shields.io/badge/environment-isolated_VirtualBox_lab-6f42c1)
![Progress](https://img.shields.io/badge/progress-2_of_8_milestones-2da44e)
![Updated](https://img.shields.io/badge/updated-2026--10--23-lightgrey)

</div>

---

> [!NOTE]
> This is a build log, not a tutorial. I build each stage myself, break it on purpose, and write down what worked, what didn't, how I fixed it, and what I learned — mistakes included.

## The idea in one line

**Build → Break → Fix → Detect → Re-test → Zero Trust**

Stand up a realistic, segmented enterprise network from scratch, attack it from a baseline to see what it exposes, then harden and monitor it — proving each change actually worked, before and after.

## At a glance

|  |  |
|---|---|
| **Stack** | OPNsense · Active Directory (Windows Server 2022) · Windows 10 · Kali Linux · Oracle VirtualBox |
| **Shape** | One firewall, three internal zones — SERVERS / USERS / ADMIN — and a WAN where the attacker sits |
| **Progress** | `██████░░░░░░░░░░░░░░░░░░░░` &nbsp; 2 / 8 milestones done |
| **Format** | One write-up per milestone: what was built, a test matrix, failures and fixes, and a "what I learned" |
| **Scope** | Everything runs isolated in VirtualBox — every test targets machines inside this lab only |

## Milestones

|  | Milestone | What it covers | Status |
|---|---|---|---|
| **M1** | [Segmented network](network-security/milestones/M1-segmented-network.md) | OPNsense firewall, three zones, routing, DHCP relay | ✅ **Done** |
| **M2** | [Baseline attack pass](network-security/milestones/M2-baseline-attacks.md) | Attack the allow-all network and record what succeeds | ✅ **Done** |
| **M3** | Firewall policy | Default-deny + explicit Active Directory allow rules; management GUI from ADMIN only | 🔜 **Next** |
| **M4** | AD / DC hardening | Password & lockout policy, admin-account separation, legacy protocols off, audit logging | ⬜ Planned |
| **M5** | Detection | One free, open-source detection system | ⬜ Planned |
| **M6** | Attack & defend | Repeat the M2 attacks plus privilege escalation, then compare before and after | ⬜ Planned |
| **M7** | Zero Trust | Least privilege per user and application, no implicit trust between zones | ⬜ Planned |
| **M8** | Packaging | Final README, diagram, and lessons learned | ⬜ Planned |

*Planned milestones aren't started yet — each gets linked here once it has content.*

## The lab today

*Last updated: 2026/10/23*

![Milestone 1 network topology: an OPNsense firewall connecting a WAN with a Kali machine to three zones: SERVERS (domain controller), USERS (employee workstation) and ADMIN (technician workstation)](network-security/images/m1-topology.png)

<details>
<summary><strong>How it fits together</strong></summary>

<br>

An OPNsense firewall sits between three internal zones (SERVERS, USERS, ADMIN) and a WAN where a Kali machine sits for attack scenarios. The domain controller — Active Directory, DNS, DHCP — lives in SERVERS. Workstations in USERS and ADMIN get their addresses from it through a DHCP relay on the firewall (OPNsense's own DHCP server is off). The lab reuses an earlier Windows domain lab of mine as the SERVERS zone.

In **M1** the rules between zones are allow-all; **M3** replaces that with default-deny plus explicit allow rules.

| Zone | Interface | Subnet | Host |
|---|---|---|---|
| **WAN** | em0 (extnet) | 10.0.99.0/24 | Kali — attacker, outside scenarios |
| **SERVERS** | em1 | 10.10.10.0/24 | `win2k22-DC01` — AD DS · DNS · DHCP · `lab.local` |
| **USERS** | em2 | 10.10.20.0/24 | `win10-client01` — employee workstation, domain-joined |
| **ADMIN** | em3 | 10.10.30.0/24 | `win10-tech01` — technician workstation, domain-joined |

</details>

## Scope & safety

> [!IMPORTANT]
> Everything runs inside VirtualBox, isolated from my home network. Every attack and defense exercise targets only machines inside this lab.
