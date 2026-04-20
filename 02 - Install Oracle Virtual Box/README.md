# 02 - Install Oracle VirtualBox

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Beginner-blue?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Tool-VirtualBox-183A61?style=for-the-badge&logo=virtualbox)
![Static Badge](https://img.shields.io/badge/Platform-Windows-0078D6?style=for-the-badge&logo=windows)

> Oracle VirtualBox is a free, open-source Type 2 hypervisor that runs on top of your existing OS.
> It is the foundation of the entire lab — every VM lives inside it.

---

## 📑 Table of Contents

- [What is VirtualBox?](#-what-is-virtualbox)
- [Download VirtualBox](#-download-virtualbox)
- [Install VirtualBox](#-install-virtualbox)
- [Install the Extension Pack](#-install-the-extension-pack)
- [Configure Global Settings](#-configure-global-settings)
- [Understand the VirtualBox Interface](#-understand-the-virtualbox-interface)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)

---

## 🔍 What is VirtualBox?

VirtualBox is a **Type 2 hypervisor** — software that runs on top of your existing operating system and lets you create and run multiple virtual machines simultaneously.

```
┌─────────────────────────────────────────┐
│         Your Physical PC                │
│                                         │
│   ┌─────────────────────────────────┐   │
│   │     Host OS (Windows 11)        │   │
│   │                                 │   │
│   │   ┌───────────┐  ┌───────────┐  │   │
│   │   │   DC01    │  │   WS01   │  │   │
│   │   │  (VM 1)   │  │  (VM 2)  │  │   │
│   │   └───────────┘  └───────────┘  │   │
│   │         Oracle VirtualBox       │   │
│   └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

**Key facts about VirtualBox:**
- Completely free and open source — no licence cost
- Runs on Windows, macOS, and Linux hosts
- Supports Windows, Linux, and many other guest operating systems
- Snapshots let you save and restore VM states instantly
- Virtual networking lets VMs communicate with each other and the internet

---

## 📥 Download VirtualBox

### Step 1 — Download the Installer

Go to the official Oracle VirtualBox download page:

```
https://www.virtualbox.org/wiki/Downloads
```

Under **"VirtualBox platform packages"** click the link that matches your host OS:

| Host OS | Package to Download |
|---------|-------------------|
| Windows 10 / 11 | Windows hosts |
| macOS Intel | macOS / Intel hosts |
| macOS Apple Silicon | macOS / Intel hosts (with Rosetta) |
| Ubuntu / Debian Linux | Linux distributions |

> ⚠️ **Always download from the official Oracle website.**
> Never use a third-party mirror or download site — tampered builds exist.

---

### Step 2 — Download the Extension Pack

On the **same page**, scroll down to find:

```
VirtualBox X.X.X Oracle VM VirtualBox Extension Pack
All supported platforms
```

Click **"All supported platforms"** to download the `.vbox-extpack` file.

> ⚠️ **Critical:** The Extension Pack version number must **exactly match** your VirtualBox version.
> Download both files from the same page visit to guarantee they match.

**What you should have downloaded:**

```
📁 Downloads\
├── VirtualBox-7.x.x-Win.exe          ← Main installer
└── Oracle_VirtualBox_Extension_Pack-7.x.x.vbox-extpack   ← Extension Pack
```

---

## 🔧 Install VirtualBox

### Step 1 — Run the Installer

Double-click `VirtualBox-7.x.x-Win.exe` to launch the setup wizard.

Work through each screen:

| Screen | Action |
|--------|--------|
| Welcome | Click **Next** |
| Custom Setup | Leave all defaults — click **Next** |
| Warning: Network Interfaces | Click **Yes** ← read the note below |
| Ready to Install | Click **Install** |
| UAC Prompt | Click **Yes** to allow |
| Installation Progress | Wait for completion |
| Finish | Leave "Start VirtualBox" ticked — click **Finish** |

> ⚠️ **"Network Interfaces" warning explained:**
> VirtualBox installs a virtual network adapter on your host PC.
> During installation your internet connection will briefly disconnect for 5–10 seconds.
> This is completely normal — click **Yes** to proceed.

---

### Step 2 — Verify VirtualBox Launched

VirtualBox Manager should open automatically. You will see the welcome screen with options to create a new VM.

```
✅ VirtualBox Manager is open
✅ Welcome screen is showing
✅ No error messages
```

If you see any error about **Kernel driver not installed** go to the Troubleshooting section below.

---

## 🔌 Install the Extension Pack

The Extension Pack adds essential features including USB device support that you will need for this lab.

### Step 1 — Open Extension Manager

In VirtualBox Manager go to:

```
File → Tools → Extension Manager
```

Or on older versions:

```
File → Preferences → Extensions
```

### Step 2 — Add the Extension Pack

Click the **green plus icon** (Add package) on the right side.

Browse to your Downloads folder and select:

```
Oracle_VirtualBox_Extension_Pack-7.x.x.vbox-extpack
```

Click **Open**.

### Step 3 — Accept the Licence

A dialogue will appear showing the Oracle VirtualBox Extension Pack Personal Use and Evaluation Licence. Scroll to the bottom and click **I Agree**.

### Step 4 — Verify Installation

The Extension Pack should now appear in the list with a green tick and the correct version number matching your VirtualBox installation.

```
Oracle VM VirtualBox Extension Pack    7.x.x    ✅ Installed
```

> 💡 **What the Extension Pack enables in this lab:**
> - USB 2.0 / 3.0 passthrough to VMs (needed for GPO 3 - Disable USB testing)
> - Seamless clipboard between host and VM
> - Better display and graphics support

---

## ⚙️ Configure Global Settings

Before creating any VMs, configure a few global settings to make the lab run smoothly.

Go to **File → Preferences** (or **VirtualBox → Preferences** on macOS):

### General Tab — Set Default VM Location

Change the **Default Machine Folder** to a location on your SSD with plenty of space:

```
C:\LabFiles\VMs\
```

This keeps all your VM files organised in one place rather than buried in your user profile.

### Input Tab — Host Key

The **Host Key** is the key you press to release your mouse from inside a VM. The default is **Right Ctrl**.

Remember this key — you will use it constantly when switching between VMs and your host PC.

### Update Tab — Disable Auto Updates (Optional)

If you want to prevent VirtualBox from updating mid-lab, untick **"Check for Updates"**. This is optional but avoids a version mismatch between VirtualBox and the Extension Pack during the project.

Click **OK** to save all settings.

---

## 🖥️ Understand the VirtualBox Interface

Before building your first VM, familiarise yourself with the layout:

```
┌─────────────────────────────────────────────────────────────┐
│  VirtualBox Manager                                          │
│                                                             │
│  ┌──────────────┐  ┌────────────────────────────────────┐   │
│  │  VM List     │  │  VM Details Panel                  │   │
│  │  (left panel)│  │                                    │   │
│  │              │  │  Name:        DC01                 │   │
│  │  DC01        │  │  OS:          Windows 2022         │   │
│  │  WS01        │  │  RAM:         4096 MB              │   │
│  │              │  │  Storage:     60.00 GB             │   │
│  │              │  │  Network:     NAT / Internal       │   │
│  └──────────────┘  └────────────────────────────────────┘   │
│                                                             │
│  [New] [Settings] [Start] [Discard] [Show]                  │
└─────────────────────────────────────────────────────────────┘
```

**Key buttons explained:**

| Button | What it does |
|--------|-------------|
| **New** | Creates a new virtual machine |
| **Settings** | Opens settings for the selected VM — hardware, network, storage |
| **Start** | Powers on the selected VM |
| **Discard** | Discards saved state (like a hard reset) |
| **Snapshots** | Manage save points for the VM |

**VM status colours:**

| Colour | Meaning |
|--------|---------|
| 🟢 Green | VM is running |
| 🟡 Yellow | VM is saved / suspended |
| ⚪ Grey | VM is powered off |

---

## 🛠️ Troubleshooting

### Issue: "Kernel driver not installed" error on Windows

This means the VirtualBox kernel module failed to load, usually because Windows blocked the driver.

**Fix:**
1. Open **Windows Security → Device Security → Core Isolation**
2. If **Memory Integrity** is ON — turn it OFF and restart
3. Reinstall VirtualBox
4. If still failing, go to **Settings → Windows Update → Advanced Options → Recovery → Advanced Startup** and disable Secure Boot temporarily

---

### Issue: "VT-x is not available" when starting a VM

VirtualBox can see that virtualisation is not enabled at the CPU level.

**Fix:**
1. Shut down your PC completely (not restart — fully shut down)
2. Boot into BIOS and enable Intel VT-x or AMD-V
3. See [01 - Pre-Requirements](../01%20-%20Pre-Requirements/README.md) for detailed BIOS steps

---

### Issue: Extension Pack installation fails with version mismatch

The Extension Pack version does not match the installed VirtualBox version.

**Fix:**
1. Check your VirtualBox version: **Help → About VirtualBox**
2. Go to https://www.virtualbox.org/wiki/Downloads
3. Download the Extension Pack that matches your exact version number
4. Try installing again

---

### Issue: VirtualBox installed but shows no welcome screen

VirtualBox may have opened minimised or behind another window.

**Fix:**
1. Check the taskbar for the VirtualBox window
2. If not visible, search for **"Oracle VirtualBox"** in the Start menu and reopen it

---

## ✅ Verification Checklist

Before moving to the next step confirm the following:

```
Installation
[ ] VirtualBox installed successfully with no errors
[ ] VirtualBox Manager opens and shows the welcome screen
[ ] No "Kernel driver" or "VT-x not available" errors

Extension Pack
[ ] Extension Pack installed and showing correct version
[ ] Version number matches VirtualBox exactly

Global Settings
[ ] Default VM folder changed to your SSD location
[ ] Host Key confirmed (Right Ctrl by default)

Ready to proceed
[ ] VirtualBox Manager is open
[ ] No error messages anywhere
```

---

## ➡️ Next Step

VirtualBox is installed and configured. Time to build your first virtual machine.

**[03 - Create the Domain Controller VM (DC01) →](../03-Create-DC01-VM/README.md)**

---

## ⬅️ Previous Step

**[← 01 - Pre-Requirements](../01-Pre-Requirements/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
