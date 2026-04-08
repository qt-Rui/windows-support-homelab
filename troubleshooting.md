# Troubleshooting Log

Real issues I ran into while building this lab and how I fixed them. I kept
this log because troubleshooting is the core of help desk work, and being
able to walk through a problem from symptom to resolution matters more than
just getting things to work.

---

## Issue 1: Shared Folder Access Denied After Adding User to a Group

**What happened:** I set up a department share on the domain controller
and configured NTFS permissions so that only members of the IT Support
security group could access the IT folder. After adding a test user to
the IT Support group, they still got "Access Denied" when trying to open
the folder from the workstation.

**How I fixed it:**
1. Verified the user was actually in the IT Support group using Active
   Directory Users and Computers — they were
2. Double-checked the NTFS permissions on the folder — IT Support had
   Modify access as expected
3. Ran `whoami /groups` on the workstation while logged in as the user —
   IT Support was not listed in the output
4. Realized the user's Kerberos token was generated at logon and didn't
   include the new group membership
5. Had the user log off and back on — the group now showed up in
   `whoami /groups` and the folder opened without issue

**Takeaway:** Group membership changes don't take effect until a user
logs off and back on, because security tokens are built at logon time.
This is one of the most common "but I already added them to the group"
help desk tickets, and the fix is almost always a fresh login.

---

## Issue 2: Couldn't Join Windows 11 to the Domain

**What happened:** When I tried to join WS-01 to helpdesk.lab, Windows
returned "An Active Directory Domain Controller for the domain
helpdesk.lab could not be contacted."

**How I fixed it:**
1. Pinged the domain controller by IP (10.0.0.10) — worked fine
2. Tried `nslookup helpdesk.lab` — failed to resolve
3. Checked the network adapter settings and saw DNS was set to automatic
4. Changed the preferred DNS server to 10.0.0.10 (the domain controller)
5. Ran `ipconfig /flushdns` and retried the domain join — it worked

**Takeaway:** DNS is the foundation of Active Directory. If a client can't
resolve the domain name, it can't find a domain controller. Any
domain-joined machine needs to point to an internal DNS server, not a
public one like 8.8.8.8.

---

## Issue 3: Group Policy Wasn't Applying

**What happened:** I created a GPO to restrict Control Panel access for
standard users and linked it to the correct OU. After running
`gpupdate /force` on the workstation, the user could still open Control
Panel normally.

**How I fixed it:**
1. Confirmed the GPO was linked to the right OU
2. Confirmed the user account was in that OU
3. Ran `gpresult /r` and noticed the GPO wasn't in the Applied list
4. Realized user policies only apply at logon, not during `gpupdate`
5. Logged the user off and back on — the restriction now worked as expected

**Takeaway:** `gpupdate /force` refreshes computer policies but user-side
policies only reapply at logon. When a GPO isn't working, checking whether
the user has logged off and back on is one of the first things to try.

---

These three issues cover the kinds of problems help desk techs handle every
day: installation and compatibility, networking and name resolution, and
Group Policy. Working through them in the lab gave me experience with the
full diagnostic process, not just the end resolution.
