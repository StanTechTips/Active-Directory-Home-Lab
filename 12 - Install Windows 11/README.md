# 12 - Install Windows 11

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Beginner-blue?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/VM-WS01-blue?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/OS-Windows_11_Pro-0078D6?style=for-the-badge&logo=windows)

> Installing Windows 11 on a domain workstation requires a few extra steps
> compared to a standard home install.
> The most important is bypassing the Microsoft account requirement —
> we need a local admin account, not a Microsoft account,
> because this machine will be managed by Active Directory.
> Every screen in the installer is documented here.

---

## 📑 Table of Contents

- [Why Windows 11 Pro — Not Home](#-why-windows-11-pro--not-home)
- [Windows 11 Installation Walkthrough](#-windows-11-installation-walkthrough)
- [Bypass the Microsoft Account Requirement](#-bypass-the-microsoft-account-requirement)
- [Create the Local Admin Account](#-create-the-local-admin-account)
- [Post Installation Configuration](#-post-installation-configuration)
- [Install VirtualBox Guest Additions](#-install-virtualbox-guest-additions)
- [Set a Static IP Address](#-set-a-static-ip-address)
- [Take a Snapshot](#-take-a-snapshot)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)

---

## 💡 Why Windows 11 Pro — Not Home

During the edition selection screen you will see several options.
Choosing the wrong one will prevent domain joining entirely.

| Edition | Domain Join | Group Policy | Use in This Lab |
|---------|-------------|-------------|-----------------|
| Windows 11 Home | ❌ Cannot join domain | ❌ No GPO support | ❌ Never use |
| **Windows 11 Pro** | **✅ Full domain join** | **✅ Full GPO support** | **✅ Always use** |
| Windows 11 Pro Education | ✅ Full domain join | ✅ Full GPO support | ✅ Also works |
| Windows 11 Pro for Workstations | ✅ Full domain join | ✅ Full GPO support | ✅ Also works |
| Windows 11 Education | ✅ Full domain join | ✅ Full GPO support | ✅ Also works |

> ❌ **Windows 11 Home cannot join an Active Directory domain.**
> This is a hard Microsoft limitation — not a configuration issue.
> Always select **Windows 11 Pro** for any domain workstation.

---

## 🛠️ Windows 11 Installation Walkthrough

### Screen 1 — Language and Regional Settings

| Field | Value |
|-------|-------|
| Language to install | English (United Kingdom) or your locale |
| Time and currency format | Your local region |
| Keyboard layout | Your keyboard type |

Click **Next → Install now**.

---

### Screen 2 — Activate Windows

You will see a product key entry screen.

Click **"I don't have a product key"** at the bottom.

> This is legitimate — the Windows 11 evaluation ISO does not require
> a product key and will run fully functional for the lab.

---

### Screen 3 — Select the Operating System Edition ⬅ Critical

You will see the full edition list:

```
○  Windows 11 Home
○  Windows 11 Home N
○  Windows 11 Home Single Language
✅  Windows 11 Pro                    ← Select this one
○  Windows 11 Pro N
○  Windows 11 Pro Education
○  Windows 11 Pro for Workstations
○  Windows 11 Education
```

**Click "Windows 11 Pro"** and click **Next**.

---

### Screen 4 — Licence Terms

Tick **"I accept the Microsoft Software Licence Terms"** and click **Next**.

---

### Screen 5 — Installation Type

| Option | Choose? |
|--------|---------|
| Upgrade: Install Windows and keep files, settings, and applications | ❌ No |
| **Custom: Install Windows only (advanced)** | **✅ Yes** |

Click **"Custom: Install Windows only (advanced)"**.

---

### Screen 6 — Where to Install Windows

You will see one unallocated drive:

```
Drive 0 Unallocated Space     50.0 GB
```

Click on it to select it and click **Next**.

The installer creates the required partitions automatically.

---

### Screen 7 — Installing Windows 11

The progress screen runs through five stages:

```
✅  Copying Windows files
✅  Getting files ready for installation
✅  Installing features
✅  Installing updates
✅  Finishing up
```

**This takes 10–20 minutes.** The VM will restart automatically at least once.

> ⚠️ If during the restart you see "Press any key to boot from CD or DVD" —
> **do NOT press any key.** Let it time out and boot from the hard disk.
> Windows has already copied everything it needs from the ISO.

---

## 🔓 Bypass the Microsoft Account Requirement

After the installation restart, Windows 11 enters the **Out-of-Box Experience (OOBE)** setup.
Microsoft forces you to connect to the internet and sign in with a Microsoft account.

**We do not want this.** We need a local account because this machine
will be joined to and managed by Active Directory.

### The Setup Flow You Will See

```
1.  Country / Region             ← Select your region and click Yes
2.  Keyboard layout              ← Confirm your layout and click Yes
3.  Second keyboard layout       ← Click "Skip"
4.  "Let's connect you to a      ← STOP HERE — bypass required
     network" screen
```

### Bypass Method — OOBE\BYPASSNRO

When you reach the **"Let's connect you to a network"** screen:

**1.** Press **Shift + F10** simultaneously.

A **Command Prompt** window opens on top of the setup screen.

**2.** Type the following exactly and press **Enter**:

```
OOBE\BYPASSNRO
```

**3.** The VM will **automatically restart** — this is expected and normal.

**4.** Setup runs through the first three screens again (country, keyboard, skip).

**5.** On the network screen this time you will see a new option at the bottom:

```
[I don't have internet]
```

Click **"I don't have internet"**.

**6.** The next screen says:

```
"There's more to discover when you connect to the internet"
```

Click **"Continue with limited setup"** at the bottom.

You now reach the local account creation screen.

> 💡 **Why does this work?**
> `OOBE\BYPASSNRO` (Network Requirements Override) is a Microsoft-provided
> command that disables the internet connectivity requirement during setup.
> It is built into Windows 11 for exactly this scenario — enterprise deployments
> that set up machines before connecting them to a domain.

---

## 👤 Create the Local Admin Account

You are now on the **"Who's going to use this PC?"** screen.

### Username

Type:
```
LocalAdmin
```

Click **Next**.

---

### Password

Type a password:
```
Local@Admin1
```

Click **Next** — confirm the password — click **Next** again.

---

### Security Questions

You will be asked three security questions. Choose any questions and
enter answers you will remember. These are for the local account only.

Click **Next** after each one.

---

### Privacy Settings

Windows 11 shows a series of privacy toggle screens:

| Setting | Recommended for Lab |
|---------|-------------------|
| Location | Off |
| Find my device | Off |
| Diagnostic data | Required only |
| Inking and typing | Off |
| Tailored experiences | Off |
| Advertising ID | Off |

Click **Accept** or **Next** through each screen.

Windows 11 will spend a few minutes on **"Just a moment..."** and
**"Getting things ready"** — this is normal first-boot configuration.

**You will land on the Windows 11 desktop.**

---

## ⚙️ Post Installation Configuration

Once you reach the desktop, complete these three quick tasks
before installing Guest Additions or doing anything else.

### Task 1 — Rename the Computer to WS01

Right-click the **Start button** → **System** → scroll down to
**"Rename this PC"** → click it → type:

```
WS01
```

Click **Next** → **Restart now**.

After the restart log back in as `LocalAdmin`.

> 💡 Alternatively via PowerShell:
> ```powershell
> Rename-Computer -NewName "WS01" -Restart -Force
> ```

---

### Task 2 — Confirm Windows 11 Pro Edition

After the restart, right-click **Start** → **System**.

Under **Windows specifications** confirm:

```
Edition:   Windows 11 Pro     ✅
```

If it shows **Home** the wrong edition was selected — you will need
to reinstall Windows 11 from the beginning and select Pro.

---

### Task 3 — Check Windows Activation Status

```powershell
# Open PowerShell and check activation
slmgr /xpr
```

A popup will appear. For the evaluation ISO it will say:

```
The machine is permanently activated.
```

or

```
Windows is in Notification mode.
```

Both are fine for the lab — the machine will work fully for the evaluation period.

---

## 🔧 Install VirtualBox Guest Additions

Install Guest Additions on WS01 just as we did on DC01.
This enables clipboard sharing, seamless mouse, and better display resolution.

### Step 1 — Insert the Guest Additions Disc

In the VirtualBox menu at the top of the WS01 window click:

```
Devices → Insert Guest Additions CD Image...
```

---

### Step 2 — Run the Installer

Inside WS01 open **File Explorer** → **This PC**.

Double-click the **CD Drive (VirtualBox Guest Additions)**.

Double-click **VBoxWindowsAdditions-amd64.exe**.

Work through the installer:

| Screen | Action |
|--------|--------|
| Welcome | Click **Next** |
| Installation folder | Leave default → Click **Next** |
| Components | Leave all ticked → Click **Install** |
| Driver security popups | Click **Install** on any that appear |
| Finish | Select **"Reboot now"** → Click **Finish** |

---

### Step 3 — Enable Shared Clipboard

After WS01 restarts, in the VirtualBox menu click:

```
Devices → Shared Clipboard → Bidirectional
Devices → Drag and Drop → Bidirectional
```

You can now copy and paste freely between your host PC and WS01.

---

## 🌐 Set a Static IP Address

WS01 needs a static IP on the Internal Network adapter so DC01
can always find it at a predictable address.

### Method 1 — Via Windows Settings (GUI)

**1.** Right-click **Start** → **Settings** → **Network & Internet** → **Ethernet**

**2.** Click on your Ethernet adapter → click **Edit** next to "IP assignment"

**3.** Change dropdown from **Automatic (DHCP)** to **Manual**

**4.** Toggle **IPv4** to **On** and enter:

| Field | Value |
|-------|-------|
| IP address | `192.168.100.20` |
| Subnet mask | `255.255.255.0` |
| Gateway | `192.168.100.10` |
| Preferred DNS | `192.168.100.10` |
| Alternate DNS | Leave blank |

**5.** Click **Save**.

---

### Method 2 — Via PowerShell (Faster)

Open **PowerShell as Administrator** and run:

```powershell
# Set the static IP address
New-NetIPAddress `
    -InterfaceAlias "Ethernet" `
    -IPAddress "192.168.100.20" `
    -PrefixLength 24 `
    -DefaultGateway "192.168.100.10"

# Point DNS to DC01
Set-DnsClientServerAddress `
    -InterfaceAlias "Ethernet" `
    -ServerAddresses "192.168.100.10"
```

---

### Verify the IP Settings

```powershell
Get-NetIPConfiguration
```

Expected output:

```
InterfaceAlias       : Ethernet
IPv4Address          : 192.168.100.20     ✅
IPv4DefaultGateway   : 192.168.100.10     ✅
DNSServer            : 192.168.100.10     ✅
```

---

### Test Connectivity to DC01

```powershell
# Ping DC01
ping 192.168.100.10
```

Expected output — 4 successful replies:

```
Reply from 192.168.100.10: bytes=32 time=2ms TTL=128   ✅
Reply from 192.168.100.10: bytes=32 time=1ms TTL=128   ✅
Reply from 192.168.100.10: bytes=32 time=2ms TTL=128   ✅
Reply from 192.168.100.10: bytes=32 time=1ms TTL=128   ✅

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

If you get 4 replies — WS01 and DC01 are communicating. ✅

If you get "Request timed out" — go to the Troubleshooting section below.

---

### Test DNS Resolution

```powershell
# Verify WS01 can resolve the domain via DC01's DNS
nslookup Lab.local 192.168.100.10
```

Expected output:

```
Server:   DC01.Lab.local
Address:  192.168.100.10

Name:     Lab.local
Addresses: 192.168.100.10   ✅
```

Both the ping and the nslookup must succeed before proceeding to domain joining.

---

## 📸 Take a Snapshot

Windows 11 Pro is installed, renamed to WS01, Guest Additions are installed,
and the static IP is configured. This is a perfect save point.

In VirtualBox go to:
```
Machine → Take Snapshot
```

Name it:
```
WS01 - Windows 11 Pro Installed - Static IP Set - Pre Domain Join
```

Click **OK**.

> This snapshot is invaluable. If domain joining fails for any reason —
> wrong credentials, DNS issues, or network misconfiguration —
> you can roll back to this snapshot and have a clean,
> correctly configured Windows 11 machine ready to try again.

---

## 🛠️ Troubleshooting

### Issue: Shift + F10 did not open a Command Prompt

The keyboard shortcut may not have worked inside the VM window.

**Fix:**
1. Click inside the VM window to ensure it has focus
2. Try **Input → Keyboard → Insert Ctrl+Alt+Del** to confirm the VM is responding
3. Try pressing **Shift + F10** again
4. If still not working — in the VirtualBox menu click
   **Input → Keyboard → Soft Keyboard** to use a virtual on-screen keyboard

---

### Issue: OOBE\BYPASSNRO command typed but nothing happened

The command may have been typed incorrectly.

**Fix:**
The command is case-insensitive but must be typed exactly:
```
OOBE\BYPASSNRO
```
Common mistakes:
- Using a forward slash `/` instead of backslash `\`
- Adding spaces between characters
- Typing `OOBEBYPASSNRO` without the backslash

---

### Issue: After OOBE\BYPASSNRO restart, still no "I don't have internet" option

The bypass did not apply correctly.

**Fix:**
```
1.  When you reach the network screen again
2.  Press Shift + F10 again
3.  Type:  OOBE\BYPASSNRO
4.  Press Enter — VM restarts again
5.  This time the option should appear
```

If after two attempts the option still does not appear:
- Power off WS01 completely
- Roll back to the `WS01 - Clean VM - Pre OS Install` snapshot
- Start the Windows 11 installation fresh

---

### Issue: Ping to 192.168.100.10 fails — "Request timed out"

WS01 cannot reach DC01 on the internal network.

**Fix — Check in this order:**

```powershell
# Step 1 — Confirm WS01's IP is set correctly
Get-NetIPAddress -AddressFamily IPv4

# Step 2 — Confirm DNS is pointing to DC01
Get-DnsClientServerAddress -AddressFamily IPv4
```

If IP or DNS is wrong — reapply using the PowerShell method above.

If IP and DNS are correct:
1. Power off both VMs
2. Check VirtualBox settings:
   - DC01 → Settings → Network → Adapter 2 → Name: `LabNet`
   - WS01 → Settings → Network → Adapter 1 → Name: `LabNet`
3. Names must be identical — restart both VMs and test again

---

### Issue: Windows 11 installed as Home edition instead of Pro

The wrong edition was selected during installation.

**Fix:**
1. Roll back to the `WS01 - Clean VM - Pre OS Install` snapshot
2. Start WS01 again
3. On the edition selection screen carefully select **Windows 11 Pro** — not Home
4. Continue the installation

---

### Issue: "Your PC doesn't meet Windows 11 system requirements"

TPM or Secure Boot configuration is not correct.

**Fix:**
1. Power off WS01
2. Go to **Settings → System → Motherboard**:
   - TPM Version: `2.0` ✅
   - Enable EFI: ✅
   - Secure Boot: ❌ (must be unticked)
3. Restart WS01 and try again

---

### Issue: Guest Additions installer not found in the CD drive

The disc image was not inserted correctly.

**Fix:**
1. Go to **Devices → Insert Guest Additions CD Image...**
2. If it says unable to insert — go to **Settings → Storage**,
   remove the Windows 11 ISO from the optical drive, and try again
3. The CD drive should then appear in File Explorer as
   **VirtualBox Guest Additions**

---

## ✅ Verification Checklist

```
Windows 11 Installation
[ ] Windows 11 Pro selected — not Home or any other edition
[ ] No product key entered — clicked "I don't have a product key"
[ ] Custom install selected — not Upgrade
[ ] Installation completed and VM restarted successfully

Microsoft Account Bypass
[ ] OOBE\BYPASSNRO command run successfully via Shift+F10
[ ] VM restarted after command
[ ] "I don't have internet" option appeared and was clicked
[ ] "Continue with limited setup" clicked

Local Account Created
[ ] Username: LocalAdmin
[ ] Password set and remembered
[ ] Security questions answered
[ ] Reached the Windows 11 desktop successfully

Post Installation
[ ] Computer renamed to WS01
[ ] VM restarted after rename
[ ] Windows 11 Pro confirmed in System settings
[ ] Guest Additions installed and VM restarted
[ ] Shared Clipboard set to Bidirectional

Network Configuration
[ ] Static IP set: 192.168.100.20
[ ] Subnet mask: 255.255.255.0
[ ] Default gateway: 192.168.100.10
[ ] Preferred DNS: 192.168.100.10
[ ] Ping 192.168.100.10 returns 4 successful replies
[ ] nslookup Lab.local resolves to 192.168.100.10

Snapshot
[ ] Snapshot taken named "WS01 - Windows 11 Pro Installed - Static IP Set - Pre Domain Join"

Ready to proceed
[ ] WS01 is running Windows 11 Pro
[ ] Named WS01
[ ] Static IP 192.168.100.20 confirmed
[ ] DNS pointing to DC01 at 192.168.100.10
[ ] Connectivity to DC01 verified
```

---

## ➡️ Next Step

Windows 11 is installed and WS01 can reach DC01 over the network.
Time to join WS01 to the Lab.local domain.

**[13 - Join WS01 to the Domain →](../13-Join-Domain/README.md)**

---

## ⬅️ Previous Step

**[← 11 - Create the WS01 VM](../11-Create-WS01-VM/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
