# 17 - GPO 4 — Desktop Wallpaper

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Intermediate-orange?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/VM-DC01-purple?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Tool-Group_Policy-red?style=for-the-badge&logo=windows)
![Static Badge](https://img.shields.io/badge/Lab-Complete-gold?style=for-the-badge)

> The Desktop Wallpaper GPO is the perfect final step for this lab.
> It is visual proof that your entire Group Policy infrastructure works —
> when a domain user logs into WS01 and sees the custom wallpaper,
> it confirms that DC01 is authenticating the user, delivering policies
> via SYSVOL, and managing the workstation end-to-end.
> This GPO is also used in real enterprise environments to display
> legal notices, branding, and security warnings on all managed machines.

---

## 📑 Table of Contents

- [Why a Wallpaper GPO?](#-why-a-wallpaper-gpo)
- [How This GPO Differs from GPO 3](#-how-this-gpo-differs-from-gpo-3)
- [Step 1 — Create the Wallpaper Folder and Share](#-step-1--create-the-wallpaper-folder-and-share)
- [Step 2 — Create the Wallpaper Image](#-step-2--create-the-wallpaper-image)
- [Step 3 — Create the Desktop Wallpaper GPO](#-step-3--create-the-desktop-wallpaper-gpo)
- [Step 4 — Configure the GPO Setting](#-step-4--configure-the-gpo-setting)
- [Step 5 — Link the GPO to _USERS OU](#-step-5--link-the-gpo-to-_users-ou)
- [Step 6 — Apply and Verify the Policy](#-step-6--apply-and-verify-the-policy)
- [Step 7 — Test on WS01](#-step-7--test-on-ws01)
- [Complete Lab Verification](#-complete-lab-verification)
- [Take Final Snapshots](#-take-final-snapshots)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)
- [Lab Complete — What You Have Built](#-lab-complete--what-you-have-built)

---

## 💡 Why a Wallpaper GPO?

Beyond being a satisfying visual confirmation that Group Policy works,
the Desktop Wallpaper GPO has real-world applications:

```
┌─────────────────────────────────────────────────────────────┐
│          Real World Uses of Wallpaper GPO                    │
│                                                             │
│   ⚖️  Legal Notice Display                                  │
│       "This system is for authorised users only.            │
│        All activity is monitored and logged."               │
│       Required by many compliance frameworks                │
│                                                             │
│   🏢 Corporate Branding                                     │
│       Company logo and colours on every managed PC          │
│       Consistent look across thousands of machines          │
│                                                             │
│   🔐 Security Awareness                                     │
│       "Do not leave your PC unattended while logged in"     │
│       Constant reminder of security responsibilities        │
│                                                             │
│   🖥️  Environment Identification                            │
│       "PRODUCTION ENVIRONMENT — Changes affect live systems" │
│       Prevents accidental changes in wrong environment      │
│                                                             │
│   ✅ GPO Verification                                        │
│       Visual proof that Group Policy is delivering          │
│       policies to domain workstations correctly             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 How This GPO Differs from GPO 3

This final GPO introduces the last key concept in Group Policy — the
difference between **Computer Configuration** and **User Configuration**:

```
┌─────────────────────────────────────────────────────────────┐
│                  GPO Configuration Areas                     │
│                                                             │
│   GPO 3 — Disable USB Storage                               │
│   ┌─────────────────────────────────────────────┐           │
│   │  Computer Configuration  ← Used here        │           │
│   │  Policies                                   │           │
│   │  Administrative Templates                   │           │
│   │  System → Removable Storage Access          │           │
│   └─────────────────────────────────────────────┘           │
│   Linked to: _COMPUTERS OU                                  │
│   Applies when: Computer starts up                          │
│   Applies to: The machine itself — regardless of user       │
│                                                             │
│   GPO 4 — Desktop Wallpaper                                 │
│   ┌─────────────────────────────────────────────┐           │
│   │  User Configuration      ← Used here        │           │
│   │  Policies                                   │           │
│   │  Administrative Templates                   │           │
│   │  Desktop → Desktop                          │           │
│   └─────────────────────────────────────────────┘           │
│   Linked to: _USERS OU                                      │
│   Applies when: User logs in                                │
│   Applies to: The user account — on any domain machine      │
└─────────────────────────────────────────────────────────────┘
```

> 💡 **User Configuration GPOs follow the user.**
> If Ronald Gomez logs into WS01 — he gets the wallpaper.
> If Ronald Gomez logs into a different domain computer — he still gets the wallpaper.
> The policy travels with the user account, not the machine.

---

## 📁 Step 1 — Create the Wallpaper Folder and Share

The wallpaper image must be stored in a **network share** on DC01
so that WS01 can access it over the LabNet internal network.
A local path on WS01 will not work — the GPO uses a UNC path
(`\\DC01\Wallpaper\wallpaper.jpg`) that every domain computer can reach.

On **DC01** open **PowerShell as Administrator** and run:

```powershell
# Create the wallpaper folder
New-Item -ItemType Directory -Path "C:\Wallpaper"

# Share the folder on the network
New-SmbShare `
    -Name "Wallpaper" `
    -Path "C:\Wallpaper" `
    -FullAccess "Everyone"

# Verify the share was created
Get-SmbShare -Name "Wallpaper"
```

Expected output:

```
Name      ScopeName  Path          Description
----      ---------  ----          -----------
Wallpaper *          C:\Wallpaper              ✅
```

### Verify the Share is Accessible

```powershell
# Confirm the UNC path is reachable from DC01 itself
Test-Path "\\DC01\Wallpaper"
```

Expected output: `True` ✅

---

## 🎨 Step 2 — Create the Wallpaper Image

We create the wallpaper programmatically using PowerShell and the
.NET `System.Drawing` library — no external software or image editor needed.

On **DC01** run this script in PowerShell:

```powershell
Add-Type -AssemblyName System.Drawing

# Canvas dimensions — standard 1080p
$Width    = 1920
$Height   = 1080
$Bitmap   = New-Object System.Drawing.Bitmap($Width, $Height)
$Graphics = [System.Drawing.Graphics]::FromImage($Bitmap)

# Set rendering quality
$Graphics.SmoothingMode     = [System.Drawing.Drawing2D.SmoothingMode]::AntiAlias
$Graphics.TextRenderingHint = [System.Drawing.Text.TextRenderingHint]::ClearTypeGridFit

# Background — dark navy blue
$BgColour = [System.Drawing.Color]::FromArgb(0, 35, 102)
$Graphics.Clear($BgColour)

# Draw a subtle accent bar across the top
$AccentBrush = New-Object System.Drawing.SolidBrush(
    [System.Drawing.Color]::FromArgb(0, 120, 215))
$Graphics.FillRectangle($AccentBrush, 0, 0, $Width, 8)

# Draw a subtle accent bar across the bottom
$Graphics.FillRectangle($AccentBrush, 0, ($Height - 8), $Width, 8)

# Main title text — large and bold
$TitleFont  = New-Object System.Drawing.Font("Arial", 56, [System.Drawing.FontStyle]::Bold)
$WhiteBrush = New-Object System.Drawing.SolidBrush([System.Drawing.Color]::White)
$TitleText  = "Lab.local — Active Directory Domain"
$TitleSize  = $Graphics.MeasureString($TitleText, $TitleFont)
$TitleX     = ($Width - $TitleSize.Width) / 2
$Graphics.DrawString($TitleText, $TitleFont, $WhiteBrush, $TitleX, 380)

# Subtitle text — medium
$SubFont    = New-Object System.Drawing.Font("Arial", 30, [System.Drawing.FontStyle]::Regular)
$GreyBrush  = New-Object System.Drawing.SolidBrush(
    [System.Drawing.Color]::FromArgb(180, 200, 230))
$SubText    = "Managed by Group Policy  ·  DC01 — Windows Server 2022"
$SubSize    = $Graphics.MeasureString($SubText, $SubFont)
$SubX       = ($Width - $SubSize.Width) / 2
$Graphics.DrawString($SubText, $SubFont, $GreyBrush, $SubX, 470)

# Security notice text — smaller
$NoticeFont = New-Object System.Drawing.Font("Arial", 20, [System.Drawing.FontStyle]::Italic)
$NoticeText = "This system is for authorised users only.  All activity is monitored and logged."
$NoticeSize = $Graphics.MeasureString($NoticeText, $NoticeFont)
$NoticeX    = ($Width - $NoticeSize.Width) / 2
$Graphics.DrawString($NoticeText, $NoticeFont, $GreyBrush, $NoticeX, 570)

# Save the wallpaper as JPEG
$SavePath = "C:\Wallpaper\wallpaper.jpg"
$Bitmap.Save($SavePath, [System.Drawing.Imaging.ImageFormat]::Jpeg)

# Clean up
$Graphics.Dispose()
$Bitmap.Dispose()

Write-Host "Wallpaper saved to: $SavePath" -ForegroundColor Green

# Verify the file was created
Get-Item $SavePath | Select-Object Name, Length, LastWriteTime
```

Expected output:

```
Wallpaper saved to: C:\Wallpaper\wallpaper.jpg

Name           Length    LastWriteTime
----           ------    -------------
wallpaper.jpg  245760    20/04/2026 09:30:00   ✅
```

---

## 🆕 Step 3 — Create the Desktop Wallpaper GPO

### Open Group Policy Management

On **DC01**:
```
Server Manager → Tools → Group Policy Management
```

### Create the New GPO

Right-click **"Lab.local"** in the left panel →
click **"Create a GPO in this domain and Link it here..."**

Type the name:
```
Desktop Wallpaper
```

Click **OK**.

The **"Desktop Wallpaper"** GPO now appears in the left panel.

---

## ⚙️ Step 4 — Configure the GPO Setting

### Open the GPO for Editing

Right-click **"Desktop Wallpaper"** → click **"Edit"**.

### Navigate to the Wallpaper Setting

In the left panel navigate to:

```
Desktop Wallpaper [DC01.Lab.local]
└── User Configuration
    └── Policies
        └── Administrative Templates
            └── Desktop
                └── Desktop    ← Click the second "Desktop" folder
```

> ⚠️ **There are two "Desktop" folders.**
> The first is the parent folder. Click the **second "Desktop"**
> which is the sub-folder inside the first one.
> The wallpaper setting lives in the second one.

Click on the second **"Desktop"** folder.

A list of settings appears on the right side.

### Enable the Wallpaper Setting

Find and double-click:

```
Desktop Wallpaper
```

The settings window opens.

Select **"Enabled"**.

In the **"Wallpaper Name"** field below, type the full UNC path:

```
\\DC01\Wallpaper\wallpaper.jpg
```

In the **"Wallpaper Style"** dropdown select:

```
Fill
```

> 💡 **Wallpaper Style options explained:**
>
> | Style | Behaviour |
> |-------|-----------|
> | Fill | Stretches to fill the screen — no borders ✅ Recommended |
> | Fit | Fits within the screen — may show black bars |
> | Stretch | Stretches to fill — may distort aspect ratio |
> | Tile | Repeats the image across the screen |
> | Center | Centers the image at original size |

Click **Apply** → Click **OK**.

### Confirm the Setting

Back in the right panel:

```
Setting               State    Comment
-------               -----    -------
Desktop Wallpaper     Enabled  \\DC01\Wallpaper\wallpaper.jpg    ✅
```

### Close the Editor

Close the **Group Policy Management Editor**.

---

## 🔗 Step 5 — Link the GPO to _USERS OU

Just like GPO 3, this GPO was initially linked to the domain root.
We need to move the link to the **_USERS OU** so it targets users only.

### Remove the Domain-Level Link

In Group Policy Management click on **"Lab.local"**.

In the right panel under **"Linked Group Policy Objects"** tab,
right-click **"Desktop Wallpaper"** → click **"Delete Link"**.

Click **OK** to confirm.

> ⚠️ **Delete Link — not Delete GPO.**
> This only removes the link at domain level.
> The GPO itself remains in Group Policy Objects.

### Link to _USERS OU

In the left panel right-click **"_USERS"** →
click **"Link an Existing GPO..."**

Select **"Desktop Wallpaper"** from the list.

Click **OK**.

The GPO tree should now look like this:

```
Lab.local
├── Default Domain Policy          → Domain root
├── _COMPUTERS
│   └── 📋 Disable USB Storage     → Computers only
└── _USERS
    └── 📋 Desktop Wallpaper       → Users only ✅
        ├── Finance
        ├── HR
        ├── IT
        └── Management
```

> 💡 **Inheritance in action:**
> The wallpaper GPO is linked to `_USERS` and automatically
> **inherits down** to all four department sub-OUs.
> IT users, HR users, Finance users, and Management users
> all receive the wallpaper without linking the GPO to each sub-OU individually.

---

## ✅ Step 6 — Apply and Verify the Policy

### Force Policy Update on WS01

On **WS01** open **PowerShell as Administrator** and run:

```powershell
gpupdate /force
```

### Verify on WS01 via gpresult

```powershell
gpresult /r
```

Look in the **USER SETTINGS** section for:

```
USER SETTINGS
-------------
    Applied Group Policy Objects
    ----------------------------
        Desktop Wallpaper          ← ✅ Confirmed applied
        Default Domain Policy
```

### Verify GPO Linkage on DC01

```powershell
# Confirm GPO is linked to _USERS OU on DC01
Get-GPInheritance -Target "OU=_USERS,DC=Lab,DC=local" |
    Select-Object -ExpandProperty InheritedGpoLinks |
    Select-Object DisplayName, Enabled
```

Expected output:

```
DisplayName        Enabled
-----------        -------
Desktop Wallpaper  True     ✅
Default Domain Policy True  ✅
```

---

## 🖥️ Step 7 — Test on WS01

### Log Off and Log Back In as a Domain User

User Configuration GPOs apply at **login time** — not just on gpupdate.
Log off completely and log back in to see the wallpaper:

**1.** On WS01 click **Start** → click the user icon → **Sign out**.

**2.** At the login screen click **"Other user"**.

**3.** Log in with a domain user:

```
Username:  LAB\rgomez
Password:  (the new password set on first login)
```

**4.** As Windows 11 loads the user profile and desktop,
the custom wallpaper will appear:

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│                                                            │
│         Lab.local — Active Directory Domain                │
│                                                            │
│     Managed by Group Policy  ·  DC01 — Windows Server 2022│
│                                                            │
│   This system is for authorised users only.                │
│   All activity is monitored and logged.                    │
│                                                            │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

✅ **The wallpaper confirms your entire Active Directory infrastructure is working.**

---

### Test with Multiple Domain Users

Log out and log back in with a different user from a different department:

```
Username:  LAB\bjackson    (Management user)
Username:  LAB\pcampbell   (Finance user)
```

All domain users in `_USERS` and its sub-OUs should receive the same wallpaper. ✅

---

## 🏁 Complete Lab Verification

Now that all four GPOs are configured, run a complete verification of
the entire lab from DC01:

### Verify All GPOs Exist

```powershell
Get-GPO -All | Select-Object DisplayName, GpoStatus, CreationTime |
    Sort-Object CreationTime |
    Format-Table -AutoSize
```

Expected output:

```
DisplayName           GpoStatus         CreationTime
-----------           ---------         ------------
Default Domain Policy AllSettingsEnabled 20/04/2026...
Default Domain Con... AllSettingsEnabled 20/04/2026...
Disable USB Storage   AllSettingsEnabled 20/04/2026...
Desktop Wallpaper     AllSettingsEnabled 20/04/2026...  ✅
```

---

### Verify All GPO Links

```powershell
# Check domain root links
Write-Host "=== Domain Root ===" -ForegroundColor Cyan
Get-GPInheritance -Target "DC=Lab,DC=local" |
    Select-Object -ExpandProperty InheritedGpoLinks |
    Select-Object DisplayName, Enabled

# Check _COMPUTERS OU links
Write-Host "`n=== _COMPUTERS OU ===" -ForegroundColor Cyan
Get-GPInheritance -Target "OU=_COMPUTERS,DC=Lab,DC=local" |
    Select-Object -ExpandProperty InheritedGpoLinks |
    Select-Object DisplayName, Enabled

# Check _USERS OU links
Write-Host "`n=== _USERS OU ===" -ForegroundColor Cyan
Get-GPInheritance -Target "OU=_USERS,DC=Lab,DC=local" |
    Select-Object -ExpandProperty InheritedGpoLinks |
    Select-Object DisplayName, Enabled
```

Expected output:

```
=== Domain Root ===
DisplayName             Enabled
Default Domain Policy   True    ✅

=== _COMPUTERS OU ===
DisplayName             Enabled
Disable USB Storage     True    ✅
Default Domain Policy   True    ✅

=== _USERS OU ===
DisplayName             Enabled
Desktop Wallpaper       True    ✅
Default Domain Policy   True    ✅
```

---

### Verify the Complete Domain at a Glance

```powershell
# Domain summary
Write-Host "=== Domain Summary ===" -ForegroundColor Yellow
Get-ADDomain | Select-Object DNSRoot, NetBIOSName, DomainMode

# DC summary
Write-Host "`n=== Domain Controller ===" -ForegroundColor Yellow
Get-ADDomainController | Select-Object Name, IPv4Address, OperatingSystem, IsGlobalCatalog

# User count
Write-Host "`n=== User Count ===" -ForegroundColor Yellow
$UserCount = (Get-ADUser -Filter * -SearchBase "OU=_USERS,DC=Lab,DC=local" -SearchScope Subtree).Count
Write-Host "Total domain users in _USERS: $UserCount"

# Computer count
Write-Host "`n=== Computer Count ===" -ForegroundColor Yellow
Get-ADComputer -Filter * | Select-Object Name, DNSHostName, Enabled

# GPO count
Write-Host "`n=== GPO Summary ===" -ForegroundColor Yellow
$GPOCount = (Get-GPO -All).Count
Write-Host "Total GPOs in domain: $GPOCount"
Get-GPO -All | Select-Object DisplayName, GpoStatus
```

---

## 📸 Take Final Snapshots

You have completed the entire Active Directory Home Lab.
Take final snapshots on both VMs to preserve this milestone permanently.

**On DC01:**

```
Machine → Take Snapshot
Name: DC01 - LAB COMPLETE - All GPOs Configured
```

**On WS01:**

```
Machine → Take Snapshot
Name: WS01 - LAB COMPLETE - Domain Joined - All GPOs Applied
```

> 🏆 **These are your trophy snapshots.**
> They represent a fully working, enterprise-grade Active Directory
> environment built entirely from scratch. Keep them.

---

## 🛠️ Troubleshooting

### Issue: Wallpaper does not appear after login — desktop shows default Windows 11 background

The GPO may not have applied yet or the UNC path is wrong.

**Fix — Step 1: Verify the UNC path is accessible from WS01:**

```powershell
# Run on WS01
Test-Path "\\DC01\Wallpaper\wallpaper.jpg"
```

Must return `True`. If it returns `False`:
- Confirm the Wallpaper share exists on DC01 (`Get-SmbShare -Name "Wallpaper"`)
- Confirm DC01 is running and reachable (`ping 192.168.100.10`)

---

**Fix — Step 2: Confirm the GPO is in gpresult output:**

```powershell
gpresult /r
```

Look for **"Desktop Wallpaper"** under **USER SETTINGS → Applied GPOs**.

If not listed — check the GPO is linked to `_USERS` and the user
account is in a sub-OU of `_USERS`.

---

### Issue: "Desktop Wallpaper" GPO showing twice in gpresult

The GPO is linked at both the domain root AND the _USERS OU.

**Fix:**

In Group Policy Management click **Lab.local** → **Linked GPO Objects** tab.

If **Desktop Wallpaper** appears here — right-click → **Delete Link**.

The GPO should only be linked to `_USERS`.

---

### Issue: Wallpaper applies but user can change it back

The GPO enforces the wallpaper but does not prevent users from changing it.
To also prevent changes, enable an additional setting in the same GPO:

In the **Desktop Wallpaper** GPO editor navigate to:

```
User Configuration → Policies → Administrative Templates
→ Control Panel → Personalization
```

Enable:
```
Prevent changing desktop background
```

---

### Issue: Wallpaper shows as a solid colour (not the custom image)

The wallpaper image file cannot be read from the UNC path.

**Fix — check share permissions on DC01:**

```powershell
# Check share permissions
Get-SmbShareAccess -Name "Wallpaper"
```

Ensure **Everyone** has at least **Read** access.

If not:
```powershell
Grant-SmbShareAccess -Name "Wallpaper" -AccountName "Everyone" -AccessRight Read -Force
```

---

### Issue: Some users get the wallpaper but others do not

Users in sub-OUs should inherit the policy from `_USERS`.
Check if inheritance is blocked on a sub-OU.

**Fix:**

```powershell
# Check each sub-OU for inheritance blocking
$SubOUs = @("IT","HR","Finance","Management")
foreach ($OU in $SubOUs) {
    $Inheritance = Get-GPInheritance -Target "OU=$OU,OU=_USERS,DC=Lab,DC=local"
    Write-Host "$OU — Inheritance blocked: $($Inheritance.GpoInheritanceBlocked)"
}
```

If any show `True` — right-click that OU in GPMC and untick **"Block Inheritance"**.

---

## ✅ Verification Checklist

```
Share and Image
[ ] C:\Wallpaper folder created on DC01
[ ] Wallpaper SMB share created — accessible at \\DC01\Wallpaper
[ ] wallpaper.jpg file created in C:\Wallpaper
[ ] Test-Path \\DC01\Wallpaper\wallpaper.jpg returns True from WS01

GPO Creation
[ ] "Desktop Wallpaper" GPO created under Lab.local

GPO Configuration
[ ] Navigated to User Configuration → Administrative Templates → Desktop → Desktop
[ ] Desktop Wallpaper setting set to Enabled
[ ] Wallpaper Name: \\DC01\Wallpaper\wallpaper.jpg
[ ] Wallpaper Style: Fill
[ ] Editor closed — settings saved

GPO Linking
[ ] Domain-level link removed from Lab.local
[ ] GPO linked to _USERS OU
[ ] Desktop Wallpaper visible under _USERS in GPMC left panel

Policy Applied
[ ] gpupdate /force ran on WS01
[ ] gpresult /r shows "Desktop Wallpaper" under USER SETTINGS Applied GPOs
[ ] Get-GPInheritance confirms GPO linked and enabled on _USERS OU

Visual Confirmation
[ ] Logged off WS01 completely
[ ] Logged back in as a domain user (LAB\rgomez or similar)
[ ] Custom dark blue Lab.local wallpaper is visible on the desktop
[ ] Tested with at least two different domain users
[ ] All users from different department OUs see the wallpaper

Complete Lab Verification
[ ] Get-GPO -All shows all 4 GPOs exist
[ ] GPO links verified for domain root, _COMPUTERS, and _USERS
[ ] Complete domain summary script ran successfully

Final Snapshots
[ ] DC01 snapshot: "DC01 - LAB COMPLETE - All GPOs Configured"
[ ] WS01 snapshot: "WS01 - LAB COMPLETE - Domain Joined - All GPOs Applied"
```

---

## 🏆 Lab Complete — What You Have Built

Congratulations — you have built a fully functional,
enterprise-grade Active Directory home lab from scratch.

```
┌─────────────────────────────────────────────────────────────┐
│            Active Directory Home Lab — Complete              │
│                                                             │
│   Infrastructure                                            │
│   ✅ Oracle VirtualBox — virtualisation platform            │
│   ✅ DC01 — Windows Server 2022 Domain Controller           │
│   ✅ WS01 — Windows 11 Pro domain workstation               │
│   ✅ LabNet — isolated internal network 192.168.100.0/24    │
│                                                             │
│   Active Directory                                          │
│   ✅ Lab.local forest and domain                            │
│   ✅ DNS integrated with Active Directory                   │
│   ✅ Kerberos authentication                                │
│   ✅ SYSVOL and NETLOGON shares                             │
│   ✅ 7 Organisational Units (3 top-level, 4 departments)    │
│   ✅ 110 domain user accounts across 4 departments          │
│   ✅ WS01 joined to domain and verified                     │
│                                                             │
│   Group Policy                                              │
│   ✅ GPO 1 — Password Policy (complexity, length, history)  │
│   ✅ GPO 2 — Account Lockout (5 attempts, 30 min lock)      │
│   ✅ GPO 3 — USB Storage blocked on all domain computers    │
│   ✅ GPO 4 — Custom wallpaper pushed to all domain users    │
│                                                             │
│   Skills Demonstrated                                       │
│   ✅ Windows Server administration                          │
│   ✅ Active Directory design and deployment                 │
│   ✅ DNS configuration and management                       │
│   ✅ PowerShell scripting and automation                    │
│   ✅ Group Policy creation and targeting                    │
│   ✅ Virtual networking and VM management                   │
│   ✅ Security policy implementation                         │
│   ✅ Domain user and computer management                    │
└─────────────────────────────────────────────────────────────┘
```

### What to Explore Next

Now that the core lab is complete, here are ideas to extend it further:

| Project | Skill Developed |
|---------|----------------|
| Add a second workstation VM (WS02) | Managing multiple domain members |
| Create security groups and nest them | Role-based access control (RBAC) |
| Set up a File Server with NTFS permissions | File server administration |
| Configure Fine-Grained Password Policies | Advanced AD security |
| Install and configure WSUS | Windows Update management |
| Enable Remote Desktop via GPO | Remote administration |
| Configure Audit Policy for logon events | Security monitoring and SIEM prep |
| Set up a DHCP server on DC01 | Network services administration |
| Create login scripts via GPO | Automation and user environment management |
| Simulate a domain user password audit | Security compliance |

---

## ⬅️ Previous Step

**[← 16 - GPO 3 Disable USB Storage](../16-GPO3-Disable-USB/README.md)**

---

## 🏠 Back to the Beginning

**[← Active Directory Home Lab — Main README](../README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*

---

> *Built with dedication, patience, and a lot of PowerShell.*
> *Every error encountered, every fix applied, every snapshot taken*
> *made this lab and this guide better.*
> *The skills you have developed here are directly transferable*
> *to real-world IT roles — you have built something to be proud of.*
