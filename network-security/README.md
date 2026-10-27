[Home](../README.md)

# Network security

The goal of this area is to take the Windows domain lab and defend it like a small company network: put a real firewall in front of it, split it into security zones, attack it from a Kali machine, and then lock it down based on what the attacks showed. I document each milestone with what I built, what broke, and what I learned.

## Goal

1. Segment the network into SERVERS, USERS and ADMIN zones behind an OPNsense firewall.
2. Attack the unfiltered network from Kali to see what it exposes, as a baseline.
3. Apply default-deny firewall rules with explicit Active Directory allow rules, and harden the domain.
4. Add one free, open-source detection system.
5. Repeat the attacks and compare before and after, including privilege-escalation attempts.
6. Apply Zero Trust principles — least privilege per user/app, no implicit trust between
   zones.

## Milestones

| Milestone | What it covers | Status |
|---|---|---|
| [M1: Segmented network](milestones/M1-segmented-network.md) | OPNsense firewall, three zones, routing, DHCP relay, three failures and fixes | Done |
| [M2: Baseline](milestones/M2-baseline-attacks.md) | Attack the network with allow-all rules and record what succeeds | Done |
| [M3: Network hardening](milestones/M3-hardening.md) | Default-deny, Active Directory allow rules, management interface from ADMIN only, GPO hardening | Done |
| [M4: Account and audit hardening](milestones/M4-ad-hardening.md) | Lockout policy, admin separation, NTLM audit-only, detection-oriented audit logging | Done |
| M5: Detection | One open-source detection system | Planned |
| M6: Attack and defend | Repeat the M2 attacks plus privilege escalation, then compare before and after | Planned |
| M7: Zero Trust | Least privilege per user and application, no implicit trust between zones | Planned |
| M8: Packaging | Final README, diagram and lessons learned | Planned |

Planned milestones are not started. They will be linked here when they have content.

---

[← Home](../README.md)
