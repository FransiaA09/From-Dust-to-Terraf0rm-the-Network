[Home](../README.md)

# Windows domain

A Windows domain in VirtualBox: Active Directory, DHCP, DNS, Group Policy, file permissions, and delegated administration. I worked through tickets from a technician's seat. The scenarios are numbered in the order I did them (1 to 7) and grouped by topic below.

Background: [Build log](build-log.md) · [Build issues](build-issues.md) · [Lab infrastructure](../docs/infrastructure.md)

## Scenarios by topic

Each scenario page starts with a short summary (what was reported, the root cause, the fix) and ends with screenshots.

### Domain services (DHCP and DNS)

| # | Scenario | What the user reported | Root cause | Main tools |
|---|---|---|---|---|
| 1 | [DHCP scope deactivated](scenarios/01-dhcp-scope-deactivated.md) | "I can't connect to anything on the network." | DHCP scope disabled on the DC; the client fell back to an APIPA address | `ipconfig`, `Get-NetAdapter`, `Get-DhcpServerv4Scope` |
| 4 | [Broken DNS](scenarios/04-broken-dns.md) | "Nothing is loading." | Client DNS manually set to a server that does not exist | `ping`, `nslookup`, `ipconfig` |

### Accounts and sign-in

| # | Scenario | What the user reported | Root cause | Main tools |
|---|---|---|---|---|
| 2 | [Logon-hours restriction](scenarios/02-logon-hours-restriction.md) | "It's telling me something about time restrictions." | The account's Logon Hours denied every hour | Event Viewer (4625) |
| 3 | [Account lockout](scenarios/03-account-lockout.md) | "Now it says my account is locked." | Three failed sign-ins reached the lockout threshold | Event Viewer on the DC (4740), Group Policy |

### Permissions and access

| # | Scenario | What the user reported | Root cause | Main tools |
|---|---|---|---|---|
| 5 | [Share access denied](scenarios/05-share-access-denied.md) | "I can't open the Sales-Docs folder." | User not in the group that the folder's NTFS permissions allow | Active Directory Users and Computers, NTFS permissions |
| 7 | [Delegated password reset](scenarios/07-delegated-password-reset.md) | "He gets an access error." (a technician cannot reset a password) | The IT-Support group had no permission to reset passwords | RSAT, Effective Access, Delegation of Control wizard |

### Group Policy

| # | Scenario | What the user reported | Root cause | Main tools |
|---|---|---|---|---|
| 6 | [GPO not applying (wrong OU)](scenarios/06-gpo-not-applying-wrong-ou.md) | "My colleagues in Sales say theirs is blocked." | User account in the wrong OU, so the OU-linked GPO did not reach it | `gpresult`, Group Policy Management |

---

[← Home](../README.md)
