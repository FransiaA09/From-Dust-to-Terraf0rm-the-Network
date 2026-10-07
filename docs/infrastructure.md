[Home](../README.md)

# Infrastructure

![Lab diagram](images/lab-diagram.png)

## Environment

| Component | Details |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Host RAM | 16 GB |
| Network mode | Internal Network (`LAB-INT`): isolated, no internet access. A NAT network was used temporarily for installs, updates, and the RSAT download |
| Domain | `lab.local` |
| NetBIOS name | `LAB` |

## Virtual machines

| VM Name | Role | OS | IP Address | RAM | vCPU |
|---|---|---|---|---|---|
| win2k22-DC01 | Domain Controller / DNS / DHCP | Windows Server 2022 Standard (Desktop Experience) | 10.10.10.10 (static) | 6 GB | 2 |
| win10-client01 | Employee workstation / ticket target (domain-joined) | Windows 10 Pro | DHCP-assigned (10.10.10.100–200) | 4 GB | 2 |
| win10-tech01 | Technician workstation: RDP source, RSAT installed, domain-joined during the delegation stage | Windows 10 Pro | DHCP-assigned (10.10.10.101 observed) | 2 GB | 1 |

*RAM and vCPU are the planned allocations and were not re-checked after the VMs were created.*

This page describes the lab as a whole and grows as new areas are added. Each area keeps its own build details in its own folder.

---

[← Home](../README.md)
