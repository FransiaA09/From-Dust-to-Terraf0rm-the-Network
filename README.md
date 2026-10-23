# From Dust to Terraf0rm the Network

A home lab where I put a firewall in front of an Active Directory network, split it into security zones, and then attack and defend it. I build it myself, break it on purpose, and document every milestone honestly: what worked, what didn't, how I fixed it, and what I learned along the way.

Everything here runs in an isolated virtual network I built myself — see [Scope](#scope) below.

## At a glance

| | |
|---|---|
| **Stack** | OPNsense, Active Directory (Windows Server 2022), Windows 10, Kali Linux, Oracle VirtualBox |
| **Progress** | Milestone 2 of 8 done (segmentation, baseline attack pass) |
| **Format** | One write-up per milestone: what was built, a test matrix, failures and fixes, and what I learned |

## The lab today

*Last updated: 2026/10/23*

![Lab topology: an OPNsense firewall connecting a WAN with a Kali machine to three zones: SERVERS (domain controller), USERS (employee workstation) and ADMIN (technician workstation)](network-security/images/m1-topology.png)

An OPNsense firewall sits between three internal zones (SERVERS, USERS, ADMIN) and a WAN where a Kali machine sits for attack scenarios. The domain controller (Active Directory, DNS, DHCP) is in SERVERS. Workstations in USERS and ADMIN get their addresses from it through a DHCP relay on the firewall. The lab builds on an earlier Windows domain lab of mine, reused as the SERVERS zone.

## Goal

1. Segment the network into SERVERS, USERS and ADMIN zones behind an OPNsense firewall.
2. Attack the unfiltered network from Kali to see what it exposes, as a baseline.
3. Apply default-deny firewall rules with explicit Active Directory allow rules, and harden the domain.
4. Add one free, open-source detection system.
5. Repeat the attacks and compare before and after, including privilege-escalation attempts.
6. Stretch goal: apply Zero Trust principles.

## Milestones

| Milestone | What it covers | Status |
|---|---|---|
| [M1: Segmented network](network-security/milestones/M1-segmented-network.md) | OPNsense firewall, three zones, routing, DHCP relay, three failures and fixes | Done |
| [M2: Baseline](network-security/milestones/M2-baseline-attacks.md) | Attack the network with allow-all rules and record what succeeds | Done |
| M3: Firewall policy | Default-deny, Active Directory allow rules, management interface from ADMIN only | Planned |
| M4: AD/DC hardening | Password and lockout policy, admin account separation, legacy protocols off, audit logging | Planned |
| M5: Detection | One open-source detection system | Planned |
| M6: Attack and defend | Repeat the M2 attacks plus privilege escalation, then compare before and after | Planned |
| M7: Zero Trust | Least privilege per user and application, no implicit trust between zones | Planned |
| M8: Packaging | Final README, diagram and lessons learned | Planned |

Planned milestones are not started. They will be linked here when they have content.

## Scope

Everything runs inside VirtualBox, isolated from my home network. All attack and defense testing targets only machines inside this lab.
