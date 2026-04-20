# 03 - Create the Domain Controller VM (DC01)

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Beginner-blue?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/VM-DC01-purple?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/OS-Windows_Server_2022-0078D6?style=for-the-badge&logo=windows)

> DC01 is the heart of the entire lab. Every other component — users, computers,
> group policies, and DNS — depends on this VM being correctly configured.
> Take your time here and follow every step precisely.

---

## 📑 Table of Contents

- [What is a Domain Controller?](#-what-is-a-domain-controller)
- [VM Specifications](#-vm-specifications)
- [Create the VM in VirtualBox](#-create-the-vm-in-virtualbox)
- [Attach the Windows Server 2022 ISO](#-attach-the-windows-server-2022-iso)
- [Configure VM Settings](#-configure-vm-settings)
- [Take Your First Snapshot](#-take-your-first-snapshot)
- [VM Architecture Overview](#-vm-architecture-overview)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)

---

## 🧠 What is a Domain Controller?

A **Domain Controller (DC)** is a server that runs **Active Directory Domain Services (AD DS)**.
It is the central authority in a Windows domain environment — responsible for:

```
┌─────────────────────────────────────────────────────────────┐
│                  Domain Controller (DC01)                    │
│                                                             │
│   🔐 Authentication    Verifies who users are               │
│   📋 Authorisation     Controls what users can access       │
│   📖 Directory         Stores all users, computers, groups  │
│   🌐 DNS               Resolves Lab.local domain names      │
│   📜 Group Policy      Pushes settings to all domain PCs    │
└─────────────────────────────────────────────────────────────┘
```

Every time a domain user logs into a computer, the DC verifies their credentials using **Kerberos authentication**. Without the DC, no one can log into the domain.

> 💡 In real enterprise environments, companies typically have **two or more** Domain Controllers for redundancy. In this lab we use one — which is perfectly fine for learning purposes.

---

## 📊 VM Specifications

These are the exact specifications used to build DC01 in this lab:

| Setting | Value | Reason |
|---------|-------|--------|
| VM Name | `DC01` | Clear, consistent naming convention |
| OS Type | Microsoft Windows | |
| OS Version | Windows 2022 (64-bit) | Matches the Server 2022 ISO |
| RAM | 4096 MB (4 GB) | Minimum for comfortable Server 2022 operation |
| CPUs | 2 virtual CPUs | Enough for all server roles in this lab |
| Video Memory | 128 MB | Required for full Desktop Experience GUI |
| Hard Disk | 60 GB dynamically allocated | Plenty for Server OS + AD database + logs |
| Adapter 1 | NAT | Internet access for downloads during setup |
| Adapter 2 | Internal Network — `LabNet` | Private domain network for communicating with WS01 |
| EFI | Enabled | Modern boot standard |
| TPM | 2.0 | Required for Windows Server 2022 |

---

## 🖥️ Create the VM in VirtualBox

### Step 1 — Open the New VM Wizard

In VirtualBox Manager click the **"New"** button at the top of the window.

The **"Create Virtual Machine"** wizard opens.

---

### Step 2 — Name and Operating System Screen

Fill in the fields exactly as shown below:

| Field | Value | Important Note |
|-------|-------|----------------|
| Name | `DC01` | |
| Folder | `C:\LabFiles\VMs\` | Or wherever you set your default VM folder |
| ISO Image | **Leave this blank** | ⚠️ Do NOT select the ISO here |
| Type | `Microsoft Windows` | Auto-fills when you type the name |
| Version | `Windows 2022 (64-bit)` | Select from the dropdown |

> ⛔ **Why leave the ISO blank?**
> If you select the ISO here VirtualBox activates its **Unattended Installer**,
> which causes this error when booting:
> ```
> Windows cannot find the Microsoft Software License Terms.
> Make sure the installation sources are valid and restart the installation.
> ```
> Always attach the ISO manually **after** the VM is created.
> This is the single most common mistake when building VMs in this lab.

Click **Next**.

---

### Step 3 — Hardware Screen

| Setting | Value |
|---------|-------|
| Base Memory | `4096` MB |
| Processors | `2` |
| Enable EFI | ✅ Tick this |

Click **Next**.

---

### Step 4 — Virtual Hard Disk Screen

| Setting | Value |
|---------|-------|
| Create a Virtual Hard Disk Now | ✅ Selected |
| Disk Size | `60 GB` |
| Pre-allocate Full Size | ❌ Leave unticked (dynamically allocated) |

Click **Next** then **Finish**.

DC01 now appears in your VM list on the left panel — powered off.

---

## 💿 Attach the Windows Server 2022 ISO

Now that the VM exists, attach the ISO manually so it boots correctly.

### Step 1 — Open Storage Settings

Click **DC01** in the VM list to select it.

Click **Settings** (the orange gear icon) → click **Storage** in the left panel.

### Step 2 — Find the Empty Optical Drive

Under **"Controller: SATA"** you will see a disc icon with the label **"Empty"**.

Click on the **"Empty"** line to select it.

### Step 3 — Attach the ISO

On the right side of the window, find **"Optical Drive"** and click the small **blue disc icon** next to it.

Select **"Choose a disk file..."** from the dropdown.

Browse to your Downloads folder (or wherever you saved it) and select:

```
SERVER_EVAL_x64FRE_en-us.iso
```

Click **Open**.

### Step 4 — Confirm

The optical drive entry should now show the ISO filename instead of "Empty":

```
Before:  💿 Empty
After:   💿 SERVER_EVAL_x64FRE_en-us.iso   ✅
```

---

## ⚙️ Configure VM Settings

Before booting DC01, configure the remaining settings. Click **Settings** on DC01.

### System → Motherboard Tab

| Setting | Value |
|---------|-------|
| Boot Order — Optical | ✅ Ticked and at the top |
| Boot Order — Hard Disk | ✅ Ticked and second |
| Boot Order — Floppy | ❌ Untick this |
| Chipset | PIIX3 |
| TPM | 2.0 |
| Enable EFI | ✅ Ticked |
| Pointing Device | USB Tablet |

### System → Processor Tab

| Setting | Value |
|---------|-------|
| Processor(s) | 2 |
| Enable PAE/NX | ✅ Ticked |

### Display → Screen Tab

| Setting | Value |
|---------|-------|
| Video Memory | `128` MB |
| Graphics Controller | VMSVGA |
| Enable 3D Acceleration | ✅ Tick if available |

### Network → Adapter 1 Tab

| Setting | Value |
|---------|-------|
| Enable Network Adapter | ✅ |
| Attached to | `NAT` |

### Network → Adapter 2 Tab

| Setting | Value |
|---------|-------|
| Enable Network Adapter | ✅ |
| Attached to | `Internal Network` |
| Name | `LabNet` |

> ⚠️ **The Internal Network name is critical.**
> Both DC01 and WS01 must use the **exact same name** — `LabNet`.
> The name is case-sensitive. `LabNet` ≠ `labnet` ≠ `Labnet`.
> If the names don't match the VMs will be on completely separate virtual networks
> and will not be able to communicate with each other.

Click **OK** to save all settings.

---

## 📸 Take Your First Snapshot

Snapshots are one of the most powerful features in VirtualBox.
They let you save the exact state of a VM and roll back to it instantly if something goes wrong.

### Create the Snapshot

With DC01 selected and **powered off**, go to:

```
Machine → Take Snapshot
```

Or click the **Snapshots** button and then the camera icon.

Name it:

```
DC01 - Clean VM - Pre OS Install
```

Click **OK**.

> 💡 **Snapshot strategy for this lab:**
> Take a snapshot at every major milestone. Good snapshot points are:
> - Before the OS installation ← you are here now
> - After Windows Server installs successfully
> - After renaming to DC01 and setting static IP
> - After Active Directory is installed
> - After 110 users are created
>
> Storage is cheap — snapshots save enormous amounts of time when troubleshooting.

---

## 🗺️ VM Architecture Overview

Here is how DC01 fits into the complete lab network:

```
┌─────────────────────────────────────────────────────────────┐
│                     DC01 — VM Overview                       │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              Windows Server 2022                    │   │
│   │                                                     │   │
│   │  ┌───────────────────┐  ┌───────────────────────┐   │   │
│   │  │   AD DS / DNS     │  │   Group Policy (GPO)  │   │   │
│   │  │   LDAP / Kerberos │  │   SYSVOL / NETLOGON   │   │   │
│   │  └───────────────────┘  └───────────────────────┘   │   │
│   │                                                     │   │
│   │  Hostname:  DC01                                    │   │
│   │  Domain:    Lab.local                               │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
│   Adapter 1 (NAT)              Adapter 2 (Internal)         │
│   IP: 10.0.2.15 (auto)         IP: 192.168.100.10 (static)  │
│   Gateway: 10.0.2.2            DNS: 127.0.0.1               │
│   Purpose: Internet access     Purpose: Domain network       │
│        │                                   │                 │
│        ▼                                   ▼                 │
│   🌐 Internet               LabNet Internal Network          │
│                             192.168.100.0/24                 │
│                                   │                          │
│                                   ▼                          │
│                              WS01 (coming later)             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Troubleshooting

### Issue: "Windows cannot find the Microsoft Software Licence Terms" error on boot

The Unattended Installer was triggered because the ISO was selected during VM creation.

**Fix:**
1. Power off DC01 — right-click → Close → Power Off
2. Delete the VM — right-click → Remove → Delete all files
3. Recreate the VM following this guide exactly — **leave ISO blank during creation**
4. Attach the ISO manually in Storage settings after creation

---

### Issue: DC01 shows a black screen and nothing boots

The VM has no bootable media attached or the boot order is wrong.

**Fix:**
1. Power off DC01
2. Go to Settings → Storage — confirm the ISO is attached and not showing "Empty"
3. Go to Settings → System → Motherboard — confirm Optical is first in boot order and ticked
4. Start DC01 again — if you see "Press any key to boot from CD/DVD" click inside the VM and press any key immediately

---

### Issue: Cannot enable Adapter 2 in Network settings

VirtualBox may be showing the adapter as greyed out.

**Fix:**
1. Make sure DC01 is completely powered off — not saved, not running
2. Go to Settings → Network → Adapter 2 tab
3. Tick "Enable Network Adapter" first — the other options will then become available
4. Set Attached to: Internal Network and Name: LabNet

---

### Issue: VM was created with wrong RAM or CPU settings

Settings can be changed after creation as long as the VM is powered off.

**Fix:**
1. Power off DC01
2. Go to Settings → System → Motherboard (for RAM) or Processor (for CPUs)
3. Adjust the values
4. Click OK and restart

---

## ✅ Verification Checklist

```
VM Creation
[ ] DC01 appears in the VirtualBox Manager VM list
[ ] ISO was left blank during creation — attached manually afterwards
[ ] Windows Server 2022 ISO is attached in Storage settings
[ ] ISO shows filename — not "Empty"

System Settings
[ ] RAM set to 4096 MB
[ ] CPUs set to 2
[ ] EFI enabled
[ ] TPM set to 2.0
[ ] Video Memory set to 128 MB

Network Settings
[ ] Adapter 1 set to NAT
[ ] Adapter 2 enabled and set to Internal Network
[ ] Adapter 2 name is exactly "LabNet" (capital L, capital N)

Snapshot
[ ] Snapshot taken named "DC01 - Clean VM - Pre OS Install"

Ready to proceed
[ ] DC01 is powered off and ready to boot for the first time
```

---

## ➡️ Next Step

DC01 is built and ready. Time to boot it up and install Windows Server 2022.

**[04 - Install Windows Server 2022 →](../04-Install-Windows-Server-2022/README.md)**

---

## ⬅️ Previous Step

**[← 02 - Install Oracle VirtualBox](../02-Install-Oracle-VirtualBox/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
