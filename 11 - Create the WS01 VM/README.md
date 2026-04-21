# 11 - Create the Windows 11 Workstation VM (WS01)

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Beginner-blue?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/VM-WS01-blue?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/OS-Windows_11_Pro-0078D6?style=for-the-badge&logo=windows)

> WS01 is the domain workstation — the end-user machine that will join
> Lab.local and be managed by Group Policy from DC01.
> Building this VM correctly from the start avoids the most common
> issues people encounter — black screens, boot loops, and ISO errors.
> Follow every step precisely and the Windows 11 installer will load first time.

---

## 📑 Table of Contents

- [What WS01 Represents](#-what-ws01-represents)
- [VM Specifications](#-vm-specifications)
- [Create the VM in VirtualBox](#-create-the-vm-in-virtualbox)
- [Configure VM Settings](#-configure-vm-settings)
- [Attach the Windows 11 ISO](#-attach-the-windows-11-iso)
- [Common Boot Issues and Fixes](#-common-boot-issues-and-fixes)
- [Verify WS01 is Ready to Boot](#-verify-ws01-is-ready-to-boot)
- [Take a Snapshot](#-take-a-snapshot)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)

---

## 🧠 What WS01 Represents

In a real enterprise environment, **WS01** represents a typical employee workstation —
the kind of machine that sits on every desk in every office.

```
┌─────────────────────────────────────────────────────────────┐
│                  WS01 — Role in the Lab                      │
│                                                             │
│   👤 End User Machine     Where domain users log in         │
│                                                             │
│   🔐 Domain Member        Authenticated by DC01             │
│                           via Kerberos                      │
│                                                             │
│   📜 GPO Target           Receives policies pushed          │
│                           from DC01 — wallpaper,            │
│                           USB restrictions, password rules  │
│                                                             │
│   🌐 DNS Client           Uses DC01 as DNS server           │
│                           to resolve Lab.local              │
│                                                             │
│   💼 Real World           In production this would be       │
│      Equivalent           a laptop or desktop running       │
│                           Windows 11 Pro on a desk          │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 VM Specifications

| Setting | Value | Reason |
|---------|-------|--------|
| VM Name | `WS01` | Clean, consistent naming convention |
| OS Type | Microsoft Windows | |
| OS Version | Windows 11 (64-bit) | Client OS for domain workstation |
| ISO Image | **Leave blank during creation** | Attach manually — avoids Unattended Installer |
| RAM | 4096 MB (4 GB) | Windows 11 minimum requirement |
| CPUs | 2 virtual CPUs | Smooth performance for a workstation |
| Video Memory | 128 MB | Full resolution display |
| Hard Disk | 50 GB dynamically allocated | Less than DC01 — no server roles needed |
| EFI | ✅ Enabled | Required by Windows 11 |
| TPM | 2.0 | Required by Windows 11 |
| Secure Boot | ❌ Disabled | Prevents ISO from booting in VirtualBox |
| Adapter 1 | Internal Network — `LabNet` | Must match DC01's internal adapter name exactly |

> ⚠️ **Only one network adapter for WS01.**
> Unlike DC01 which has two adapters (NAT + Internal),
> WS01 only needs one — the Internal Network adapter.
> WS01 does not need internet access for any step in this lab.

---

## 🖥️ Create the VM in VirtualBox

### Step 1 — Open the New VM Wizard

In VirtualBox Manager click the **"New"** button at the top.

---

### Step 2 — Name and Operating System Screen

| Field | Value | Note |
|-------|-------|------|
| Name | `WS01` | |
| Folder | `C:\LabFiles\VMs\` | Or your chosen VM folder |
| ISO Image | **Leave completely blank** | ⚠️ Do NOT select the ISO here |
| Type | `Microsoft Windows` | |
| Version | `Windows 11 (64-bit)` | |

> ⛔ **Critical — Leave the ISO blank.**
> Selecting the ISO here activates VirtualBox's Unattended Installer
> which causes boot failures and licence errors.
> The ISO is attached manually in a later step in this guide.
> This is the single most common mistake when creating WS01.

Click **Next**.

---

### Step 3 — Hardware Screen

| Setting | Value |
|---------|-------|
| Base Memory | `4096` MB |
| Processors | `2` |

Click **Next**.

---

### Step 4 — Virtual Hard Disk Screen

| Setting | Value |
|---------|-------|
| Create a Virtual Hard Disk Now | ✅ Selected |
| Disk Size | `50 GB` |
| Pre-allocate Full Size | ❌ Leave unticked |

Click **Next** → **Finish**.

**WS01 now appears in your VM list — powered off.**

---

## ⚙️ Configure VM Settings

Before attaching the ISO or booting WS01, configure all settings first.
Click **WS01 → Settings**.

---

### System → Motherboard Tab

| Setting | Value | Why |
|---------|-------|-----|
| Base Memory | 4096 MB | Already set — confirm |
| Boot Order — Floppy | ❌ Untick | No floppy drive exists |
| Boot Order — Optical | ✅ Ticked | Needed to boot from ISO |
| Boot Order — Hard Disk | ✅ Ticked | Needed after Windows installs |
| Chipset | PIIX3 | Default — leave as is |
| TPM Version | `2.0` | Windows 11 hardware requirement |
| Enable EFI | ✅ Ticked | Windows 11 requires UEFI |
| Secure Boot | ❌ **Untick this** | Blocks ISO boot in VirtualBox |

> ⚠️ **Secure Boot must be disabled.**
> With Secure Boot enabled VirtualBox cannot boot unsigned boot media
> including the Windows 11 ISO — resulting in a black screen or
> `BdsDxe: failed to start Boot` error.
> Untick it now before any boot attempt.

---

### System → Processor Tab

| Setting | Value |
|---------|-------|
| Processor(s) | 2 |
| Enable PAE/NX | ✅ Ticked |

---

### Display → Screen Tab

| Setting | Value |
|---------|-------|
| Video Memory | `128` MB |
| Graphics Controller | VMSVGA |
| Enable 3D Acceleration | ✅ Tick if available |

---

### Network → Adapter 1 Tab

| Setting | Value |
|---------|-------|
| Enable Network Adapter | ✅ |
| Attached to | `Internal Network` |
| Name | `LabNet` |

> ⚠️ **The network name must be exactly `LabNet`.**
> Open DC01 → Settings → Network → Adapter 2 and confirm the name there.
> Copy it exactly — including capitalisation.
> If the names differ by even one character the VMs will be on
> completely separate virtual networks and cannot communicate.

---

### Confirm No Adapter 2

Click the **Adapter 2 tab** and confirm:

```
Enable Network Adapter:  ❌ Not ticked
```

WS01 only needs one adapter. Adapter 2 should remain disabled.

Click **OK** to save all settings.

---

## 💿 Attach the Windows 11 ISO

Now attach the ISO manually — this is the correct way to do it.

### Step 1 — Open Storage Settings

Click **WS01 → Settings → Storage**.

---

### Step 2 — Locate the Empty Optical Drive

Under **Controller: SATA** you will see:

```
💿 Empty
```

Click on the **Empty** line to select it.

---

### Step 3 — Attach the ISO File

On the right side of the window, next to **Optical Drive**, click the small **blue disc icon**.

Select **"Choose a disk file..."** from the dropdown.

Browse to your Downloads folder and select:

```
Win11_25H2_EnglishInternational_x64_v2.iso
```

Click **Open**.

---

### Step 4 — Confirm the ISO is Attached

The optical drive entry should now show the filename:

```
Before:  💿 Empty
After:   💿 Win11_25H2_EnglishInternational_x64_v2.iso  ✅
```

Click **OK** to close Settings.

---

## 🚀 Verify WS01 is Ready to Boot

Before starting WS01 for the first time, do a final pre-flight check.

### VirtualBox Settings Summary

Go to **WS01 → Settings** and confirm every item below:

```
System → Motherboard
[ ] Floppy unticked in boot order
[ ] Optical ticked and above Hard Disk in boot order
[ ] TPM set to 2.0
[ ] EFI enabled
[ ] Secure Boot DISABLED

System → Processor
[ ] Processors: 2
[ ] PAE/NX enabled

Display → Screen
[ ] Video Memory: 128 MB

Storage
[ ] Windows 11 ISO attached — not showing "Empty"

Network → Adapter 1
[ ] Internal Network selected
[ ] Name: LabNet (matches DC01 exactly)

Network → Adapter 2
[ ] Disabled
```

### Start WS01

Double-click **WS01** in the VirtualBox Manager to start it.

Since the hard disk is completely empty and UEFI can see the ISO,
WS01 will boot directly into the Windows 11 setup without needing
you to press any key.

**Within 10–15 seconds you should see the Windows 11 installation screen.**

> 💡 **If you see a prompt saying "Press any key to boot from CD or DVD":**
> Click inside the VM window immediately and press any key.
> You have approximately 2–3 seconds before it times out.

---

## ⚠️ Common Boot Issues and Fixes

### Black Screen — Nothing Loads

**Cause:** ISO not attached or boot order wrong.

**Fix:**
1. Power off WS01 — right-click → Close → Power Off
2. Go to Settings → Storage — confirm ISO filename is showing (not "Empty")
3. Go to Settings → System → Motherboard — confirm Optical is ticked
4. With UEFI and an empty hard disk, WS01 will boot the ISO automatically
5. Start WS01 again

---

### `BdsDxe: failed to start Boot` Error

**Cause:** Secure Boot is blocking the ISO or boot entries are corrupt
from a previous failed installation.

**Fix:**
1. Power off WS01
2. Go to Settings → System → Motherboard
3. Untick **Secure Boot** if it is ticked
4. If this VM was used before and has a partial Windows install —
   delete the VM entirely, recreate it fresh, and re-attach the ISO

---

### VirtualBox Popup — "The virtual machine failed to boot"

VirtualBox shows a helpful dialogue with a **DVD dropdown**.

**Fix:**
1. Click the DVD dropdown in the popup
2. Select your Windows 11 ISO from the list
   (if it is not listed, click the folder icon to browse to it)
3. Click **"Mount and Retry Boot"**

---

### Windows Setup Loads but Only Shows 2 Editions

**Cause:** A different ISO is mounted — possibly the Windows Server ISO.

**Fix:**
1. Power off WS01
2. Go to Settings → Storage
3. Eject the current ISO (click the blue disc icon → Remove Disc)
4. Re-attach the correct `Win11_25H2_EnglishInternational_x64_v2.iso`
5. Start WS01 again

---

## 📸 Take a Snapshot

WS01 is built, configured, and ready to boot into the Windows 11 installer.
Take a snapshot of this clean state before the OS installation begins.

In VirtualBox with **WS01 powered off** go to:
```
Machine → Take Snapshot
```

Name it:
```
WS01 - Clean VM - Pre OS Install
```

Click **OK**.

> If the Windows 11 installation fails partway through —
> TPM errors, network issues, or Microsoft account prompts you cannot bypass —
> rolling back to this snapshot gives you a completely clean VM
> to start the installation over from scratch in seconds.

---

## 🛠️ Troubleshooting

### Issue: WS01 was accidentally created with the ISO selected during creation

The Unattended Installer may have started.

**Fix:**
1. Power off WS01 immediately
2. Right-click WS01 → **Remove → Delete all files**
3. Recreate WS01 following this guide — leave ISO blank during creation
4. Attach the ISO manually in Storage settings

---

### Issue: Cannot set TPM to 2.0 — option is greyed out

EFI may not be enabled, or the chipset setting is wrong.

**Fix:**
1. Ensure **Enable EFI** is ticked on the Motherboard tab first
2. Set Chipset to **PIIX3**
3. The TPM dropdown should then become active and allow 2.0

---

### Issue: Network adapter name field is not editable

The VM may still be running — VirtualBox locks settings on running VMs.

**Fix:**
1. Power off WS01 completely (not saved state — fully powered off)
2. Go to Settings → Network → Adapter 1
3. The name field is now editable — type `LabNet`
4. Click OK

---

### Issue: Hard disk shows as 0 bytes or incorrectly sized

The virtual hard disk was not created correctly.

**Fix:**
1. Power off WS01
2. Go to Settings → Storage
3. If the hard disk shows wrong size — remove it (right-click → Remove)
4. Create a new one: click the hard disk icon → Create → 50 GB → Dynamic
5. Attach it to the SATA controller

---

### Issue: VirtualBox says "Cannot register the hard disk — already exists"

A virtual hard disk with that name already exists from a previous VM deletion.

**Fix:**
```
File → Virtual Media Manager
```

Find the orphaned VHD in the list, right-click → **Release** then → **Remove**.
Now create WS01 again fresh.

---

## ✅ Verification Checklist

```
VM Creation
[ ] WS01 appears in VirtualBox Manager VM list
[ ] ISO was left blank during creation wizard
[ ] Hard disk created at 50 GB dynamically allocated

System Settings
[ ] RAM: 4096 MB
[ ] CPUs: 2
[ ] EFI: Enabled
[ ] TPM: 2.0
[ ] Secure Boot: DISABLED
[ ] Video Memory: 128 MB
[ ] PAE/NX: Enabled

Storage Settings
[ ] Windows 11 ISO attached — filename showing in Storage
[ ] Not showing "Empty" in optical drive

Network Settings
[ ] Adapter 1: Internal Network
[ ] Adapter 1 Name: LabNet (matches DC01 Adapter 2 exactly)
[ ] Adapter 2: Disabled

Boot Readiness
[ ] WS01 starts and reaches Windows 11 setup screen
[ ] No black screen errors
[ ] No BdsDxe errors
[ ] No Secure Boot errors

Snapshot
[ ] Snapshot taken named "WS01 - Clean VM - Pre OS Install"

Ready to proceed
[ ] WS01 VM is built and configured
[ ] Windows 11 setup screen is visible
[ ] Ready to install Windows 11 in the next step
```

---

## ➡️ Next Step

WS01 is built and the Windows 11 installer is on screen.
Time to install Windows 11 Pro and configure it for domain joining.

**[12 - Install Windows 11 →](../12-Install-Windows-11/README.md)**

---

## ⬅️ Previous Step

**[← 10 - Configure Lab Networking](../10-Configure-Networking/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
