# 13 - Join WS01 to the Domain

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Intermediate-orange?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/VM-WS01-blue?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Domain-Lab.local-darkgreen?style=for-the-badge)

> Joining WS01 to the domain is the moment everything comes together.
> DC01 becomes the authority that authenticates every login on WS01.
> Group Policy from DC01 will apply to WS01 automatically.
> All 110 domain users will be able to log into WS01 with their credentials.
> This single step transforms WS01 from a standalone workstation
> into a fully managed enterprise domain member.

---

## 📑 Table of Contents

- [What Domain Joining Does](#-what-domain-joining-does)
- [Prerequisites Check](#-prerequisites-check)
- [Join the Domain via PowerShell](#-join-the-domain-via-powershell)
- [Join the Domain via GUI](#-join-the-domain-via-gui)
- [After the Restart — First Domain Login](#-after-the-restart--first-domain-login)
- [Verify the Domain Join on DC01](#-verify-the-domain-join-on-dc01)
- [Move WS01 to the _COMPUTERS OU](#-move-ws01-to-the-_computers-ou)
- [Test Logging In with Domain Users](#-test-logging-in-with-domain-users)
- [Take a Snapshot](#-take-a-snapshot)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)

---

## 🧠 What Domain Joining Does

When a computer joins a domain, a **trust relationship** is established
between the computer and the Domain Controller. Here is what changes:

```
┌─────────────────────────────────────────────────────────────┐
│               Before Domain Join                             │
│                                                             │
│   WS01 knows about:    LocalAdmin (local account only)      │
│   Authenticates via:   Local SAM database on WS01           │
│   Group Policy:        Local GPO only                       │
│   Management:          Must physically go to the machine     │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│               After Domain Join                              │
│                                                             │
│   WS01 knows about:    All 110 LAB domain users             │
│   Authenticates via:   DC01 Kerberos authentication         │
│   Group Policy:        Domain GPOs pushed from DC01         │
│   Management:          Fully managed remotely from DC01     │
│                                                             │
│   What gets created on DC01:                                │
│   - Computer account object for WS01 in Active Directory    │
│   - Secure channel between WS01 and DC01                    │
│   - Machine certificate for Kerberos trust                  │
└─────────────────────────────────────────────────────────────┘
```

---

## ✅ Prerequisites Check

Before attempting the domain join, run all three tests from WS01.
All three must pass — if any fail, do not proceed until fixed.

Open **PowerShell as Administrator** on WS01 and run:

### Test 1 — Ping DC01

```powershell
ping 192.168.100.10 -n 4
```

Expected — 4 successful replies:

```
Reply from 192.168.100.10: bytes=32 time=2ms TTL=128
Reply from 192.168.100.10: bytes=32 time=1ms TTL=128
Reply from 192.168.100.10: bytes=32 time=2ms TTL=128
Reply from 192.168.100.10: bytes=32 time=1ms TTL=128

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)  ✅
```

---

### Test 2 — DNS Resolution

```powershell
nslookup Lab.local 192.168.100.10
```

Expected — domain resolves to DC01's IP:

```
Server:   DC01.Lab.local
Address:  192.168.100.10

Name:     Lab.local
Addresses: 192.168.100.10   ✅
```

---

### Test 3 — LDAP Port Reachable

```powershell
Test-NetConnection -ComputerName 192.168.100.10 -Port 389
```

Expected — LDAP port is open:

```
ComputerName     : 192.168.100.10
RemoteAddress    : 192.168.100.10
RemotePort       : 389
TcpTestSucceeded : True   ✅
```

> 💡 **Port 389 is LDAP** — the protocol Active Directory uses for directory queries.
> If this test fails the domain join will also fail, even if ping succeeds.
> A failed LDAP test usually means Windows Firewall on DC01 is blocking the port.

---

### All Three Tests Must Pass

| Test | Expected Result | If It Fails |
|------|----------------|-------------|
| Ping 192.168.100.10 | 4 replies, 0% loss | Check LabNet adapter name matches on both VMs |
| nslookup Lab.local | Returns 192.168.100.10 | Check WS01 DNS is set to 192.168.100.10 |
| Test-NetConnection Port 389 | TcpTestSucceeded: True | Check DC01 Windows Firewall |

---

## ⚡ Join the Domain via PowerShell

This is the fastest and most reliable method. Run on **WS01**.

### Single Command — Join and Rename Simultaneously

```powershell
Add-Computer `
    -DomainName "Lab.local" `
    -Credential "LAB\Administrator" `
    -NewName "WS01" `
    -OUPath "OU=_COMPUTERS,DC=Lab,DC=local" `
    -Restart `
    -Force
```

When prompted enter your **DC01 Administrator password**:

```
Windows PowerShell credential request
Enter your credentials.
Password for user LAB\Administrator: ************
```

After entering the password you will see:

```
WARNING: The changes will take effect after you restart the computer WS01.
```

WS01 will **automatically restart** to complete the domain join.

> 💡 **What each parameter does:**
> - `-DomainName` — the domain to join
> - `-Credential` — authenticates with DC01 using the domain Administrator account
> - `-NewName` — renames the computer at the same time as joining
> - `-OUPath` — places the computer object directly in _COMPUTERS OU
> - `-Restart` — restarts automatically to complete the join
> - `-Force` — suppresses confirmation prompts

---

## 🖥️ Join the Domain via GUI

If you prefer the graphical method, follow these steps on WS01.

### Step 1 — Open System Properties

Right-click the **Start button** → **System** → scroll down and click
**"Domain or workgroup"** → click **"Change settings"**.

The **System Properties** window opens. Click **"Change..."**.

---

### Step 2 — Select Domain

Under **"Member of"** select **Domain** (not Workgroup).

Type:
```
Lab.local
```

Click **OK**.

---

### Step 3 — Enter Credentials

A credentials box appears asking for an account with permission to join the domain.

| Field | Value |
|-------|-------|
| Username | `LAB\Administrator` |
| Password | Your DC01 Administrator password |

Click **OK**.

---

### Step 4 — Success Message

If everything is correct you will see:

```
┌────────────────────────────────┐
│                                │
│  Welcome to the Lab.local      │
│  domain.                       │
│                                │
│            [ OK ]              │
└────────────────────────────────┘
```

Click **OK** → **OK** → **Restart Now**.

---

## 🔐 After the Restart — First Domain Login

After WS01 restarts the login screen looks different.

### What You Will See

```
┌──────────────────────────────────────────┐
│                                          │
│         Sign in                          │
│                                          │
│   Other user  ←── Click this             │
│                                          │
│   LocalAdmin                             │
│                                          │
└──────────────────────────────────────────┘
```

### Log In as a Domain User

Click **"Other user"** at the bottom left of the login screen.

The login prompt changes to show:

```
Sign in to: LAB
```

Enter one of your bulk-created domain users:

```
Username:  LAB\rgomez
Password:  Welcome1@Lab
```

> 💡 **First login behaviour:**
> Because the script set `ChangePasswordAtLogon = $true`,
> the user will immediately be prompted to create a new password.
> This is correct and expected enterprise behaviour.
> Enter the old password `Welcome1@Lab` then set a new one meeting
> the complexity requirements (10+ characters, mixed case, numbers/symbols).

After the password change Windows 11 will log in and set up the
user profile — this takes 30–60 seconds on first login.

---

## 🔍 Verify the Domain Join on DC01

Switch to **DC01** and confirm WS01 appeared in Active Directory.

### Check 1 — Active Directory Users and Computers

Open **Server Manager → Tools → Active Directory Users and Computers**.

Expand **Lab.local**. Look in two places:

**Option A — Default Computers container** (if `OUPath` was not used):
```
Lab.local → Computers → WS01
```

**Option B — _COMPUTERS OU** (if `OUPath` was used in PowerShell):
```
Lab.local → _COMPUTERS → WS01
```

You should see a **computer object** named **WS01** with a monitor icon. ✅

---

### Check 2 — PowerShell on DC01

```powershell
# Confirm WS01 is registered in AD
Get-ADComputer -Identity "WS01" -Properties *|
    Select-Object Name, DNSHostName, IPv4Address,
                  DistinguishedName, Enabled, OperatingSystem
```

Expected output:

```
Name              : WS01
DNSHostName       : WS01.Lab.local
IPv4Address       : 192.168.100.20
DistinguishedName : CN=WS01,OU=_COMPUTERS,DC=Lab,DC=local
Enabled           : True
OperatingSystem   : Windows 11 Pro
```

---

### Check 3 — DNS Registration

```powershell
# Confirm WS01 registered its DNS record
Resolve-DnsName -Name "WS01.Lab.local"
```

Expected output:

```
Name           Type  TTL   Section  IPAddress
----           ----  ---   -------  ---------
WS01.Lab.local A     1200  Answer   192.168.100.20   ✅
```

WS01 automatically registered its own DNS A record with DC01's DNS
during the domain join process.

---

## 📁 Move WS01 to the _COMPUTERS OU

If WS01 landed in the default **Computers** container instead of your
custom **_COMPUTERS** OU, move it now. GPOs can only be linked to
proper Organisational Units — not to the default Computers container.

### Check Where WS01 Currently Is

```powershell
Get-ADComputer -Identity "WS01" |
    Select-Object DistinguishedName
```

If the output contains `CN=Computers` it needs to be moved:

```
CN=WS01,CN=Computers,DC=Lab,DC=local   ← Needs moving
CN=WS01,OU=_COMPUTERS,DC=Lab,DC=local  ← Already correct ✅
```

### Move via PowerShell

```powershell
# Move WS01 to the _COMPUTERS OU
Get-ADComputer -Identity "WS01" |
    Move-ADObject -TargetPath "OU=_COMPUTERS,DC=Lab,DC=local"

# Verify the move
Get-ADComputer -Identity "WS01" | Select-Object DistinguishedName
```

### Move via ADUC (GUI)

In Active Directory Users and Computers:

1. Find **WS01** in the **Computers** container
2. Right-click **WS01** → **Move**
3. Expand **Lab.local** → click **_COMPUTERS**
4. Click **OK**

---

## 👥 Test Logging In with Domain Users

Now test that multiple domain users can log into WS01.

### Test User Logins from WS01

Log out of the current session and try these at the login screen
using **"Other user"**:

```
User 1:   LAB\rgomez        Password: Welcome1@Lab
User 2:   LAB\bjackson      Password: Welcome1@Lab
User 3:   LAB\pcampbell     Password: Welcome1@Lab
```

Each will be prompted to change their password on first login — this is correct.

---

### Verify Group Membership on WS01

While logged in as a domain user, open PowerShell and run:

```powershell
# Show the current logged-in user
whoami
```

Expected output:

```
lab\rgomez
```

```powershell
# Show the domain groups this user belongs to
whoami /groups | findstr "Domain"
```

Expected output includes:

```
LAB\Domain Users         Group  Mandatory group, Enabled ✅
```

---

### Check GPO Status on WS01

```powershell
# Show which GPOs are currently applied
gpresult /r
```

Look for the **Computer Settings** and **User Settings** sections.
After the GPOs are created in Steps 14–17, you will see them listed here.

---

### Verify the Secure Channel Between WS01 and DC01

```powershell
# Test the trust relationship between WS01 and DC01
Test-ComputerSecureChannel -Verbose
```

Expected output:

```
VERBOSE: Performing the operation "Test-ComputerSecureChannel" on target "WS01".
True   ✅
```

`True` confirms the secure trust channel between WS01 and DC01 is healthy.

---

## 📸 Take a Snapshot

WS01 is domain-joined, verified in Active Directory, and domain users
can log in successfully. This is one of the most important milestones
in the entire lab.

Take a snapshot on **both VMs**:

**On WS01:**
```
Machine → Take Snapshot
Name: WS01 - Domain Joined - Lab.local - Pre GPO
```

**On DC01:**
```
Machine → Take Snapshot
Name: DC01 - WS01 Joined - Domain Complete - Pre GPO
```

> 🔒 **These snapshots protect your entire lab.**
> At this point you have a fully working Active Directory environment —
> DC01 with 110 users and WS01 joined to the domain.
> If any GPO configuration in the next steps causes problems,
> rolling back to these snapshots restores the complete working lab instantly.

---

## 🛠️ Troubleshooting

### Issue: "The specified domain either does not exist or could not be contacted"

The most common domain join error. Almost always a DNS problem.

**Fix — check in this order:**

```powershell
# Step 1 — Confirm WS01 DNS is pointing to DC01
Get-DnsClientServerAddress -AddressFamily IPv4
# Must show 192.168.100.10 — not 8.8.8.8, not blank

# Step 2 — Test DNS resolution directly
nslookup Lab.local 192.168.100.10
# Must return 192.168.100.10

# Step 3 — If DNS is wrong, fix it
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "192.168.100.10"

# Step 4 — Retry the domain join
Add-Computer -DomainName "Lab.local" -Credential "LAB\Administrator" -Restart -Force
```

---

### Issue: "Access is denied" or "Logon failure"

Wrong credentials were entered.

**Fix:**
- Username must be `LAB\Administrator` — not just `Administrator`
- Password must be the DC01 Administrator password set in Step 04
- Confirm you can log into DC01 with those credentials before retrying

---

### Issue: "The account already exists" error

A computer object named WS01 already exists in Active Directory from
a previous join attempt.

**Fix — on DC01:**

```powershell
# Remove the existing stale computer object
Remove-ADComputer -Identity "WS01" -Confirm:$false

# Verify it is gone
Get-ADComputer -Filter { Name -eq "WS01" }
```

Then retry the domain join from WS01.

---

### Issue: WS01 joined successfully but does not appear in ADUC

ADUC may need to be refreshed, or the computer landed in a different OU.

**Fix:**

```powershell
# On DC01 — search the entire domain for WS01
Get-ADComputer -Filter { Name -eq "WS01" } | Select-Object DistinguishedName
```

This will show exactly where the computer object is, regardless of which
container it landed in. Then move it to `_COMPUTERS` using the commands above.

---

### Issue: "Other user" option not showing on the WS01 login screen

The domain join may not have completed, or the login screen is showing
the local account only.

**Fix:**
1. Click anywhere on the login screen
2. Look at the bottom-left corner for **"Other user"**
3. If still not visible — press **Ctrl+Alt+Del** (via Input → Keyboard → Insert Ctrl+Alt+Del)
4. Select **Switch user** or **Sign in as different user**

---

### Issue: Test-ComputerSecureChannel returns False

The secure channel between WS01 and DC01 is broken.

**Fix:**

```powershell
# Attempt to repair the secure channel automatically
Test-ComputerSecureChannel -Repair -Credential "LAB\Administrator"
```

If repair fails:
```powershell
# Remove WS01 from the domain and rejoin
Remove-Computer -UnjoinDomainCredential "LAB\Administrator" -Restart -Force
# After restart, rejoin using Add-Computer
```

---

## ✅ Verification Checklist

```
Prerequisites (all three must pass before joining)
[ ] Ping 192.168.100.10 returns 4 replies, 0% loss
[ ] nslookup Lab.local returns 192.168.100.10
[ ] Test-NetConnection Port 389 shows TcpTestSucceeded: True

Domain Join
[ ] Add-Computer command ran successfully OR GUI method completed
[ ] Credentials entered as LAB\Administrator
[ ] WS01 restarted after domain join

First Domain Login
[ ] "Other user" option visible on login screen
[ ] Logged in with LAB\rgomez (or any bulk-created user)
[ ] Password change prompted and completed on first login
[ ] Windows 11 desktop loaded as domain user

DC01 Verification
[ ] Get-ADComputer WS01 shows correct details
[ ] DistinguishedName shows OU=_COMPUTERS (not CN=Computers)
[ ] Resolve-DnsName WS01.Lab.local returns 192.168.100.20
[ ] WS01 visible in Active Directory Users and Computers

WS01 Verification
[ ] whoami returns lab\username (domain format)
[ ] Domain Users group membership confirmed
[ ] Test-ComputerSecureChannel returns True

OU Placement
[ ] WS01 computer object is in OU=_COMPUTERS,DC=Lab,DC=local
[ ] Not in the default CN=Computers container

Snapshots
[ ] WS01 snapshot: "WS01 - Domain Joined - Lab.local - Pre GPO"
[ ] DC01 snapshot: "DC01 - WS01 Joined - Domain Complete - Pre GPO"

Ready to proceed
[ ] WS01 is a fully functioning domain member
[ ] Domain users can log into WS01 successfully
[ ] Secure channel between WS01 and DC01 is healthy
[ ] WS01 is in the _COMPUTERS OU — ready to receive GPOs
```

---

## ➡️ Next Step

WS01 is domain-joined and the lab infrastructure is complete.
Now we build the Group Policy Objects that manage the domain.

**[14 - GPO 1 Password Policy →](../14%20-%20GPO%201%20-%20Password%20Policy/README.md)**

---

## ⬅️ Previous Step

**[← 12 - Install Windows 11](../12%20-%20Install%20Windows%2011/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
