# 07 - Promote to Domain Controller

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Intermediate-orange?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/VM-DC01-purple?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Domain-Lab.local-darkgreen?style=for-the-badge)

> This is the most important step in the entire lab.
> Promotion transforms DC01 from a regular Windows Server into a fully
> functioning Domain Controller — creating the Lab.local domain,
> installing DNS, and establishing the Active Directory database.
> Every other component in this lab depends on this step completing successfully.

---

## 📑 Table of Contents

- [What Promotion Does](#-what-promotion-does)
- [Launch the Promotion Wizard](#-launch-the-promotion-wizard)
- [Walk Through Every Screen](#-walk-through-every-screen)
- [The Restart and First Domain Login](#-the-restart-and-first-domain-login)
- [Verify Active Directory is Working](#-verify-active-directory-is-working)
- [Understand What Was Created](#-understand-what-was-created)
- [Take a Snapshot](#-take-a-snapshot)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)

---

## 🧠 What Promotion Does

**Promotion** is the process of configuring a server as a Domain Controller. When you promote DC01 it performs the following actions automatically:

```
┌─────────────────────────────────────────────────────────────┐
│                  What Promotion Creates                      │
│                                                             │
│   🌲 New Forest         A new AD forest called Lab.local    │
│                                                             │
│   📂 AD Database        NTDS.dit — stores all AD objects    │
│                         Location: C:\Windows\NTDS\          │
│                                                             │
│   📁 SYSVOL Share       Stores Group Policy files and       │
│                         scripts replicated across DCs       │
│                         Location: C:\Windows\SYSVOL\        │
│                                                             │
│   🌐 DNS Zone           Forward lookup zone for Lab.local   │
│                         DC01 registers its own A record     │
│                                                             │
│   🔐 Kerberos KDC       Key Distribution Centre for         │
│                         issuing authentication tickets      │
│                                                             │
│   👤 Domain Admin       Administrator becomes               │
│                         LAB\Administrator (domain account)  │
└─────────────────────────────────────────────────────────────┘
```

After promotion, logging in as Administrator means logging in as a **domain administrator** — the most powerful account in the entire domain.

---

## 🚀 Launch the Promotion Wizard

There are two ways to start the promotion wizard:

### Method 1 — Yellow Flag Notification (Recommended)

In Server Manager click the **yellow flag icon** in the top navigation bar.

A dropdown appears showing:

```
⚠️  Configuration required for Active Directory Domain Services at DC01
    🔗 Promote this server to a domain controller
```

Click **"Promote this server to a domain controller"**.

---

### Method 2 — Via the AD DS Section

In Server Manager click **"AD DS"** in the left panel.

In the main panel you will see DC01 listed with a warning. In the **Tasks** dropdown at the top right of that panel, click **"Promote this server to a domain controller"**.

---

The **Active Directory Domain Services Configuration Wizard** opens.

---

## 📋 Walk Through Every Screen

### Screen 1 — Deployment Configuration ⬅ Critical Screen

This screen asks what kind of AD deployment you are creating.

Three options are shown:

| Option | When to Use |
|--------|------------|
| Add a domain controller to an existing domain | When joining a second DC to an existing domain |
| Add a new domain to an existing forest | When creating a child domain (e.g. uk.company.local) |
| **Add a new forest** | **Building a brand new domain from scratch ✅** |

**Select "Add a new forest".**

In the **"Root domain name"** field that appears, type:

```
Lab.local
```

> 💡 **Why Lab.local?**
> `.local` is a private domain suffix used for internal networks.
> It is never registered on the public internet so it is safe for lab use.
> The name `Lab` clearly identifies this as a lab environment.
> In a real business you might see names like `contoso.local` or `corp.company.com`.

Click **Next >**

---

### Screen 2 — Domain Controller Options ⬅ Important Screen

| Setting | Value | Notes |
|---------|-------|-------|
| Forest functional level | Windows Server 2016 | Maximises compatibility — leave as default |
| Domain functional level | Windows Server 2016 | Maximises compatibility — leave as default |
| Domain Name System (DNS) server | ✅ Ticked | DNS installs alongside AD — required |
| Global Catalog (GC) | ✅ Ticked | Makes this DC searchable — required for first DC |
| Read only domain controller (RODC) | ❌ Unticked | Leave unticked — RODCs are for branch offices |

**Directory Services Restore Mode (DSRM) Password:**

This is a special recovery password used only if Active Directory needs to be restored in disaster recovery mode. It is completely separate from your normal Administrator password.

Set a strong DSRM password and write it down:

```
Example: DSRM@Lab2024!
```

> 🔒 **Write this password down and keep it safe.**
> You will rarely need it — but if Active Directory ever becomes corrupted
> and you need to restore from backup, this is the only way in.
> Without it, disaster recovery is impossible.

Type the DSRM password, press Tab, confirm it, then click **Next >**

---

### Screen 3 — DNS Options

You will see a yellow warning:

```
⚠️  A delegation for this DNS server cannot be created because the
    authoritative parent zone cannot be found or it does not run
    Windows DNS Server.
```

**This warning is completely normal and expected in a homelab.**

It means there is no parent DNS zone above `Lab.local` — which is correct because we are building a private internal domain, not a subdomain of a public domain. You can safely ignore this warning.

Click **Next >**

---

### Screen 4 — Additional Options

The wizard automatically suggests a **NetBIOS domain name** based on the domain name you entered:

```
NetBIOS domain name:  LAB
```

**Leave this as LAB** — do not change it.

> 💡 **What is NetBIOS?**
> NetBIOS is an older naming system used for backwards compatibility.
> The NetBIOS name is the short version of your domain that appears
> before the backslash in usernames — for example `LAB\Administrator`.
> When you log into the domain you will use `LAB\username` format.

Wait a few seconds for the wizard to finish validating the name, then click **Next >**

---

### Screen 5 — Paths

This screen shows where Active Directory will store its critical files:

| Path | Default Location | Purpose |
|------|-----------------|---------|
| Database folder | `C:\Windows\NTDS` | NTDS.dit — the AD database file |
| Log files folder | `C:\Windows\NTDS` | Transaction logs for database recovery |
| SYSVOL folder | `C:\Windows\SYSVOL` | Group Policy files and login scripts |

**Leave all three paths at their default values.**

> 💡 In production environments these folders are often placed on separate
> dedicated drives for performance and reliability.
> For a homelab the defaults are perfectly appropriate.

Click **Next >**

---

### Screen 6 — Review Options

A summary of all your configuration choices is displayed:

```
The target server will be configured as follows:

Configure this server as the first Active Directory domain
controller in a new forest named "Lab.local".

The NetBIOS name of the domain will be "LAB".

Forest Functional Level:   Windows Server 2016
Domain Functional Level:   Windows Server 2016

Additional Options:
  Global catalog:  Yes
  DNS Server:      Yes
  Create DNS Delegation: No
```

Review that everything matches what you configured.

You can optionally click **"View script"** to see the equivalent PowerShell command that would perform this same promotion — this is excellent for learning:

```powershell
# This is what the GUI wizard does behind the scenes
Install-ADDSForest `
    -DomainName "Lab.local" `
    -DomainNetbiosName "LAB" `
    -ForestMode "WinThreshold" `
    -DomainMode "WinThreshold" `
    -InstallDns:$true `
    -CreateDnsDelegation:$false `
    -DatabasePath "C:\Windows\NTDS" `
    -LogPath "C:\Windows\NTDS" `
    -SysvolPath "C:\Windows\SYSVOL" `
    -NoRebootOnCompletion:$false `
    -Force:$true
```

Click **Next >**

---

### Screen 7 — Prerequisites Check

The wizard runs an automated prerequisites check. This takes 30–60 seconds.

You will see several **yellow warning messages** — these are all normal in a homelab:

```
⚠️  Windows Server 2022 domain controllers have a default for
    the security setting named "Allow cryptography algorithms
    compatible with Windows NT 4.0" that prevents weaker
    cryptography algorithms when establishing security channel
    sessions.

⚠️  A delegation for this DNS server cannot be created.

⚠️  This computer has at least one physical network adapter
    that does not have static IP address(es) assigned.
```

> ✅ **Yellow warnings are acceptable — you can proceed.**
> ❌ **Red errors must be fixed before you can proceed.**

As long as you see the green message at the top:

```
✅ All prerequisite checks passed successfully.
   Click "Install" to begin installation.
```

You are ready to install.

Click **Install**.

---

### Screen 8 — Installation Progress

The promotion process runs automatically:

```
Active Directory Domain Services Installation

Configuring Active Directory Domain Services...

✅ Verifying domain functional level
✅ Creating the NTDS database
✅ Creating the SYSVOL share
✅ Installing the DNS Server role
✅ Creating DNS zones for Lab.local
✅ Configuring Kerberos
✅ Configuring security policies
```

**This takes approximately 3–8 minutes.**

At the end of the installation the server will display:

```
This server was successfully configured as a domain controller.
The server will automatically restart in a few seconds.
```

> ⚠️ **Do NOT power off or reset the VM during promotion.**
> Interrupting this process can corrupt the Active Directory database.
> Let the automatic restart happen naturally.

---

## 🔐 The Restart and First Domain Login

After the automatic restart the login screen looks noticeably different.

### What Changed on the Login Screen

```
Before promotion:                After promotion:
┌─────────────────────┐         ┌─────────────────────┐
│                     │         │                     │
│  Administrator      │   →     │  LAB\Administrator  │
│  (local account)    │         │  (domain account)   │
│                     │         │                     │
└─────────────────────└         └─────────────────────┘
```

The username now shows **LAB\Administrator** — this means you are logging in as the domain administrator, not just a local account. The domain is live.

Log in with:
```
Username:  LAB\Administrator
Password:  Lab@Admin2024!   (your Administrator password from Step 04)
```

Server Manager will open automatically and you may see it take slightly longer to load than before — it is now initialising all the AD DS and DNS services.

---

## ✅ Verify Active Directory is Working

Once logged in, run these verification commands in **PowerShell as Administrator**:

### Test 1 — Confirm Domain Details

```powershell
Get-ADDomain
```

Expected output confirms your domain is live:

```
AllowedDNSSuffixes                 : {}
ChildDomains                       : {}
DNSRoot                            : Lab.local
DomainMode                         : WinThreshold
DomainSID                          : S-1-5-21-XXXXXXXXXX
Forest                             : Lab.local
InfrastructureMaster               : DC01.Lab.local
Name                               : Lab
NetBIOSName                        : LAB
PDCEmulator                        : DC01.Lab.local
RIDMaster                          : DC01.Lab.local
```

Key fields to confirm:
- `DNSRoot: Lab.local` ✅
- `NetBIOSName: LAB` ✅
- `PDCEmulator: DC01.Lab.local` ✅

---

### Test 2 — Confirm DC01 is Listed as a Domain Controller

```powershell
Get-ADDomainController -Filter *
```

Expected output:

```
ComputerObjectDN : CN=DC01,OU=Domain Controllers,DC=Lab,DC=local
DefaultPartition : DC=Lab,DC=local
Domain           : Lab.local
Enabled          : True
Forest           : Lab.local
HostName         : DC01.Lab.local
IPv4Address      : 192.168.100.10
IsGlobalCatalog  : True
IsReadOnly       : False
Name             : DC01
OperatingSystem  : Windows Server 2022 Standard Evaluation
```

---

### Test 3 — Confirm DNS is Resolving the Domain

```powershell
Resolve-DnsName -Name "Lab.local"
```

Expected output:

```
Name       Type  TTL  Section  IPAddress
----       ----  ---  -------  ---------
Lab.local  A     600  Answer   192.168.100.10
```

`Lab.local` resolves to `192.168.100.10` — DNS is working correctly. ✅

---

### Test 4 — Confirm SYSVOL and NETLOGON Shares Exist

```powershell
Get-SmbShare | Where-Object { $_.Name -in @("SYSVOL","NETLOGON") }
```

Expected output:

```
Name     ScopeName  Path                                    Description
----     ---------  ----                                    -----------
NETLOGON *          C:\Windows\SYSVOL\sysvol\Lab.local\...  Logon server share
SYSVOL   *          C:\Windows\SYSVOL\sysvol               Logon server share
```

Both shares present confirms Group Policy delivery infrastructure is ready. ✅

---

### Test 5 — Open Active Directory Users and Computers

In Server Manager click **Tools → Active Directory Users and Computers**.

You should see:

```
Active Directory Users and Computers
└── Lab.local
    ├── Builtin
    ├── Computers
    ├── Domain Controllers
    │   └── DC01
    ├── ForeignSecurityPrincipals
    └── Users
```

DC01 is listed under **Domain Controllers** — confirming successful promotion. ✅

---

## 📖 Understand What Was Created

After promotion several default containers and groups exist in Active Directory:

### Default Containers

| Container | Purpose |
|-----------|---------|
| Builtin | Built-in local groups (Administrators, Users, etc.) |
| Computers | Default location for newly joined domain computers |
| Domain Controllers | Contains all DC objects in the domain |
| ForeignSecurityPrincipals | External domain objects |
| Users | Default location for new users and built-in accounts |

> 💡 **Note:** In the next step (08) we create our own **Organisational Units (OUs)**
> with a proper structure. We do not add our users and computers to these
> default containers — we use our custom OUs instead.

### Default Domain Groups Created

| Group | Purpose |
|-------|---------|
| Domain Admins | Members have full control over the entire domain |
| Domain Users | All domain users are automatically members |
| Domain Computers | All domain computers are automatically members |
| Enterprise Admins | Full control over the entire forest |
| Schema Admins | Can modify the AD schema |
| Group Policy Creator Owners | Can create and manage GPOs |

---

## 📸 Take a Snapshot

This is the most important snapshot in the entire lab. DC01 is now a fully functioning Domain Controller with the Lab.local domain live.

In VirtualBox go to:
```
Machine → Take Snapshot
```

Name it:
```
DC01 - Domain Controller Live - Lab.local Created
```

Click **OK**.

> 🔒 **This snapshot is your ultimate safety net.**
> If anything goes seriously wrong in the rest of the lab —
> bulk user creation, GPO configuration, workstation joining —
> you can roll back to this snapshot and have a perfectly clean
> Domain Controller with a working domain to start from.

---

## 🛠️ Troubleshooting

### Issue: "The forest functional level must be Windows Server 2003 or higher" error

An older functional level was selected accidentally.

**Fix:**
Go back to Screen 2 — Domain Controller Options and ensure both Forest and Domain functional levels are set to **Windows Server 2016**.

---

### Issue: Promotion fails with "DNS server could not be contacted"

DNS validation is failing during prerequisites check.

**Fix:**
```powershell
# Verify the static IP is set correctly on the internal adapter
Get-NetIPAddress -InterfaceAlias "Ethernet 2"

# Verify DNS is pointing to loopback
Get-DnsClientServerAddress -InterfaceAlias "Ethernet 2"
```

Both must be correct before retrying promotion.

---

### Issue: After restart the login screen still shows just "Administrator" without LAB\ prefix

The promotion may not have completed correctly, or the login screen is showing a cached local login.

**Fix:**
1. Click **"Other user"** on the login screen
2. Type `LAB\Administrator` manually
3. Enter your password
4. If this fails, open PowerShell and run:

```powershell
# Check if AD DS services are running
Get-Service -Name ADWS, NTDS, DNS, KDC, Netlogon | Select-Object Name, Status
```

All five services should show **Running**.

---

### Issue: Get-ADDomain returns "The term 'Get-ADDomain' is not recognised"

The Active Directory PowerShell module is not loaded.

**Fix:**
```powershell
# Import the module manually
Import-Module ActiveDirectory

# Retry
Get-ADDomain
```

---

### Issue: SYSVOL share is missing from Get-SmbShare output

SYSVOL replication may still be initialising after promotion.

**Fix:**
```powershell
# Check SYSVOL initialisation status
Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\Netlogon\Parameters" `
    -Name SysvolReady
```

If `SysvolReady` returns `0` wait 2–3 minutes and check again.
It should return `1` once SYSVOL is fully initialised.

---

### Issue: Active Directory Users and Computers will not open

The RSAT tools may not have installed correctly.

**Fix:**
```powershell
# Reinstall AD DS management tools
Install-WindowsFeature -Name RSAT-ADDS -IncludeAllSubFeature

# Then open ADUC from PowerShell
dsa.msc
```

---

## ✅ Verification Checklist

```
Promotion Wizard
[ ] Selected "Add a new forest" on Screen 1
[ ] Root domain name set to Lab.local
[ ] DNS Server ticked on Screen 2
[ ] Global Catalog ticked on Screen 2
[ ] DSRM password set and written down safely
[ ] NetBIOS name confirmed as LAB on Screen 4
[ ] All paths left at defaults on Screen 5
[ ] Prerequisites check showed green — no red errors
[ ] Installation completed and server restarted automatically

First Domain Login
[ ] Login screen shows LAB\Administrator
[ ] Logged in successfully with Administrator password
[ ] Server Manager opened after login

PowerShell Verification
[ ] Get-ADDomain returns DNSRoot: Lab.local
[ ] Get-ADDomain returns NetBIOSName: LAB
[ ] Get-ADDomainController shows DC01 with IP 192.168.100.10
[ ] Resolve-DnsName Lab.local returns 192.168.100.10
[ ] SYSVOL and NETLOGON shares confirmed present
[ ] Active Directory Users and Computers opens successfully
[ ] DC01 visible under Domain Controllers OU in ADUC

Snapshot
[ ] Snapshot taken named "DC01 - Domain Controller Live - Lab.local Created"

Ready to proceed
[ ] Lab.local domain is live
[ ] DC01 is the Primary Domain Controller
[ ] DNS is resolving Lab.local correctly
[ ] All five AD services (ADWS, NTDS, DNS, KDC, Netlogon) are running
```

---

## ➡️ Next Step

The Lab.local domain is live and DC01 is a fully functioning Domain Controller.
Now we build a proper Organisational Unit structure to organise
users, computers, and groups.

**[08 - Create Organisational Unit Structure →](../08%20-%20Create%20OU%20Structure/README.md)**

---

## ⬅️ Previous Step

**[← 06 - Install Active Directory Domain Services](../06%20-%20Install%20Active%20Directory/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
