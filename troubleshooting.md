# Troubleshooting Log

Real issues I encountered while building this lab and how I diagnosed and
resolved them. Each entry follows the same format used in a help desk ticket:
symptom, diagnosis, root cause, and resolution.

---

## Issue 1: Windows 11 Install Failed Hardware Requirements Check

**Symptom:** During Windows 11 installation in VirtualBox, setup displayed
the error "This PC can't run Windows 11" and would not proceed past the
hardware check, even with TPM 2.0 and Secure Boot enabled in the VM settings.

**Diagnosis:**
1. Verified TPM 2.0 was enabled in VirtualBox VM settings under System
2. Verified Secure Boot and EFI were enabled
3. Confirmed the VM had 4 GB RAM and 2 vCPUs assigned
4. Reviewed Microsoft's documented bypass method for VM installations

**Root Cause:** VirtualBox's TPM emulation is not always recognized by the
Windows 11 installer, even when correctly configured. This is a known
limitation when installing Windows 11 in VirtualBox.

**Resolution:**
1. Pressed Shift+F10 during the hardware check error to open Command Prompt
2. Ran `regedit` and navigated to `HKEY_LOCAL_MACHINE\SYSTEM\Setup`
3. Created a new key named `LabConfig`
4. Added three DWORD values, each set to 1:
   - `BypassTPMCheck`
   - `BypassSecureBootCheck`
   - `BypassRAMCheck`
5. Closed regedit, returned to the installer, and clicked back then Next
6. Installation proceeded successfully

**Lesson Learned:** Virtualized environments sometimes need registry
workarounds that would never be used in production. Knowing the difference
between a lab workaround and a production fix is important.

---

## Issue 2: Domain Join Failed with "The domain could not be contacted"

**Symptom:** When attempting to join WS-01 to the helpdesk.lab domain,
Windows returned the error "An Active Directory Domain Controller for the
domain helpdesk.lab could not be contacted."

**Diagnosis:**
1. Ran `ping 10.0.0.10` from WS-01 — successful, confirming network
   connectivity to the domain controller
2. Ran `ping helpdesk.lab` — failed with "could not find host"
3. Ran `nslookup helpdesk.lab` — returned "server can't find helpdesk.lab"
4. Ran `ipconfig /all` and noticed the DNS server was set to automatic
   instead of pointing to the domain controller

**Root Cause:** The workstation was not configured to use DC-01 as its DNS
server. Without DNS resolution, the client could not locate the SRV records
that Active Directory uses to find a domain controller.

**Resolution:**
1. Opened Network and Sharing Center on WS-01
2. Changed adapter properties > IPv4 > set Preferred DNS Server to 10.0.0.10
3. Ran `ipconfig /flushdns` to clear cached DNS entries
4. Ran `nslookup helpdesk.lab` again — successfully resolved to 10.0.0.10
5. Retried the domain join and it completed successfully

**Lesson Learned:** DNS is the foundation of Active Directory. When anything
related to domain authentication or discovery fails, DNS is the first thing
to check. This experience reinforced why domain-joined workstations must
always point to an internal DNS server, never public DNS like 8.8.8.8.

---

## Issue 3: Group Policy Not Applying to Test User

**Symptom:** After creating a Group Policy Object to restrict access to
Control Panel for standard users and linking it to the HelpDesk Lab Users
OU, the policy did not take effect. The test user could still open Control
Panel normally on WS-01.

**Diagnosis:**
1. Verified the GPO was linked to the correct OU in Group Policy Management
2. Confirmed the test user account was located in the HelpDesk Lab Users OU
3. Ran `gpupdate /force` on WS-01 — completed successfully but the policy
   still did not apply
4. Ran `gpresult /r` as the test user — the restrictive GPO was not listed
   under Applied Group Policy Objects
5. Realized the user was still logged in from a session that started before
   the GPO was created

**Root Cause:** User-side Group Policy settings are only evaluated at logon.
Running `gpupdate /force` refreshes policies but does not reapply user
settings to an active session. The user needed to log off and log back on.

**Resolution:**
1. Logged the test user off WS-01
2. Logged back in with the same account
3. Attempted to open Control Panel — access was now blocked with the
   expected message
4. Ran `gpresult /r` to confirm the GPO appeared in Applied Group Policy
   Objects

**Lesson Learned:** Computer policies apply at startup and user policies
apply at logon. `gpupdate /force` helps but it does not replace a logoff
for user-side policies. This is a common point of confusion and one of the
first things to check when a GPO is not working as expected.

---

## Summary

These three issues cover the most common categories of problems a help desk
technician handles: installation and compatibility, networking and name
resolution, and Group Policy application. Working through them in the lab
gave me hands-on experience with the diagnostic process, not just the end
resolution.
