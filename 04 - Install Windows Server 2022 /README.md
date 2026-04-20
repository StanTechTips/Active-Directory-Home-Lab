# 04 - Install Windows Server 2022

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Beginner-blue?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/VM-DC01-purple?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/OS-Windows_Server_2022-0078D6?style=for-the-badge&logo=windows)

> This step walks through booting DC01 for the first time and installing
> Windows Server 2022 with the Desktop Experience (full GUI).
> Every screen in the installer is covered — nothing is skipped.

---

## 📑 Table of Contents

- [Understanding the Edition Choice](#-understanding-the-edition-choice)
- [Boot DC01 from the ISO](#-boot-dc01-from-the-iso)
- [Windows Server 2022 Installation Walkthrough](#-windows-server-2022-installation-walkthrough)
- [First Boot and Administrator Password](#-first-boot-and-administrator-password)
- [Install VirtualBox Guest Additions](#-install-virtualbox-guest-additions)
- [Post Installation Overview](#-post-installation-overview)
- [Take a Snapshot](#-take-a-snapshot)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)

---

## 📖 Understanding the Edition Choice

During installation you will be presented with four editions to choose from.
Picking the wrong one here means starting the installation over — so understand the difference now.

| Edition | GUI | Use Case | Choose This? |
|---------|-----|----------|-------------|
| Windows Server 2022 Standard Evaluation | ❌ No GUI | CLI only — command prompt and PowerShell | ❌ No |
| **Windows Server 2022 Standard Evaluation (Desktop Experience)** | **✅ Full GUI** | **Full desktop — Server Manager, ADUC, GPMC** | **✅ Yes** |
| Windows Server 2022 Datacenter Evaluation | ❌ No GUI | CLI only — unlimited VMs in production | ❌ No |
| Windows Server 2022 Datacenter Evaluation (Desktop Experience) | ✅ Full GUI | Full desktop — unlimited VMs in production | ❌ Not needed |

### Always choose Standard Evaluation (Desktop Experience)

**Desktop Experience** gives you the full Windows graphical interface including:
- Server Manager dashboard
- Active Directory Users and Computers (ADUC)
- Group Policy Management Console (GPMC)
- DNS Manager
- All the tools you need to manage this lab visually

> 💡 **Standard vs Datacenter:**
> The only real difference is that Datacenter licences allow unlimited Windows Server VMs.
> For a homelab with one server, Standard is identical in every feature that matters.

---

## 🚀 Boot DC01 from the ISO

### Step 1 — Start DC01

In VirtualBox Manager double-click **DC01** or select it and click **Start**.

The VM window opens and begins booting.

### Step 2 — Watch for the Boot Prompt

With UEFI enabled you may briefly see:

```
Press any key to boot from CD or DVD......
```

**Click inside the VM window immediately and press any key.**

You have approximately **2–3 seconds** before it times out. If you miss it:
1. Go to Machine → Reset (or right-click → Close → Power Off)
2. Start DC01 again
3. Click inside the window faster this time

> 💡 If DC01 consistently skips the ISO and boots to a black screen,
> go back to **Settings → Storage** and confirm the ISO is still attached.
> UEFI with an empty hard disk will also boot straight to the ISO without needing a keypress.

---

## 🛠️ Windows Server 2022 Installation Walkthrough

### Screen 1 — Language and Regional Settings

| Field | Recommended Value |
|-------|------------------|
| Language to install | English (United States) |
| Time and currency format | Your local region |
| Keyboard or input method | Your keyboard layout |

Click **Next**.

---

### Screen 2 — Install Now

Click the **"Install now"** button in the centre of the screen.

The installer begins loading files — this takes 30–60 seconds.

---

### Screen 3 — Select the Operating System Edition ⬅ Critical Choice

You will see four options as described in the table above.

Click on:

```
✅ Windows Server 2022 Standard Evaluation (Desktop Experience)
```

Confirm the description at the bottom reads:
> *"This option installs the full Windows graphical environment, consuming extra drive space. It can be useful if you want to use the Windows desktop or have an app that requires it."*

Click **Next**.

---

### Screen 4 — Licence Terms

Read the Microsoft Software Licence Terms (or scroll to the bottom).

Tick **"I accept the Microsoft Software Licence Terms"**.

Click **Next**.

---

### Screen 5 — Installation Type ⬅ Important Choice

You will see two options:

| Option | When to Use |
|--------|------------|
| Upgrade: Install Windows and keep files, settings, and applications | Only for upgrading an existing Windows installation |
| **Custom: Install Windows only (advanced)** | **Fresh installation — always use this for a new VM** |

Click **"Custom: Install Windows only (advanced)"**.

---

### Screen 6 — Where to Install Windows

You will see one unallocated disk listed:

```
Drive 0 Unallocated Space     60.0 GB
```

Click on it to select it and click **Next**.

> 💡 You do not need to manually create partitions.
> The Windows installer will automatically create the required system partitions
> (EFI, MSR, and the main Windows partition) when you click Next.

---

### Screen 7 — Installing Windows

The installation progress screen appears with five stages:

```
✅ Copying Windows files
✅ Getting files ready for installation
✅ Installing features
✅ Installing updates
✅ Finishing up
```

**This process takes 10–25 minutes** depending on your SSD speed.

The VM will **automatically restart** at least once during this process — this is completely normal. Do not power off the VM or interact with it during this stage.

> ⚠️ If you see "Press any key to boot from CD or DVD" during the restart —
> **do NOT press any key.** Let it time out and boot from the hard disk.
> The installer has already copied what it needs from the ISO.

---

## 🔐 First Boot and Administrator Password

After the final restart, Windows Server 2022 boots for the first time and immediately asks you to set the Administrator password.

### Password Requirements

The password must meet Windows complexity requirements:
- At least 8 characters (we recommend 12+)
- Must contain uppercase letters
- Must contain lowercase letters
- Must contain numbers or special characters
- Cannot contain the username

### Recommended Lab Password

```
Lab@Admin2024!
```

> 🔒 **Write this password down somewhere safe.**
> This is your domain Administrator password — you will type it dozens of times
> throughout this lab. Losing it means rebuilding the VM from scratch.

Type your password, press **Tab**, type it again to confirm, then press **Enter** or click the arrow.

Windows will finish configuring and take you to the **lock screen**.

Press **Ctrl + Alt + Del** inside the VM — in VirtualBox you can do this with:
```
Input → Keyboard → Insert Ctrl+Alt+Del
```

Or press **Host Key + Del** (Host Key is Right Ctrl by default).

Log in with:
```
Username:  Administrator
Password:  Lab@Admin2024!
```

**Server Manager opens automatically** — you are now inside Windows Server 2022 for the first time.

---

## 🔧 Install VirtualBox Guest Additions

Guest Additions is a small software package installed **inside the VM** that dramatically improves the experience. Install this before doing anything else.

### What Guest Additions Gives You

| Feature | Without Guest Additions | With Guest Additions |
|---------|------------------------|---------------------|
| Screen resolution | Fixed low resolution | Resizes with the window |
| Mouse | Requires click to capture / Host Key to release | Seamless — moves freely in and out |
| Clipboard | Cannot copy/paste between host and VM | Full bidirectional copy/paste |
| Drag and Drop | Not available | Drag files between host and VM |
| Time sync | VM clock drifts | Stays in sync with host |

### Installation Steps

**Step 1 — Insert the Guest Additions disc**

In the VirtualBox menu at the top of the VM window click:
```
Devices → Insert Guest Additions CD Image...
```

**Step 2 — Open the disc inside the VM**

Inside the VM open **File Explorer** → click **This PC** in the left panel.

You will see a CD Drive labelled **"VirtualBox Guest Additions"**.

Double-click it to open it.

**Step 3 — Run the installer**

Inside the disc double-click:
```
VBoxWindowsAdditions-amd64.exe
```

Work through the installer:

| Screen | Action |
|--------|--------|
| Welcome | Click **Next** |
| Installation Location | Leave default — click **Next** |
| Components | Leave all ticked — click **Install** |
| Device driver popups | Click **Install** on any Windows security popups |
| Finish | Select **"Reboot now"** → click **Finish** |

**Step 4 — Enable Shared Clipboard after reboot**

After DC01 restarts and you log back in, go to the VirtualBox menu:
```
Devices → Shared Clipboard → Bidirectional
Devices → Drag and Drop → Bidirectional
```

You can now copy text from this guide and paste it directly into the VM with **Ctrl+V**.

---

## 📋 Post Installation Overview

After Guest Additions is installed, Server Manager will open automatically.
Here is what you are looking at:

```
┌─────────────────────────────────────────────────────────────┐
│  Server Manager Dashboard                                    │
│                                                             │
│  Left Panel          Main Panel                             │
│  ┌─────────────┐     ┌───────────────────────────────────┐  │
│  │ Dashboard   │     │  WELCOME TO SERVER MANAGER        │  │
│  │ Local Server│     │                                   │  │
│  │ All Servers │     │  1. Configure this local server   │  │
│  │ File and... │     │  2. Add roles and features        │  │
│  └─────────────┘     │  3. Add other servers to manage   │  │
│                      └───────────────────────────────────┘  │
│                                                             │
│  ROLES AND SERVER GROUPS                                     │
│  Roles: 0  ←  No roles installed yet — this is correct      │
└─────────────────────────────────────────────────────────────┘
```

**Key things to confirm on the dashboard:**

Click **"Local Server"** in the left panel and verify:

| Property | Expected Value | Status |
|----------|---------------|--------|
| Computer name | Random (e.g. WIN-XXXXX) | Will rename in next step |
| Windows Firewall | Domain: Off or Public: On | Fine for now |
| Remote Desktop | Disabled | Fine for now |
| IE Enhanced Security | On | Fine for now |
| OS Version | Windows Server 2022 Standard Evaluation | ✅ |

> 💡 The random computer name is completely normal at this stage.
> We rename it to **DC01** in the next section (05 - Configure DC01).

---

## 📸 Take a Snapshot

Windows Server 2022 is now installed and running. This is a great save point.

In VirtualBox go to:
```
Machine → Take Snapshot
```

Name it:
```
DC01 - Windows Server 2022 Installed - Pre Configuration
```

Click **OK**.

> If something goes wrong in the next steps (renaming, IP configuration, AD installation)
> you can roll back to this snapshot instantly and have a clean installed server again.

---

## 🛠️ Troubleshooting

### Issue: Black screen after DC01 starts — nothing loads

The VM is not booting from the ISO.

**Fix:**
1. Power off DC01
2. Go to **Settings → Storage** — confirm the ISO is attached and not showing "Empty"
3. Go to **Settings → System → Motherboard** — confirm Optical is first in boot order
4. Start DC01 — when you see any text on screen, click inside the VM and press a key

---

### Issue: Licence terms error — "Windows cannot find the Microsoft Software Licence Terms"

The Unattended Installer was triggered during VM creation.

**Fix:**
1. Power off DC01 → right-click → Remove → **Delete all files**
2. Recreate the VM following [03 - Create the DC01 VM](../03%20-%20Create%20the%20DC01%20VM/README.md) exactly
3. Leave the ISO blank during creation — attach it manually afterwards

---

### Issue: Only two edition options showing instead of four

A different or corrupted ISO may be mounted.

**Fix:**
1. Power off DC01
2. Go to **Settings → Storage** — eject the current ISO
3. Re-attach your `SERVER_EVAL_x64FRE_en-us.iso` file
4. Start DC01 again — you should see all four editions

---

### Issue: Installation is stuck on "Getting files ready" for more than 30 minutes

This is usually a storage performance issue.

**Fix:**
1. Check your host PC's disk usage in Task Manager — if it is at 100% the VM is overwhelmed
2. Close any unnecessary applications on your host PC
3. If this persists, the virtual hard disk may be on a slow HDD — consider moving the VM to an SSD

---

### Issue: Password rejected — "The password does not meet the password policy requirements"

Windows Server enforces complexity requirements even for the initial setup.

**Fix:** Use a password that contains:
- At least 8 characters
- At least one uppercase letter (A–Z)
- At least one lowercase letter (a–z)
- At least one number (0–9) or special character (!@#$%)

Example: `Lab@Admin2024!`

---

### Issue: Guest Additions installer not found in the CD drive

The disc image was not inserted correctly.

**Fix:**
1. Inside the VM go to **Devices → Insert Guest Additions CD Image...**
2. If it says "Unable to insert" — go to **Settings → Storage**, remove any existing ISO from the optical drive, and try again
3. If the CD drive does not appear in File Explorer, open **Disk Management** and check if the virtual CD is visible there

---

## ✅ Verification Checklist

```
Installation Complete
[ ] Windows Server 2022 Standard Evaluation (Desktop Experience) installed
[ ] VM restarted successfully after installation
[ ] Administrator password set and written down safely
[ ] Logged in successfully as Administrator
[ ] Server Manager opened automatically on the desktop

Guest Additions
[ ] Guest Additions CD inserted via Devices menu
[ ] VBoxWindowsAdditions-amd64.exe installed successfully
[ ] VM restarted after Guest Additions install
[ ] Shared Clipboard set to Bidirectional
[ ] Drag and Drop set to Bidirectional
[ ] Copy and paste working between host PC and VM

Server Manager Confirmed
[ ] Dashboard shows Roles: 0 (no roles installed yet — correct)
[ ] Local Server shows correct OS version
[ ] No critical errors showing in red anywhere

Snapshot
[ ] Snapshot taken named "DC01 - Windows Server 2022 Installed - Pre Configuration"

Ready to proceed
[ ] DC01 is running with Windows Server 2022
[ ] Server Manager is open and showing the dashboard
```

---

## ➡️ Next Step

Windows Server 2022 is installed and running. Now we configure the server
before installing Active Directory — renaming it to DC01 and setting a static IP.

**[05 - Configure DC01 →](../005%20-%20Congigure%20DC01/README.md)**

---

## ⬅️ Previous Step

**[← 03 - Create the DC01 VM](../03%20-%20Create%20the%20DC01%20VM/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
