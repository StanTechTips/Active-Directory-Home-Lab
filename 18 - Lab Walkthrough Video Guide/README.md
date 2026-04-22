# 18 - Lab Walkthrough — Full Video Demo

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Type-Video_Walkthrough-red?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Duration-15--20_mins-blue?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Lab-Complete-gold?style=for-the-badge)

> This section is a complete screen recording walkthrough of the
> finished Active Directory Home Lab — demonstrating every component
> live across both virtual machines.
> The video proves the lab works end-to-end and is the most powerful
> addition to your portfolio — showing real, working infrastructure
> rather than just configuration screenshots.

---

## 📑 Table of Contents

- [Before You Hit Record](#-before-you-hit-record)
- [Recommended Recording Software](#-recommended-recording-software)
- [Video Structure and Scene List](#-video-structure-and-scene-list)
- [Scene 1 — Introduction and Lab Overview](#-scene-1--introduction-and-lab-overview)
- [Scene 2 — VirtualBox Manager Overview](#-scene-2--virtualbox-manager-overview)
- [Scene 3 — DC01 Active Directory Overview](#-scene-3--dc01-active-directory-overview)
- [Scene 4 — PowerShell Verification on DC01](#-scene-4--powershell-verification-on-dc01)
- [Scene 5 — Group Policy Management Overview](#-scene-5--group-policy-management-overview)
- [Scene 6 — WS01 Domain Login](#-scene-6--ws01-domain-login)
- [Scene 7 — GPO Verification on WS01](#-scene-7--gpo-verification-on-ws01)
- [Scene 8 — Live GPO Demo — USB Block](#-scene-8--live-gpo-demo--usb-block)
- [Scene 9 — Live GPO Demo — Account Lockout](#-scene-9--live-gpo-demo--account-lockout)
- [Scene 10 — Closing Summary](#-scene-10--closing-summary)
- [Recording Tips](#-recording-tips)
- [Adding the Video to GitHub](#-adding-the-video-to-github)
- [Video Description Template](#-video-description-template)

---

## 🎬 Before You Hit Record

### Prepare Both VMs

Before starting the recording make sure both VMs are in a clean state:

**On DC01:**
```powershell
# Confirm all AD services are running
Get-Service ADWS, NTDS, DNS, KDC, Netlogon |
    Select-Object Name, Status |
    Format-Table -AutoSize

# Confirm domain is healthy
Get-ADDomain | Select-Object DNSRoot, NetBIOSName
```

**On WS01:**
```powershell
# Confirm WS01 is domain joined
(Get-WmiObject -Class Win32_ComputerSystem).Domain

# Confirm DNS points to DC01
Get-DnsClientServerAddress -AddressFamily IPv4 |
    Select-Object InterfaceAlias, ServerAddresses
```

---

### Pre-Video Checklist

```
Both VMs
[ ] DC01 is running and fully booted
[ ] WS01 is running and at the login screen (not logged in)
[ ] Both VMs are visible in the VirtualBox Manager window
[ ] Shared clipboard is working (test with a copy/paste)

DC01 Ready Checks
[ ] Server Manager is open
[ ] Active Directory Users and Computers is ready to open
[ ] Group Policy Management is ready to open
[ ] PowerShell is ready to open as Administrator

WS01 Ready Checks
[ ] WS01 is at the Windows 11 login screen
[ ] At least one domain user password is known and ready
[ ] The LAB\rgomez or similar account is unlocked

Recording Software
[ ] Recording software is installed and tested
[ ] Microphone is working (do a test recording)
[ ] Screen resolution looks clean at 1080p
[ ] Notifications and popups are turned off on host PC
[ ] Phone is on silent
```

---

### Turn Off Distracting Notifications

On your **host PC** (not inside the VM) before recording:

**Windows:**
```
Settings → System → Notifications → Turn off all notifications
```
Or press **Windows + N** → click the bell icon to enable Focus Assist.

**Browser:** Close all browser tabs not related to the demo.

**Email/Slack/Teams:** Minimise or close before recording.

---

## 🎥 Recommended Recording Software

| Software | Cost | Platform | Best For |
|----------|------|----------|----------|
| **OBS Studio** | Free | Windows, Mac, Linux | Full control, professional quality |
| Xbox Game Bar | Free (built-in) | Windows 11 | Quick recordings, no install needed |
| ShareX | Free | Windows | Screenshots + recording |
| Loom | Free tier | Windows, Mac | Easy sharing + viewer analytics |
| Camtasia | Paid | Windows, Mac | Editing + annotations |

### OBS Studio Quick Setup (Recommended)

**1.** Download from `https://obsproject.com`

**2.** On first launch the Auto-Configuration Wizard runs:
- Optimise for recording (not streaming)
- Resolution: 1920x1080
- Frame rate: 30 FPS

**3.** In **Sources** click **+** → **Display Capture** → select your monitor.

**4.** In **Audio** confirm your microphone is listed.

**5.** Click **Start Recording** when ready.

**6.** Recordings save to your Videos folder by default.

> 💡 **Resolution tip:** Set your host PC display to **1920x1080**
> before recording. This ensures the VirtualBox windows display
> cleanly without scaling artifacts.

---

## 📋 Video Structure and Scene List

The video is structured as 10 scenes. Each scene has a purpose,
what to show on screen, and suggested narration talking points.

**Target total length: 15–20 minutes**

| Scene | Topic | Target Length |
|-------|-------|--------------|
| 1 | Introduction and lab overview | 1–2 minutes |
| 2 | VirtualBox Manager overview | 1 minute |
| 3 | DC01 Active Directory overview | 2–3 minutes |
| 4 | PowerShell verification on DC01 | 2–3 minutes |
| 5 | Group Policy Management overview | 2 minutes |
| 6 | WS01 domain login | 1–2 minutes |
| 7 | GPO verification on WS01 | 1–2 minutes |
| 8 | Live GPO demo — USB block | 1–2 minutes |
| 9 | Live GPO demo — account lockout | 1–2 minutes |
| 10 | Closing summary | 1 minute |

---

## 🎬 Scene 1 — Introduction and Lab Overview

**Screen to show:** Your GitHub repository README or a clean desktop

**Duration:** 1–2 minutes

**What to say:**

> "Hi — in this video I'm going to walk through my Active Directory
> Home Lab which I've built from scratch using Oracle VirtualBox,
> Windows Server 2022, and Windows 11.
>
> The lab simulates a real enterprise environment with a Domain Controller,
> 110 domain users, a domain-joined workstation, and four Group Policy
> Objects covering password policy, account lockout, USB restriction,
> and desktop wallpaper.
>
> Everything you're about to see is running live on my machine in
> two virtual machines — DC01 running Windows Server 2022 as the
> Domain Controller, and WS01 running Windows 11 Pro as the workstation.
>
> Let me start with VirtualBox to show you both VMs are up and running."

---

## 🖥️ Scene 2 — VirtualBox Manager Overview

**Screen to show:** VirtualBox Manager window with both VMs visible

**Duration:** 1 minute

**What to show on screen:**

1. Open **Oracle VirtualBox Manager** on your host PC
2. Show both **DC01** and **WS01** in the left panel — both showing green (Running)
3. Click on **DC01** to show its specifications in the right panel:
   - RAM: 4096 MB
   - Network adapters: NAT + Internal Network (LabNet)
4. Click on **WS01** to show its specifications:
   - RAM: 4096 MB
   - Network adapter: Internal Network (LabNet)

**What to say:**

> "Here's VirtualBox Manager — you can see both virtual machines
> are running simultaneously. DC01 on the left is our Windows Server
> 2022 Domain Controller, and WS01 is our Windows 11 workstation.
>
> DC01 has two network adapters — one NAT adapter for internet access
> and one Internal Network adapter called LabNet on the 192.168.100.0
> subnet. WS01 only has the internal adapter since it only needs to
> communicate with DC01 on the domain network."

---

## 🏢 Scene 3 — DC01 Active Directory Overview

**Screen to show:** DC01 VM — Active Directory Users and Computers

**Duration:** 2–3 minutes

**What to show on screen:**

**Step 1 — Switch to DC01 window**

Click on the DC01 VirtualBox window to bring it into focus.

**Step 2 — Open ADUC**

```
Server Manager → Tools → Active Directory Users and Computers
```

**Step 3 — Expand the domain structure and narrate each part:**

```
Lab.local
├── _COMPUTERS    ← click to show WS01 computer object
├── _GROUPS       ← click to show it exists
└── _USERS
    ├── Finance   ← click to show users inside
    ├── HR        ← click to show users inside
    ├── IT        ← click to show users inside
    └── Management ← click to show users inside
```

**Step 4 — Double-click a user** (e.g. Ronald Gomez) to show their properties:
- General tab: Name, description, email
- Account tab: Username, UPN, domain
- Member Of tab: Domain Users group membership

**What to say:**

> "Here's Active Directory Users and Computers on DC01.
> You can see the domain is Lab.local, and I've created a clean
> OU structure with underscore prefixes so they sort to the top.
>
> Inside _COMPUTERS is WS01 — our domain-joined workstation.
>
> Inside _USERS I have four department sub-OUs — Finance, HR,
> IT, and Management. Each one is populated with domain users
> created by my PowerShell bulk provisioning script.
>
> Let me open one of these users to show the properties that
> were set automatically by the script."

---

## 💻 Scene 4 — PowerShell Verification on DC01

**Screen to show:** DC01 — PowerShell as Administrator

**Duration:** 2–3 minutes

**Open PowerShell as Administrator on DC01 and run these commands live:**

### Command 1 — Domain information

```powershell
Get-ADDomain | Select-Object DNSRoot, NetBIOSName, DomainMode, PDCEmulator
```

**Narrate:** *"Get-ADDomain confirms the domain is Lab.local with NetBIOS name LAB,
running at Windows Server 2016 functional level — and DC01 is the PDC Emulator."*

---

### Command 2 — User count by department

```powershell
Get-ADUser -Filter * `
    -SearchBase "OU=_USERS,DC=Lab,DC=local" `
    -SearchScope Subtree `
    -Properties Department |
    Group-Object Department |
    Select-Object Name, Count |
    Sort-Object Count -Descending |
    Format-Table -AutoSize
```

**Narrate:** *"Here's a breakdown of all 110 users by department —
all created automatically by my PowerShell script in under 60 seconds."*

---

### Command 3 — Password policy confirmation

```powershell
Get-ADDefaultDomainPasswordPolicy |
    Select-Object MinPasswordLength, ComplexityEnabled,
                  MaxPasswordAge, PasswordHistoryCount,
                  LockoutThreshold, LockoutDuration |
    Format-List
```

**Narrate:** *"The domain password policy — minimum 10 characters,
complexity required, 90 day maximum age, 5 password history,
and account lockout after 5 failed attempts with a 30 minute lock.
All configured via Group Policy."*

---

### Command 4 — WS01 computer account

```powershell
Get-ADComputer -Identity "WS01" -Properties * |
    Select-Object Name, DNSHostName, IPv4Address,
                  OperatingSystem, DistinguishedName, Enabled
```

**Narrate:** *"And here's WS01's computer account in Active Directory —
you can see it's running Windows 11 Pro, its IP address is
192.168.100.20, and it's correctly placed in the _COMPUTERS OU."*

---

## 📋 Scene 5 — Group Policy Management Overview

**Screen to show:** DC01 — Group Policy Management Console

**Duration:** 2 minutes

**What to show on screen:**

Open **Group Policy Management** (Server Manager → Tools → Group Policy Management).

**Step 1 — Show the GPO tree structure:**

```
Lab.local
├── Default Domain Policy     ← click to show
├── _COMPUTERS
│   └── Disable USB Storage   ← click to show
└── _USERS
    └── Desktop Wallpaper     ← click to show
```

**Step 2 — Click on each GPO and show the Settings tab:**

For **Default Domain Policy:**
- Click the **Settings** tab on the right
- Show the password and lockout settings listed

For **Disable USB Storage:**
- Click the **Settings** tab
- Show "All Removable Storage classes: Deny all access: Enabled"

For **Desktop Wallpaper:**
- Click the **Settings** tab
- Show the UNC path and Fill style setting

**What to say:**

> "Here's the Group Policy Management Console — the central place
> where all domain policies are managed.
>
> I have four GPOs deployed. The Default Domain Policy handles
> password complexity and account lockout for the entire domain.
>
> Disable USB Storage is linked specifically to the _COMPUTERS OU —
> so it only applies to domain workstations like WS01, not to users.
>
> Desktop Wallpaper is linked to the _USERS OU — it follows users
> wherever they log in on any domain machine.
>
> Each GPO has a clear, single purpose — this is Group Policy best practice."

---

## 🔐 Scene 6 — WS01 Domain Login

**Screen to show:** WS01 VM — Login screen then desktop

**Duration:** 1–2 minutes

**What to show on screen:**

**Step 1 — Switch to WS01 window**

Click on the WS01 VirtualBox window.

**Step 2 — Show the login screen**

WS01 should be at the Windows 11 login screen (log out first if needed).
Show the "Other user" option at the bottom left.

**Step 3 — Log in as a domain user**

Click **Other user** and type:
```
Username:  LAB\rgomez
Password:  (domain user password)
```

**Step 4 — As the desktop loads, highlight:**
- The domain wallpaper appearing automatically
- The username shown in the taskbar
- "Sign in to: LAB" shown during login

**What to say:**

> "Switching over to WS01 — our Windows 11 workstation.
>
> You can see it's at the login screen. I'll click Other user
> and log in as Ronald Gomez — one of our 110 domain users
> created by the PowerShell script.
>
> Notice as soon as the desktop loads — the custom wallpaper
> appears automatically. This is the Desktop Wallpaper GPO
> delivered from DC01 via the SYSVOL share.
>
> The wallpaper confirms the entire chain is working —
> DC01 authenticated this user via Kerberos, found their
> account in the _USERS OU, and delivered the Group Policy
> linked to that OU."

---

## ✅ Scene 7 — GPO Verification on WS01

**Screen to show:** WS01 — PowerShell as Administrator

**Duration:** 1–2 minutes

**Open PowerShell as Administrator on WS01 and run:**

### Command 1 — Confirm domain membership

```powershell
(Get-WmiObject -Class Win32_ComputerSystem) |
    Select-Object Name, Domain, PartOfDomain
```

**Narrate:** *"WS01 confirms it is a member of Lab.local domain."*

---

### Command 2 — Show applied GPOs

```powershell
gpresult /r
```

Point out in the output:

```
COMPUTER SETTINGS
-----------------
Applied Group Policy Objects:
    Disable USB Storage          ← Point this out
    Default Domain Policy        ← Point this out

USER SETTINGS
-------------
Applied Group Policy Objects:
    Desktop Wallpaper            ← Point this out
    Default Domain Policy        ← Point this out
```

**Narrate:** *"Gpresult shows every GPO applied to this machine and
this user. You can see Disable USB Storage is applied at the computer
level — and Desktop Wallpaper is applied at the user level for Ronald Gomez."*

---

### Command 3 — Verify secure channel

```powershell
Test-ComputerSecureChannel -Verbose
```

**Narrate:** *"Test-ComputerSecureChannel returns True — confirming
the trust relationship between WS01 and DC01 is healthy."*

---

## 🔌 Scene 8 — Live GPO Demo — USB Block

**Screen to show:** WS01 — attempting to access a USB drive

**Duration:** 1–2 minutes

> 💡 **Setup:** Have a USB drive plugged into your host PC.
> If you do not have a USB drive available, you can demonstrate
> this via the registry confirmation instead (see alternative below).

### With a USB drive available:

**Step 1 — Attach USB to WS01 via VirtualBox:**

In the VirtualBox WS01 menu at the top click:
```
Devices → USB → [Your USB drive name]
```

**Step 2 — Open File Explorer on WS01**

The USB drive appears under **"This PC"** — but when you try to open it:

```
G:\ is not accessible.
This operation has been cancelled due to restrictions in effect
on this computer. Please contact your system administrator.
```

**What to say:**

> "Here's the USB restriction GPO in action. I've attached a USB
> drive through VirtualBox — you can see it appears in File Explorer,
> but the moment I try to open it Windows blocks access completely.
>
> The error message tells the user to contact their system administrator.
> From DC01 I can remove this policy in minutes if a user legitimately
> needs USB access — but by default it's locked down for everyone."

---

### Alternative — Registry confirmation (no USB drive needed):

```powershell
# Show the registry keys the GPO created
Get-ItemProperty `
    -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}"
```

**Narrate:** *"The registry shows Deny_Read and Deny_Write both set to 1 —
confirming the USB restriction GPO has been applied at the kernel level."*

---

## 🔒 Scene 9 — Live GPO Demo — Account Lockout

**Screen to show:** WS01 login screen and DC01 PowerShell

**Duration:** 1–2 minutes

### Demonstrate the lockout live:

**Step 1 — Log out of WS01**

Sign out of the current domain user session.

**Step 2 — At the login screen attempt wrong passwords**

Click **Other user** and enter:
```
Username:  LAB\bjackson
Password:  wrongpassword    ← Do this 5 times deliberately
```

After 5 attempts the message appears:
```
Your account has been locked out.
Please contact your system administrator.
```

**Step 3 — Switch to DC01 and show the lockout**

```powershell
# On DC01 — confirm the account is locked
Get-ADUser -Identity "bjackson" -Properties LockedOut, BadLogonCount |
    Select-Object Name, LockedOut, BadLogonCount
```

Expected output:
```
Name           LockedOut  BadLogonCount
----           ---------  -------------
Betty Jackson  True       5
```

**Step 4 — Unlock it from DC01**

```powershell
Unlock-ADAccount -Identity "bjackson"
Write-Host "bjackson has been unlocked." -ForegroundColor Green
```

**What to say:**

> "This is the account lockout policy — after 5 wrong password attempts
> the account locks automatically.
>
> Switching to DC01, I can see Betty Jackson's account shows
> LockedOut: True with 5 bad logon attempts recorded.
>
> As the domain administrator I can unlock this instantly with
> a single PowerShell command — Unlock-ADAccount — and the user
> can log back in immediately.
>
> This is a fundamental security control that every Active Directory
> environment should have configured."

---

## 🎯 Scene 10 — Closing Summary

**Screen to show:** Split view of both VMs or the GitHub repository

**Duration:** 1 minute

**What to say:**

> "So that's a full walkthrough of the Active Directory Home Lab.
>
> To summarise what we've built and demonstrated:
>
> On the infrastructure side — two virtual machines, an isolated
> internal network, Windows Server 2022 as the Domain Controller,
> and Windows 11 Pro as a domain workstation.
>
> On the Active Directory side — the Lab.local domain, a clean
> OU structure, 110 domain users provisioned via PowerShell
> automation, and WS01 fully joined to the domain.
>
> On the Group Policy side — password complexity enforced,
> account lockout working and demonstrated live, USB storage
> blocked at the kernel level, and desktop wallpaper pushed
> to all domain users — all managed centrally from DC01.
>
> The full step-by-step written guide for every part of this
> lab is documented in the GitHub repository linked below.
>
> Thanks for watching."

---

## 🎙️ Recording Tips

### Audio Tips

- Speak clearly and at a **steady pace** — faster than you think is natural
- Keep sentences **short** — one idea per sentence
- It is okay to pause — silence is easier to edit out than rushed speech
- Do a 30-second test recording and listen back before the full recording
- If you make a mistake — **pause, then repeat** the sentence cleanly
  (easy to cut in editing)

---

### Screen Tips

- **Zoom in** when showing text-heavy screens — especially PowerShell output
- **Move your mouse slowly** — viewers follow the cursor to understand what you're pointing at
- **Hover over items** before clicking them — give viewers time to read
- **Maximise windows** when showing them — avoid clutter in the background
- Switch between VMs using **Alt+Tab** on your host PC — smoother than clicking

---

### Flow Tips

- Do a **full dry run** before recording — go through all 10 scenes without recording
- Keep both VMs on separate **taskbar buttons** — makes switching visible and clean
- Start each scene with what you are about to show — *"Now I'm going to open..."*
- End each scene with a summary line — *"...confirming the policy is working."*
- Do not try to fix mistakes live — just re-record that scene

---

### OBS Scene Switching (Advanced)

If using OBS, set up two scenes:

**Scene 1 — DC01 Focus:**
- Add Display Capture filtered to your DC01 window

**Scene 2 — WS01 Focus:**
- Add Display Capture filtered to your WS01 window

Switching between scenes in OBS with a hotkey looks more professional
than moving the mouse to switch VirtualBox windows.

---

## 📤 Adding the Video to GitHub

GitHub has a **25 MB file size limit** for uploads — most screen recordings
will exceed this. Here are the best options:

### Option 1 — YouTube (Recommended)

**1.** Upload your recording to **YouTube** (can be unlisted if you prefer)

**2.** Copy the video URL

**3.** In this README, replace the placeholder below with your actual URL:

```markdown
## 📹 Video Walkthrough

[![Active Directory Home Lab Walkthrough](https://img.youtube.com/vi/YOUR_VIDEO_ID/maxresdefault.jpg)](https://www.youtube.com/watch?v=YOUR_VIDEO_ID)

▶️ [Watch on YouTube](https://www.youtube.com/watch?v=YOUR_VIDEO_ID)
```

> Replace `YOUR_VIDEO_ID` with the ID from your YouTube URL.
> For example if the URL is `https://www.youtube.com/watch?v=dQw4w9WgXcQ`
> the video ID is `dQw4w9WgXcQ`.

---

### Option 2 — GitHub README Embed (under 25 MB only)

If your recording is under 25 MB (unlikely for a 15-20 min video):

Drag and drop the video file directly into the GitHub README editor.
GitHub will host it and embed it automatically.

---

### Option 3 — Loom

Upload to Loom (`https://www.loom.com`) — free tier allows sharing.
Copy the Loom link and add it to the README.

---

## 📝 Video Description Template

Use this as your YouTube / Loom video description:

```
Active Directory Home Lab — Full Walkthrough

A complete demonstration of a functional enterprise-grade
Active Directory home lab built from scratch using Oracle VirtualBox.

What's demonstrated in this video:
✅ VirtualBox VM overview — DC01 (Windows Server 2022) and WS01 (Windows 11)
✅ Active Directory Users and Computers — OU structure and 110 domain users
✅ PowerShell verification — domain health, user counts, password policy
✅ Group Policy Management — all 4 GPOs configured and linked
✅ Domain login on WS01 — Kerberos authentication and GPO wallpaper delivery
✅ Live USB storage block — Disable USB GPO demonstrated in action
✅ Live account lockout — 5 failed attempts, lock confirmed, unlock from DC01

Technologies used:
• Oracle VirtualBox 7.x
• Windows Server 2022 Standard Evaluation
• Windows 11 Pro
• Active Directory Domain Services (AD DS)
• Group Policy Management (GPMC)
• PowerShell 5.1
• DNS Server

Full step-by-step written guide:
[Link to your GitHub repository]

Lab built as part of my IT portfolio to demonstrate hands-on skills in:
Windows Server administration, Active Directory, PowerShell automation,
Group Policy, and virtual network configuration.
```

---

## 📹 Video Walkthrough

> 🎬 **Add your video link here after recording.**
>
> Replace this section with your YouTube embed using the template above.
>
> Example:
> ```
> [![Lab Walkthrough](thumbnail_url)](your_video_url)
> ▶️ Watch on YouTube
> ```

---

## ⬅️ Previous Step

**[← 17 - GPO 4 Desktop Wallpaper](../17-GPO4-Desktop-Wallpaper/README.md)**

---

## 🏠 Back to Main README

**[← Active Directory Home Lab](../README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*

---

> *A written guide shows you know the theory.*
> *A working lab shows you can implement it.*
> *A video walkthrough shows you can explain and demonstrate it.*
> *All three together make an outstanding portfolio piece.*
