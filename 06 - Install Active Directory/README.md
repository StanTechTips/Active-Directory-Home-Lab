# 06 - Install Active Directory Domain Services

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Intermediate-orange?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/VM-DC01-purple?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Role-AD_DS-red?style=for-the-badge&logo=windows)

> Installing the AD DS role is the first half of creating a Domain Controller.
> This step installs the software and management tools onto the server.
> The second half — actually creating the domain — happens in the next step.
> Think of this as laying the foundation before building the house.

---

## 📑 Table of Contents

- [What is AD DS?](#-what-is-ad-ds)
- [What Gets Installed in This Step](#-what-gets-installed-in-this-step)
- [Open the Add Roles and Features Wizard](#-open-the-add-roles-and-features-wizard)
- [Walk Through Every Screen](#-walk-through-every-screen)
- [What Happens After Installation](#-what-happens-after-installation)
- [Verify the Installation via PowerShell](#-verify-the-installation-via-powershell)
- [Take a Snapshot](#-take-a-snapshot)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)

---

## 🧠 What is AD DS?

**Active Directory Domain Services (AD DS)** is the core Windows Server role that enables a server to become a Domain Controller. It provides a centralised database and set of services for managing users, computers, and policies across a network.

```
┌─────────────────────────────────────────────────────────────┐
│               Active Directory Domain Services               │
│                                                             │
│   📂 Directory Database (NTDS.dit)                          │
│      Stores every user, computer, group, and policy         │
│      in the domain in a structured, searchable database     │
│                                                             │
│   🔐 Kerberos Authentication                                │
│      Issues tickets that prove who a user is                │
│      Replaces the need for passwords on every resource      │
│                                                             │
│   🌐 DNS Integration                                        │
│      Registers domain records so computers can find the DC  │
│      Lab.local resolves to 192.168.100.10                   │
│                                                             │
│   📋 LDAP Directory Service                                 │
│      Allows applications to query the directory             │
│      Used by everything from login screens to web apps      │
│                                                             │
│   📜 Group Policy Infrastructure                            │
│      SYSVOL share stores GPO files                          │
│      Delivered to domain computers at login and on schedule │
└─────────────────────────────────────────────────────────────┘
```

---

## 📦 What Gets Installed in This Step

When you install the AD DS role, Windows Server installs several components automatically:

| Component | Description |
|-----------|-------------|
| AD DS Binaries | The core Active Directory service files |
| DNS Server | Integrated DNS for resolving domain names |
| AD DS Management Tools | ADUC, ADAC, Group Policy Management Console |
| PowerShell AD Module | `Get-ADUser`, `New-ADUser`, and all AD cmdlets |
| RSAT Tools | Remote Server Administration Tools for managing AD |

> 💡 **Two steps to become a Domain Controller:**
> 1. **Install the role** (this step) — copies files and tools to the server
> 2. **Promote the server** (next step) — actually creates the domain and domain database
>
> After this step the server is still just a member server with AD tools installed.
> It only becomes a Domain Controller after promotion in Step 07.

---

## 🚀 Open the Add Roles and Features Wizard

### Method 1 — Via Server Manager Dashboard

**1.** Open **Server Manager** — it opens automatically, or click the flag icon in the taskbar.

**2.** On the Dashboard click **"Add roles and features"** — it is option 2 in the Quick Start panel.

---

### Method 2 — Via the Manage Menu

**1.** In Server Manager click **"Manage"** in the top menu bar.

**2.** Click **"Add Roles and Features"** from the dropdown.

---

The **Add Roles and Features Wizard** opens. You are ready to begin.

---

## 📋 Walk Through Every Screen

### Screen 1 — Before You Begin

This is an information screen reminding you to:
- Have a strong Administrator password ✅ Done in Step 04
- Have static IP addresses configured ✅ Done in Step 05
- Have Windows Update run *(optional for the lab)*

You can tick **"Skip this page by default"** if you do not want to see it in future.

Click **Next >**

---

### Screen 2 — Installation Type

Two options are shown:

| Option | Description | Choose? |
|--------|-------------|---------|
| **Role-based or feature-based installation** | **Install roles on this specific server** | **✅ Yes** |
| Remote Desktop Services installation | For VDI and session-based deployments | ❌ No |

**Select "Role-based or feature-based installation"** — it should already be selected by default.

Click **Next >**

---

### Screen 3 — Server Selection

You will see a server pool containing one server:

```
Server Pool
┌──────────────────────────────────────────────────────┐
│  ✅  DC01    192.168.100.10    Windows Server 2022    │
└──────────────────────────────────────────────────────┘
```

**DC01 should already be highlighted.** If not, click on it to select it.

Confirm the destination server shown at the top right reads **DC01**.

Click **Next >**

---

### Screen 4 — Server Roles ⬅ The Important Screen

This is the main screen where you select what to install. You will see a long alphabetical list of available roles.

**Scroll down the list and find:**

```
☐ Active Directory Domain Services
```

**Tick the checkbox** next to it.

A popup window immediately appears:

```
┌──────────────────────────────────────────────────────────┐
│  Add features that are required for                      │
│  Active Directory Domain Services?                       │
│                                                          │
│  You cannot install Active Directory Domain Services     │
│  unless the following role services or features are      │
│  also installed:                                         │
│                                                          │
│  [Remote Server Administration Tools]                    │
│    [Role Administration Tools]                           │
│      [AD DS and AD LDS Tools]                            │
│                                                          │
│           [Add Features]      [Cancel]                   │
└──────────────────────────────────────────────────────────┘
```

> ⚠️ **Click "Add Features" — do NOT click Cancel.**
> These are the management tools (ADUC, GPMC, DNS Manager) that you need
> to manage Active Directory. Without them you cannot administer the domain.

After clicking Add Features, the **Active Directory Domain Services** checkbox is now ticked with a blue checkmark.

Click **Next >**

---

### Screen 5 — Features

This screen shows additional Windows features you can install. The required features were already added in the previous popup.

**Do not change anything on this screen.**

Click **Next >**

---

### Screen 6 — Active Directory Domain Services

This is an information screen explaining what AD DS does and important notes about the installation.

Key note shown on this screen:
> *"When you promote this server to a domain controller, the DNS Server role is installed automatically if it is not already installed."*

This confirms DNS will be installed alongside AD DS during promotion — you do not need to install it separately.

Click **Next >**

---

### Screen 7 — Confirmation

You will see a summary of everything that will be installed:

```
The following roles, role services, or features will be installed:

Remote Server Administration Tools
  Role Administration Tools
    AD DS and AD LDS Tools
      Active Directory Administrative Centre
      AD DS Snap-Ins and Command-Line Tools
      Active Directory Module for Windows PowerShell

Active Directory Domain Services
```

> ⚠️ **Do NOT tick "Restart the destination server automatically if required"**
> AD DS installation does not require a restart at this stage.
> The restart only happens after promotion in the next step.
> You want control over when the server restarts.

Click **Install**.

---

### Screen 8 — Installation Progress

A progress bar appears showing the installation in real time:

```
Installation started on DC01

[ ████████████████░░░░ ]  Installing...

Active Directory Domain Services
Remote Server Administration Tools
AD DS and AD LDS Tools
```

**This takes approximately 2–5 minutes.**

> ⚠️ **Do not close this window while installation is in progress.**
> You can click the progress bar to see more detail, but do not close or cancel.
> If the window is closed accidentally the installation may be incomplete.

---

### Screen 9 — Installation Results

When complete the screen will show:

```
Installation succeeded on DC01.

Active Directory Domain Services
  One or more roles, role services, or features require
  the server to be restarted. View installation progress
  or open the Close button below.

⚠️ Configuration required. Installation succeeded on DC01.
   Promote this server to a domain controller
```

You will see a blue hyperlink that reads:

```
🔗 Promote this server to a domain controller
```

> 🛑 **Do NOT click this link yet.**
> Do NOT close the wizard either.
> First read the section below — then come back and click the link in the next step (07).

Click **Close** for now. The link will also be available via the notification flag in Server Manager.

---

## 📋 What Happens After Installation

After clicking Close, look at **Server Manager**. You will notice two changes:

### Change 1 — Yellow Warning Flag

A yellow flag with an exclamation mark appears in the top navigation bar of Server Manager next to the flag icon. This is the **Post-deployment Configuration notification**.

Clicking it reveals:

```
⚠️  Configuration required for Active Directory Domain Services at DC01
    Promote this server to a domain controller
```

This is your reminder that the role is installed but the server is not yet a Domain Controller. You will click this in the next step.

### Change 2 — AD DS Appears in Left Panel

In the left panel of Server Manager you will now see **"AD DS"** listed as a new section. Clicking it shows DC01 listed as a server with the AD DS role — but with a warning because it is not yet promoted.

---

## 🔍 Verify the Installation via PowerShell

Confirm AD DS installed correctly before moving to promotion.

```powershell
# Verify the AD DS role is installed
Get-WindowsFeature -Name AD-Domain-Services
```

Expected output:

```
Display Name                           Name                Install State
------------                           ----                -------------
[X] Active Directory Domain Services  AD-Domain-Services  Installed
```

The `[X]` and `Installed` confirm the role is ready.

```powershell
# Verify the AD PowerShell module is available
Get-Module -ListAvailable -Name ActiveDirectory
```

Expected output shows the ActiveDirectory module is available — this means all the `Get-ADUser`, `New-ADUser`, and other AD cmdlets are ready to use after promotion.

```powershell
# Verify DNS Server role was included
Get-WindowsFeature -Name DNS
```

Expected output:

```
Display Name   Name  Install State
------------   ----  -------------
[X] DNS Server DNS   Installed
```

---

## 📸 Take a Snapshot

The AD DS role is installed and verified. This is an excellent save point.

In VirtualBox go to:
```
Machine → Take Snapshot
```

Name it:
```
DC01 - AD DS Role Installed - Pre Promotion
```

Click **OK**.

> If the domain promotion in the next step fails or produces unexpected results,
> you can roll back to this snapshot and retry promotion with a clean slate —
> without having to reinstall the entire AD DS role.

---

## 🛠️ Troubleshooting

### Issue: "Active Directory Domain Services" is greyed out in the roles list

Another required role or feature is missing or the server does not meet the prerequisites.

**Fix:**
1. Ensure DC01 has at least **512 MB RAM** (4096 MB is our configuration — this should never be an issue)
2. Ensure the server has a **static IP** configured — AD DS installation checks for this
3. Try scrolling up in the roles list to see if there is an error message at the top

---

### Issue: The "Add Features" popup did not appear when ticking AD DS

You may have accidentally dismissed it or it appeared behind another window.

**Fix:**
1. Untick the **Active Directory Domain Services** checkbox
2. Tick it again
3. The popup should reappear — if it does not, look behind other windows
4. If still missing, close the wizard and reopen it via Manage → Add Roles and Features

---

### Issue: Installation failed with an error about insufficient disk space

The virtual hard disk does not have enough free space.

**Fix:**
```powershell
# Check available disk space
Get-PSDrive C | Select-Object Used, Free
```

AD DS requires approximately **500 MB** of free space. If space is critically low, consider expanding the virtual hard disk in VirtualBox settings (with the VM powered off).

---

### Issue: Installation stuck at the same percentage for more than 15 minutes

This is usually a performance issue on the host PC.

**Fix:**
1. Check host PC CPU and RAM usage in Task Manager
2. Close any unnecessary applications on your host
3. If the installation genuinely appears frozen (no disk activity), close the wizard and run:

```powershell
# Check if the feature is actually installing in the background
Get-WindowsFeature -Name AD-Domain-Services
```

If it shows `Install State: InstallPending` it is still installing. Wait another 5 minutes.

---

### Issue: Get-WindowsFeature shows "Available" not "Installed"

The installation did not complete successfully.

**Fix:**
```powershell
# Reinstall via PowerShell directly
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools -IncludeAllSubFeature

# Verify again
Get-WindowsFeature -Name AD-Domain-Services
```

---

## ✅ Verification Checklist

```
Wizard Navigation
[ ] Completed all 9 screens of the Add Roles and Features Wizard
[ ] "Add Features" popup was accepted when ticking AD DS
[ ] Did NOT tick "Restart automatically"
[ ] Installation completed with "Installation succeeded" message
[ ] Wizard closed — NOT promoted yet

Server Manager Changes
[ ] Yellow warning flag visible in Server Manager top bar
[ ] "Promote this server to a domain controller" link visible
[ ] AD DS section now appears in Server Manager left panel

PowerShell Verification
[ ] Get-WindowsFeature AD-Domain-Services shows Install State: Installed
[ ] Get-WindowsFeature DNS shows Install State: Installed
[ ] ActiveDirectory PowerShell module is available

Snapshot
[ ] Snapshot taken named "DC01 - AD DS Role Installed - Pre Promotion"

Ready to proceed
[ ] AD DS role is installed and verified
[ ] Server has NOT been promoted yet — that is the next step
[ ] Yellow warning flag is visible and ready to use in next step
```

---

## ➡️ Next Step

The AD DS role is installed. Now we promote DC01 to a Domain Controller
and create the **Lab.local** domain.

**[07 - Promote to Domain Controller →](../07%20-%20Promote%20to%20Domain%20Controller/README.md)**

---

## ⬅️ Previous Step

**[← 05 - Configure DC01](../05%20-%20Congigure%20DC01/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
