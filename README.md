# Windows Support Home Lab

A Windows Server 2025 and Windows 11 Pro domain environment built from scratch
in VirtualBox to practice the core skills used in IT help desk work.

![Network Diagram](screenshots/01-network-diagram.png)

## What I Built

A small Active Directory domain (lab.local) running on a Windows Server 2025
domain controller, with a Windows 11 Pro client joined to the domain. The
server handles AD, DNS, DHCP, Group Policy, and file sharing. The client
receives its IP from DHCP, authenticates to the domain, and has restrictions
applied via Group Policy.

## Lab Setup

| Machine | Role | OS | IP |
|---------|------|-----|-----|
| DC01 | Domain Controller, DNS, DHCP, File Server | Windows Server 2025 | 10.0.0.1 |
| WS01 | Domain-joined client | Windows 11 Pro | DHCP (10.0.0.100–200) |

**Domain:** lab.local
**Hypervisor:** Oracle VirtualBox
**Network:** VirtualBox Internal Network (isolated from host)

## Skills Practiced

- Active Directory Domain Services: installed AD DS, promoted server to domain controller, created forest
- User and group management: created OUs, users, security groups, and managed group membership
- DNS and DHCP: configured DNS zones and a DHCP scope with reservations and options
- Group Policy: created GPOs to enforce password policy and restrict standard user access
- File sharing: configured shared folders with NTFS and share permissions
- Windows 11 domain join and troubleshooting
- PowerShell basics for user and account management
- Network troubleshooting using ipconfig, ping, nslookup, and dcdiag

## Screenshots

See the [screenshots/](screenshots/) folder for visual documentation of the
lab including Server Manager, Active Directory Users and Computers, the DHCP
console, the domain login screen, and applied Group Policy results.

## Troubleshooting

During the build, I encountered and resolved several issues. The most
notable is documented in [troubleshooting.md](troubleshooting.md), including
a DHCP authorization failure (error code 20070) that I diagnosed and fixed.

## Why I Built This

I built this lab to get hands-on experience with the technologies I'll be
supporting in an entry-level help desk role. Reading about Active Directory
is one thing; actually creating users, resetting passwords, unlocking
accounts, and fixing a broken domain trust relationship is another. The lab
lets me practice real scenarios in a safe environment.
