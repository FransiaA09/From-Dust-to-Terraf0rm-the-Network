# Milestone 3 — Network Hardening

## Scope

This milestone hardens the segmented network built in M1 and attacked (baseline) in M2.
It replaces the temporary allow-all rules on the USERS and ADMIN firewall interfaces with
an explicit, least-privilege allow-list; restricts the firewall's own management interface
to the ADMIN zone only; and applies two Group Policy hardening settings domain-wide.

**Scenario context:** this lab models the internal network of a small (~20-25 person)
managed service provider (MSP) — specifically the MSP's own identity and access
infrastructure (the AD environment its technicians use to do their jobs and reach client
systems), not a multi-tenant, client-segregated environment. The MSP's own AD is treated
as the highest-value asset in this model: an attacker who compromises it doesn't just harm
the MSP, they gain a potential foothold toward every client the MSP supports. This mirrors
a real, documented risk class — the July 2021 Kaseya VSA supply-chain attack, in which a
vulnerability in one piece of MSP management software was used to deploy ransomware through
roughly 50 direct MSP customers to an estimated 800–1,500 downstream businesses. The
ADMIN-zone isolation and management-GUI restriction in this milestone are a direct, if
much smaller-scale, response to that risk pattern: limit who can reach the tools that
reach everything else.

No specific compliance framework (PCI, HIPAA, etc.) is claimed. Controls are justified by
what they protect and why, not by citing a named standard this lab hasn't been tested
against.

## Before / after: M2 baseline vs. M3 hardened state

The network topology itself (zones, subnets, machines) is unchanged from M1 — M3 only
changes policy and configuration, not architecture, so the clearest way to show what this
milestone did is a direct before/after comparison rather than a new topology diagram.

| Control | M2 (baseline) | M3 (hardened) |
|---|---|---|
| OPT1/OPT2 firewall rules | Single `Pass / any / any / any` rule on each interface (allow-all) | Explicit 8-rule allow-list per interface — DNS, Kerberos, RPC endpoint mapper, LDAP, SMB, Kerberos password change, dynamic RPC, and GUI access to the firewall's own ADMIN address; everything else denied by default |
| Ports reachable, USERS → SERVERS | All 11 found open in A1's scan: 53, 88, 135, **139**, 389, 445, 464, **636**, **3268**, **3269**, **3389** | 7 explicit ports only (53, 88, 135, 389, 445, 464, 49152–65535); 139, 636, 3268/3269, and 3389 deliberately excluded (see allow-list table above for reasoning) |
| OPNsense management GUI reachable from | WAN, SERVERS, USERS, and ADMIN (Listen Interfaces = "All") | ADMIN only |
| LLMNR (multicast name resolution) | Enabled (Windows default — never explicitly configured) | Disabled domain-wide via Default Domain Policy |
| SMB signing | Required on the DC only; not enforced on `win10-tech01` | Required domain-wide (`RequireSecuritySignature = True` confirmed on both DC and client server roles) |
| M2 test accounts (`TalA`, `KosiDan`, `DoeJ`, `RonA`, `SalomonT`, `svc-sql`) and their group/GPO grants | Present, active | Deleted; associated grants removed |

## What changed

### 1. Default-deny firewall policy (OPT1/USERS, OPT2/ADMIN)

The temporary allow-all rules from M1 (added to work around a gap found during that
milestone's build — see M1's own failure log) were replaced with an explicit allow-list on
both interfaces. Rules were derived from M2's actual A1 scan results (11 ports found open
under allow-all from USERS to the DC) rather than a generic AD-ports checklist, so the
list reflects what this specific single-domain, single-DC environment needs, not every
port AD can theoretically use.

