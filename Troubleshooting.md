# Troubleshooting Log

Real issues I encountered while building the Windows Support Home Lab,
how I diagnosed them, and how I resolved them. These are the kinds of
problems a help desk technician works through daily.

---

## Issue 1: DHCP Authorization Failed with Error Code 20070

### The Problem

After installing the DHCP Server role on my Windows Server 2025 domain
controller, I ran the Post-Install Configuration Wizard to authorize the
server in Active Directory. When I clicked **Commit**, the wizard returned:

> The authorization of DHCP Server failed with Error Code: 20070.
> The DHCP service could not contact Active Directory.

The DHCP console showed a red downward arrow on the IPv4 node, meaning
the server was installed but not authorized to hand out IP addresses.

### What I Tried

**Step 1 — Checked Active Directory health.** My first thought was that
AD itself might be the problem, since the error mentioned it could not
contact Active Directory. I opened an elevated command prompt and ran:

dcdiag/v

All tests passed. I also opened Active Directory Users and Computers and
confirmed the lab.local domain was visible and populated. AD was healthy.

**Step 2 — Checked DNS resolution.** Since AD depends on DNS, I tested
name resolution:

nslookup lab.local

This returned 10.0.0.1 as expected. DNS was working.

**Step 3 — Checked my user context.** I ran `whoami` and realized I was
signed in as the local Administrator account. Even on a domain controller,
the local Administrator context doesn't always carry the same domain-level
permissions needed to write the DHCP authorization entry into Active
Directory.

**Step 4 — Re-ran the wizard with explicit domain admin credentials.**
In the Post-Install Configuration Wizard, I specified
`LAB\Administrator` and the domain admin password explicitly instead of
using the current user. The commit completed successfully.

### Root Cause

DHCP server authorization writes an entry to the Active Directory
configuration partition, which requires domain-level credentials. The
wizard was defaulting to my current session, which didn't have the
permissions needed to make that write.

### Resolution

Provided `LAB\Administrator` credentials directly in the commit dialog.
The red arrow in the DHCP console turned to a green upward arrow,
confirming the server was authorized. I then created and activated the
DHCP scope without any further issues.

### What I Learned

The error message said "could not contact Active Directory," but the real
problem wasn't connectivity — it was permissions. This was a good reminder
that error messages don't always point directly at the cause, and that
local admin and domain admin are fundamentally different things, even
on a domain controller. When a wizard offers a credentials field, there's
usually a reason.

---

## Issue 2: Windows 11 Client Could Not Join the Domain

### The Problem

After installing Windows 11 Pro on WS01 and attempting to join the
lab.local domain, the join failed with:

> An Active Directory Domain Controller for the domain "lab.local"
> could not be contacted.

### What I Tried

**Step 1 — Verified basic network connectivity.** On the client, I ran:

ping 10.0.0.1

The ping succeeded, so the client could reach the server on the internal
network.

**Step 2 — Checked the client's IP configuration.** I ran `ipconfig /all`
and noticed the client had pulled an IP from DHCP (10.0.0.101), but the
DNS server field was blank. That was the problem — without a DNS server,
the client couldn't resolve `lab.local` to the domain controller's IP.

**Step 3 — Traced the cause back to the DHCP scope.** I went back to
DC01, opened the DHCP console, and looked at the scope options for my
LabNet scope. Option 006 (DNS Servers) had not been configured when I
created the scope. The client was getting an address but no DNS
information.

**Step 4 — Added the DNS server to the scope options.** I right-clicked
**Scope Options** under the LabNet scope, selected **Configure Options**,
checked **006 DNS Servers**, and added 10.0.0.1. I clicked OK to apply.

**Step 5 — Renewed the client's IP.** On the client, I ran:

ipconfig/release
ipconfig/renew
ipconfig/all

This time the DNS server showed as 10.0.0.1. I ran `nslookup lab.local`
to confirm DNS was resolving, and it returned 10.0.0.1.

**Step 6 — Retried the domain join.** The join completed successfully,
and the client restarted into the domain.

### Root Cause

I had created the DHCP scope but skipped the step of configuring scope
option 006 (DNS Servers). Without it, clients received an IP address
but had no way to resolve the domain name, which made the domain join
impossible.

### Resolution

Added 10.0.0.1 as the DNS server in the scope options, renewed the
client's lease, and rejoined the domain.

### What I Learned

DNS is the foundation of Active Directory. Almost every domain join
failure, login issue, or "can't find network resource" problem traces
back to DNS. Now, whenever a machine can't reach a domain, my first
check is always `ipconfig /all` to confirm the DNS server is set
correctly. If DNS isn't right, nothing else will work, and you can
waste hours troubleshooting the wrong thing.

---

## Quick Reference: Commands I Use Most for Troubleshooting

| Command | What It Does |
|---------|--------------|
| `ipconfig /all` | Shows full network config including DNS servers |
| `ipconfig /release` then `/renew` | Forces a new DHCP lease |
| `ipconfig /flushdns` | Clears the local DNS cache |
| `ping [host]` | Tests basic connectivity |
| `nslookup [name]` | Tests DNS resolution |
| `dcdiag /v` | Runs domain controller health checks |
| `gpupdate /force` | Forces Group Policy to refresh immediately |
| `gpresult /r` | Shows which GPOs are applied to the current user and computer |
