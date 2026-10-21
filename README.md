# Network Security Home Lab

A home lab where I put a firewall in front of an Active Directory network, split it into security zones, and then attack and defend it. I document every milestone: what I built, what broke, how I fixed it, and what I learned. I'm aiming for a junior network security role, so the focus is hands-on practice rather than theory.

## The lab today

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
| M2: Baseline | Attack the network with allow-all rules and record what succeeds | Planned |
| M3: Firewall policy | Default-deny, Active Directory allow rules, management interface from ADMIN only | Planned |
| M4: AD/DC hardening | Password and lockout policy, admin account separation, legacy protocols off, audit logging | Planned |
| M5: Detection | One open-source detection system | Planned |
| M6: Attack and defend | Repeat the M2 attacks plus privilege escalation, then compare before and after | Planned |
| M7: Zero Trust | Least privilege per user and application, no implicit trust between zones | Planned |
| M8: Packaging | Final README, diagram and lessons learned | Planned |

Planned milestones are not started. They will be linked here when they have content.

## Tools

OPNsense, Active Directory (Windows Server 2022), Windows 10, Kali Linux, Oracle VirtualBox.

## Scope

Everything runs inside VirtualBox, isolated from my home network. All attack and defense testing targets only machines inside this lab.