| Port | Protocol | Purpose | Included? | Reasoning |
|---|---|---|---|---|
| 53 | TCP/UDP | DNS | Yes | Core |
| 88 | TCP/UDP | Kerberos | Yes | Core |
| 135 | TCP | RPC Endpoint Mapper | Yes | Core |
| 139 | TCP | NetBIOS Session Service | **No** | Legacy SMB transport, redundant with 445 on an all-Windows-10/Server-2022 network; no host here needs it |
| 389 | TCP/UDP | LDAP | Yes | Core |
| 445 | TCP | SMB | Yes | Core |
| 464 | TCP/UDP | Kerberos password change | Yes | Core |
| 636 | TCP | LDAPS | No | Not confirmed in use in this environment |
| 3268/3269 | TCP | Global Catalog | **No** | Single-domain, single-DC forest; GC queries are a multi-domain feature this topology has no use for. A1 found them open only because nothing was restricting them, not because anything needs them |
| 49152–65535 | TCP | Dynamic RPC | Yes | Required for WMI/GPO/misc RPC; allowed as a full range at this milestone, with payload-level inspection deferred to M5 (Suricata IPS) rather than attempted here — a firewall ACL can allow or deny a port, it cannot selectively "inspect and release" a connection |

A second rule was added on each interface: ADMIN → the firewall's own ADMIN-zone
address, port 443 (GUI access). This wasn't in the original design — it was found during
testing, see Failures below.

### 2. Management GUI restricted to ADMIN only

`System → Settings → Administration → Listen Interfaces` was changed from "All
(recommended)" to the ADMIN interface only. This is a separate control plane from the
firewall's packet-filter rules (confirmed directly in M2: the GUI was reachable from WAN
and USERS regardless of what the ruleset said, because this setting — not a `pass` rule —
governs which interfaces the management service listens on at all).

### 3. GPO hardening (Default Domain Policy)

- **LLMNR disabled** — `Computer Configuration → Administrative Templates → Network →
  DNS Client → Turn off multicast name resolution` = Enabled.
- **SMB signing required** — `Computer Configuration → Windows Settings → Security
  Settings → Local Policies → Security Options → Microsoft network server: Digitally sign
  communications (always)` = Enabled. (Previously enforced on the DC only; now domain-wide.)

### 4. Test-account cleanup

The five M2 test users (`TalA`, `KosiDan`, `DoeJ`, `RonA`, `SalomonT`) and the `svc-sql`
service account (used for the M2 Kerberoasting demonstration) were deleted. `KosiDan`'s
Remote Desktop Users group membership was removed with the account. No separate "Allow log
on through Remote Desktop Services" grant was found defined on the Default Domain
Controllers Policy for `TalA` at cleanup time — the end state (no special RDP-logon right
present) matches the intended outcome regardless.

## Test matrix

