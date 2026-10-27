# M4 — Account & Audit Hardening

## Objective

Scope administrative access, add lockout protection against repeated failed logons, and
turn on detection-oriented audit logging across the domain — closing the account-security
and visibility gaps left after M3's protocol-level hardening (SMB signing, LAN Manager
hash storage).

![M4 hardening overview: controls per zone across M1–M4, with the events that verified them](../images/m4-hardening-overview.svg)

## What Was Built

### 1. Account Lockout Policy

- Threshold: 3 → 5 invalid attempts
- Duration / observation window: unchanged (30 minutes each)
- Applied via Default Domain Policy in Group Policy Management — AD administration stays
  GUI-based throughout this project, not PowerShell

### 2. NTLM Authentication — Audit-Only

- **Restrict NTLM: Audit NTLM authentication in this domain** → `Enable all`
- **Restrict NTLM: Audit Incoming NTLM Traffic** → `Enable auditing for all accounts`
- Deliberately set to the broadest option in each dropdown, not the narrower
  domain-accounts-only default — so NTLM traffic from a non-domain-account position (an
  external or attacker-perspective connection) is also visible, not just traffic from
  already-authenticated domain accounts
- No restriction or blocking yet — this is visibility before enforcement, so the next
  decision (actually restricting NTLM) can be made from real audit data instead of a guess

### 3. Advanced Audit Policy (detection-oriented categories)

Prerequisite: **Audit: Force audit policy subcategory settings to override audit policy
category settings** → Enabled, so the granular subcategories below actually take effect.

| Category | Subcategory | Setting |
|---|---|---|
| Account Logon | Kerberos Authentication Service | Success and Failure |
| Account Logon | Kerberos Service Ticket Operations | Success only (noise control) |
| Logon/Logoff | Logon | Success and Failure |
| Logon/Logoff | Logoff | Success |
| Logon/Logoff | Account Lockout | Success and Failure |
| Logon/Logoff | Special Logon | Success |
| Account Management | User Account Management | Success and Failure |
| Account Management | Security Group Management | Success and Failure |
| Account Management | Computer Account Management | Success |

Most of this baseline was already correctly configured from M2/M3. Only three
subcategories needed an explicit change this milestone (Kerberos Authentication Service,
Account Lockout, and User Account Management each needed Failure added to an
existing Success-only setting).

### 4. Admin-separation model (confirmed, not new this milestone)

- `adm-dan` — delegated rights, not a Domain Admin account
- Logon restricted to the ADMIN-zone workstation (win10-tech01)

## Expected vs. Actual

| Control | Expected | Actual |
|---|---|---|
| Lockout threshold | Account locks after the 5th failed attempt | Confirmed — Event 4740 logged on the 5th attempt |
| NTLM audit | Audit event logged for NTLM traffic, nothing blocked | Confirmed — Event 8002, phrased "that would be blocked," confirming audit-only mode is working as designed |
| Kerberos logging | 4768 (TGT) + 4769 (service ticket) logged on a domain logon | 4768 confirmed, cleanly attributed to a deliberate test; 4769 confirmed via category volume (869 events in the session) rather than one isolated matched pair |
| Audit categories | All nine target subcategories at spec | Confirmed via `auditpol /get`, one category at a time, after fixing three gaps |

## Failures & Fixes

Nothing went wrong this stage.


## Verification Evidence

- Event 4740 — account lockout fired on the 5th failed attempt
- Event 8002 — NTLM incoming traffic, audit-only mode confirmed
- Event 4768 — Kerberos TGT request, cleanly attributed to a deliberate test
- Event 4769 — Kerberos service ticket operations, category confirmed active (869 events)
- `auditpol /get` output, one category at a time — all nine target subcategories
  confirmed applied, post-`gpupdate /force`

## What I Learned

Configuring a control and proving it fired are two different steps, and the gap between
"I set this" and "I can show this working" is exactly where undocumented security work
loses credibility. Not every setting verifies the same way — some live in the GUI's own saved state, some in a log category, some only
show up as an actual triggered event — and reaching for the wrong one wastes time without
proving anything. The more useful realization was about 4769 specifically: a control
doesn't need one clean, isolated log line to be confirmed working. Sometimes the proof
that something is correctly configured is that it's firing constantly — and the real
engineering problem that creates isn't "is it logging," it's "how do you make that volume
usable," which is exactly the question M5 has to answer.