| # | Test | Expected | Actual | Result |
|---|---|---|---|---|
| T1 | `win10-client01` (USERS) domain logon / `gpupdate /force` after OPT1 rule apply | Succeeds | Succeeded, no errors | Pass |
| T2 | `win10-tech01` (ADMIN) domain logon / `gpupdate /force` after OPT2 rule apply | Succeeds | Succeeded, no errors (after fixing a Source misconfiguration on one rule — see Failures) | Pass |
| T3 | OPNsense GUI reachable from `win10-tech01` via `https://10.10.30.1` (ADMIN's own address) | Succeeds | Succeeded after adding the GUI-access rule | Pass |
| T4 | OPNsense GUI reachable via `https://10.10.10.1` (SERVERS address) from ADMIN | Fails (no rule permits this) | Failed — blocked, confirmed in firewall log | Pass |
| T5 | Listen Interfaces restricted — GUI unreachable from SERVERS (`https://10.10.10.1` from the DC's own browser) | Fails (connection timeout) | Failed exactly as expected — `ERR_CONNECTION_TIMED_OUT` | Pass |
| T6 | GUI still reachable from ADMIN after Listen Interfaces restriction | Succeeds | Succeeded | Pass |
| T7 | A2 reachability re-test — GUI unreachable from WAN (Kali, `curl -k -I https://10.10.10.1` and `https://10.10.30.1`) | Both fail to connect | Both failed — "Could not connect to server" on both targets | Pass |
| T8 | A2 reachability re-test — GUI reachable from ADMIN (`win10-tech01`) | Succeeds | Succeeded (TLS handshake completed; self-signed cert rejected by curl as expected) | Pass |
| T9 | SMB (port 445) transport reachable from USERS to SERVERS after GPO signing enforcement | Succeeds | Confirmed via `Test-NetConnection -ComputerName 10.10.10.10 -Port 445` → `TcpTestSucceeded: True` | Pass |
| T10 | SMB signing configuration applied domain-wide | `RequireSecuritySignature: True` on server role, both DC and client | Confirmed via `Get-SmbServerConfiguration` on both machines | Pass |

Live brute-force execution against the now-restricted GUI (the attack portion of A2, beyond
the reachability test) was not run as part of this milestone and remains available as
follow-up work; the reachability results above were judged sufficient evidence that the
GUI-restriction control itself holds.

## Failures and fixes

**Failure 1 — Missing GUI-access rule on ADMIN interface.** After replacing the OPT1/OPT2
allow-all rules with the explicit AD-ports allow-list, the OPNsense management GUI became
unreachable even from the ADMIN zone — including from `10.10.30.1`, the firewall's own
ADMIN-side address. Diagnosed via the firewall's own Live Log: blocked entries showed
destination `10.10.30.1:443` hitting the default-deny rule. Root cause: the AD-ports
allow-list (DNS, Kerberos, LDAP, SMB, etc.) never included a rule for reaching the
firewall's management interface itself — that traffic class wasn't part of the M2 scan data
the allow-list was built from, since A1 scanned the domain controller, not the firewall.
Fixed by adding one rule per interface: ADMIN network → the firewall's own ADMIN address,
TCP 443. This rule turned out to be a necessary prerequisite for Step 5 (GUI restriction)
as well, so finding it before that step avoided a second, overlapping lockout.

**Lesson, not a technical failure — testing methodology.** Verifying SMB connectivity
after the signing-enforcement GPO change took longer than it should have, because the
first test account used (not one of the M2 test users, but a separate pre-existing account)
turned out to have an unrelated "must change password at next logon" flag, which produces
a network-authentication failure (System error 5) that looks identical to a real
signing/auth problem but is unrelated to it. Switching to `Test-NetConnection` to isolate
transport reachability from credential state resolved the ambiguity directly. **Lesson for
future test design:** when verifying a security control with a specific test account,
confirm the account's own state (enabled, not locked, no pending password-change flag)
before treating an auth failure as evidence about the control under test.

## What I learned

- **A firewall allow-list and a service's "listen interfaces" setting are two independent
  control planes.** M2 demonstrated this directly: the OPNsense GUI was reachable from
  zones the firewall ruleset "shouldn't" have permitted, because the ruleset never governed
  GUI access in the first place — a separate Administration setting did. Hardening a
  network means checking both layers, not assuming a packet-filter rule change closes
  every access path.
- **Build an allow-list from measured traffic, not a checklist, then actively trim what
  the topology doesn't need.** Starting from M2's actual scan results (what was genuinely
  reachable under allow-all) rather than a generic "AD ports" list, and then reasoning
  about which of those are structurally necessary for a single-domain, single-DC forest
  (dropping GC ports, dropping legacy NetBIOS) produced a tighter, more defensible policy
  than either extreme — and the reasoning behind each inclusion/exclusion is something I
  can actually explain, which matters more for a portfolio piece than the port count itself.
- **A firewall ACL is binary (allow/deny); inline inspection is a different, separate
  capability.** The dynamic RPC range can't be "allowed but watched for the first bad
  packet" at the packet-filter layer — that requires a true inline IPS (Suricata, planned
  for M5). Knowing the boundary between what a stateless ACL can and can't do mattered more
  here than trying to force a packet-filter rule to do a job it isn't built for.
- **NTP is a dependency of Kerberos, not just a convenience.** The OPT1/OPT2 allow-list in
  this milestone does not include UDP 123, which means client clock sync via the DC is
  currently blocked for USERS/ADMIN hosts. It hasn't caused a failure yet (clocks were
  checked and are still within Kerberos's tolerance), but it's a real, currently-open gap:
  if clock drift exceeds roughly 5 minutes, Kerberos authentication will start failing
  domain-wide for USERS/ADMIN hosts, and the failure will present as an authentication
  problem with no obvious network cause. **Flagged for a follow-up fix**, not resolved in
  this milestone.

## Known open items carried forward

- NTP (UDP 123) is not in the OPT1/OPT2 allow-list — a real gap, not yet causing symptoms.
  Should be added in a near-term follow-up, independent of the next milestone.
- Live brute-force execution against the GUI (completing A2's attack portion, not just its
  reachability check) is deferred.
- 636 (LDAPS) was excluded from the allow-list as unconfirmed-in-use; revisit if any future
  milestone introduces an LDAPS-dependent service.
