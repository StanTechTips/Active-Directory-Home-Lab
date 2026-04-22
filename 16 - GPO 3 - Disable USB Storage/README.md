# 16 - GPO 3 — Disable USB Storage

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Intermediate-orange?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/VM-DC01-purple?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Tool-Group_Policy-red?style=for-the-badge&logo=windows)

> USB storage devices are one of the most common vectors for data theft
> and malware introduction in enterprise environments.
> A single uncontrolled USB drive can exfiltrate gigabytes of sensitive data
> or introduce ransomware that cripples an entire organisation.
> This GPO blocks all removable storage access on domain computers —
> one of the most frequently deployed security policies in real IT environments.

---

## 📑 Table of Contents

- [Why USB Storage Restriction Matters](#-why-usb-storage-restriction-matters)
- [How This GPO Works](#-how-this-gpo-works)
- [GPO vs Previous Policies — Key Difference](#-gpo-vs-previous-policies--key-difference)
- [Create the Disable USB Storage GPO](#-create-the-disable-usb-storage-gpo)
- [Configure the GPO Setting](#-configure-the-gpo-setting)
- [Link the GPO to the _COMPUTERS OU](#-link-the-gpo-to-the-_computers-ou)
- [Apply and Verify the Policy](#-apply-and-verify-the-policy)
- [Test the USB Block is Working](#-test-the-usb-block-is-working)
- [Generate a Full GPO Report](#-generate-a-full-gpo-report)
- [Take a Snapshot](#-take-a-snapshot)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)

---

## 🛡️ Why USB Storage Restriction Matters

USB storage restriction is a **Data Loss Prevention (DLP)** control —
one of the most commonly required policies in regulated industries:

```
┌─────────────────────────────────────────────────────────────┐
│          Real World USB Threat Scenarios                     │
│                                                             │
│   📤 Data Exfiltration                                      │
│      Disgruntled employee copies customer database          │
│      to a USB drive and walks out with it                   │
│                                                             │
│   🦠 Malware Introduction                                   │
│      Employee finds a USB drive in the car park             │
│      Plugs it in out of curiosity                           │
│      AutoRun launches ransomware — network encrypted        │
│                                                             │
│   🔍 Compliance Requirements                                │
│      PCI DSS, HIPAA, ISO 27001, Cyber Essentials           │
│      All require controls on removable media                │
│                                                             │
│   👤 Insider Threat                                         │
│      Contractor copies proprietary source code              │
│      before their contract ends                             │
└─────────────────────────────────────────────────────────────┘
```

**This GPO is deployed in the vast majority of enterprise environments.**
Knowing how to configure and troubleshoot it is a core IT admin skill.

---

## ⚙️ How This GPO Works

The setting we configure is found under **Removable Storage Access**
in Administrative Templates. It works at the Windows kernel level —
blocking read and write access to all removable storage devices
before the operating system even presents them to the user.

```
┌─────────────────────────────────────────────────────────────┐
│          What "All Removable Storage: Deny All" Blocks       │
│                                                             │
│   ✅ USB flash drives and thumb drives                      │
│   ✅ External USB hard drives                               │
│   ✅ SD cards and memory card readers                       │
│   ✅ External SSDs connected via USB                        │
│   ✅ Portable media players with storage                    │
│                                                             │
│          What It Does NOT Block                             │
│                                                             │
│   ✅ USB keyboards and mice — still work                    │
│   ✅ USB headsets and webcams — still work                  │
│   ✅ USB network adapters — still work                      │
│   ✅ Network drives (mapped drives) — still work            │
│   ✅ Internal hard drives and SSDs — still work             │
└─────────────────────────────────────────────────────────────┘
```

> 💡 The policy targets **storage class** devices specifically —
> not all USB devices. HID devices (keyboards, mice) and audio
> devices are entirely unaffected.

---

## 🎯 GPO vs Previous Policies — Key Difference

This GPO introduces two important new concepts compared to GPOs 1 and 2:

### Difference 1 — New GPO (Not Default Domain Policy)

GPOs 1 and 2 were configured inside the **Default Domain Policy**.
This GPO is a **brand new GPO** that we create from scratch and name ourselves.

```
Default Domain Policy    → Modified for password and lockout settings
Disable USB Storage      → New GPO we create for USB restriction  ← This step
Desktop Wallpaper        → New GPO we create for wallpaper        ← Next step
```

Creating separate GPOs for separate purposes is best practice —
it makes each policy independently manageable, auditable, and reversible.

---

### Difference 2 — Linked to an OU (Not the Domain Root)

GPOs 1 and 2 apply to **everything** in the domain because they are
linked at the domain root level.

This GPO is linked specifically to the **_COMPUTERS OU** — so it
only applies to domain computers, not to user accounts.

```
Lab.local (domain root)
├── Default Domain Policy     → Applies to users AND computers ✅
│
├── _COMPUTERS OU
│   └── Disable USB Storage   → Applies to computers ONLY ✅ ← This GPO
│       (WS01 is here)
│
└── _USERS OU
    └── Desktop Wallpaper     → Applies to users ONLY ← Next GPO
```

> 💡 **Why link to _COMPUTERS and not the domain root?**
> USB restriction is a **Computer Configuration** policy.
> By linking to `_COMPUTERS` we ensure only managed domain workstations
> are affected — not domain controllers or other server objects.
> This is precise, targeted Group Policy — exactly how it should be done.

---

## 🆕 Create the Disable USB Storage GPO

### Step 1 — Open Group Policy Management

On **DC01**:
```
Server Manager → Tools → Group Policy Management
```

Or press **Windows + R** → type `gpmc.msc` → Enter.

---

### Step 2 — Create the New GPO

In the left panel right-click **"Lab.local"** (the domain) →
click **"Create a GPO in this domain and Link it here..."**

> ⚠️ **Note:** We create the GPO at the domain level first, then
> move the link to _COMPUTERS. This is the standard workflow in GPMC.

A dialog box appears asking for a name.

Type exactly:
```
Disable USB Storage
```

Click **OK**.

The new GPO **"Disable USB Storage"** now appears in the left panel
under Lab.local — with a blue document icon.

---

## ⚙️ Configure the GPO Setting

### Step 3 — Open the GPO for Editing

Right-click **"Disable USB Storage"** → click **"Edit"**.

The **Group Policy Management Editor** opens.

---

### Step 4 — Navigate to Removable Storage Access

In the left panel navigate to:

```
Disable USB Storage [DC01.Lab.local]
└── Computer Configuration
    └── Policies
        └── Administrative Templates
            └── System
                └── Removable Storage Access    ← Click here
```

Click on **"Removable Storage Access"**.

A list of settings appears in the right panel — you will see many options
covering different classes of removable storage separately.

---

### Step 5 — Enable the Main Setting

Find and double-click:

```
All Removable Storage classes: Deny all access
```

The settings window opens showing:

```
○ Not Configured
○ Enabled
○ Disabled

Supported on: Windows Vista and later...
```

Select **"Enabled"**.

```
✅ Enabled
```

Click **Apply** → Click **OK**.

---

### Step 6 — Confirm the Setting is Enabled

Back in the right panel the setting should now show:

```
Setting                                        State
-------                                        -----
All Removable Storage classes: Deny all access  Enabled   ✅
```

---

### Step 7 — Close the Editor

Close the **Group Policy Management Editor**.

---

## 🔗 Link the GPO to the _COMPUTERS OU

The GPO was created linked to the domain root — but we want it
linked specifically to the **_COMPUTERS OU** where WS01 lives.

### Step 8 — Remove the Domain-Level Link

In Group Policy Management click on **"Lab.local"** in the left panel.

In the right panel under **"Linked Group Policy Objects"** tab you will see
**"Disable USB Storage"** linked at the domain level.

Right-click **"Disable USB Storage"** in this list → click **"Delete Link"**.

> ⚠️ **Click "Delete Link" — NOT "Delete GPO".**
> Deleting the link only removes the association at this level.
> The GPO itself still exists — we will link it to _COMPUTERS instead.

A warning appears:
```
Do you want to delete this link? This will not delete the GPO itself.
```

Click **OK**.

---

### Step 9 — Link the GPO to _COMPUTERS OU

In the left panel right-click **"_COMPUTERS"** →
click **"Link an Existing GPO..."**

A list of all GPOs in the domain appears.

Click on **"Disable USB Storage"** to select it.

Click **OK**.

**"Disable USB Storage"** now appears linked under **_COMPUTERS** in the left panel:

```
Lab.local
├── Default Domain Policy
├── _COMPUTERS
│   └── 📋 Disable USB Storage   ✅ (linked here)
└── _USERS
```

---

## ✅ Apply and Verify the Policy

### Force Policy Update on WS01

On **WS01** open **PowerShell as Administrator** and run:

```powershell
gpupdate /force
```

Expected output:

```
Updating policy...

Computer Policy update has completed successfully.
User Policy update has completed successfully.
```

> 💡 **Computer Configuration GPOs** (like USB restriction) sometimes
> require a **full restart** to fully apply — not just gpupdate.
> If the USB is still accessible after gpupdate, restart WS01 and test again.

---

### Restart WS01 to Fully Apply the Computer Policy

```powershell
Restart-Computer -Force
```

After WS01 restarts and you log back in as a domain user, the USB
restriction will be fully active.

---

### Verify the GPO is Applied on WS01

On **WS01** open PowerShell and run:

```powershell
gpresult /r
```

Look for **"Disable USB Storage"** in the output:

```
COMPUTER SETTINGS
-----------------
    Applied Group Policy Objects
    ----------------------------
        Disable USB Storage          ← ✅ Confirmed applied
        Default Domain Policy
```

---

### Verify via Registry on WS01

The GPO writes to the Windows registry. Confirm it is there:

```powershell
# Check the registry key the GPO creates
Get-ItemProperty `
    -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\{53f5630d-b6bf-11d0-94f2-00a0c91efb8b}" `
    -ErrorAction SilentlyContinue
```

If the GPO is applied you will see:

```
Deny_Read  : 1
Deny_Write : 1
```

Both set to `1` confirms all removable storage access is denied. ✅

---

### Verify GPO Linkage on DC01

On **DC01** in PowerShell:

```powershell
# Confirm the GPO is linked to _COMPUTERS OU
Get-GPInheritance -Target "OU=_COMPUTERS,DC=Lab,DC=local" |
    Select-Object -ExpandProperty InheritedGpoLinks |
    Select-Object DisplayName, Enabled, Enforced
```

Expected output:

```
DisplayName          Enabled  Enforced
-----------          -------  --------
Disable USB Storage  True     False    ✅
Default Domain Policy True    False    ✅
```

---

## 🧪 Test the USB Block is Working

### Test Method 1 — Via VirtualBox USB Passthrough

If you have a physical USB drive available on your host PC:

**1.** Plug the USB drive into your host PC.

**2.** In the VirtualBox WS01 window menu click:
```
Devices → USB → [Your USB drive name]
```

**3.** Inside WS01 open **File Explorer**.

**4.** The USB drive will appear under **"This PC"** but when you try to open it you will see:

```
┌────────────────────────────────────────────────────┐
│                                                    │
│   G:\ is not accessible.                          │
│                                                    │
│   This operation has been cancelled due to         │
│   restrictions in effect on this computer.         │
│   Please contact your system administrator.        │
│                                                    │
│                        [ OK ]                      │
└────────────────────────────────────────────────────┘
```

✅ The USB restriction GPO is working correctly.

---

### Test Method 2 — Via PowerShell on WS01

```powershell
# Check if any removable drives are accessible
Get-WmiObject -Class Win32_DiskDrive |
    Where-Object { $_.MediaType -like "*Removable*" } |
    Select-Object Model, MediaType, Status
```

Even if a USB device is connected it will show as present but inaccessible.

---

### What the User Sees vs What IT Sees

```
┌──────────────────────────────────────────────────┐
│  User experience on WS01                         │
│                                                  │
│  ❌ Cannot read files from USB                   │
│  ❌ Cannot write files to USB                    │
│  ❌ Cannot copy files to USB                     │
│  ✅ USB keyboard still works                     │
│  ✅ USB mouse still works                        │
└──────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────┐
│  IT administrator view on DC01                   │
│                                                  │
│  ✅ GPO deployed and linked to _COMPUTERS        │
│  ✅ Confirmed in gpresult /r on WS01             │
│  ✅ Registry keys confirm policy active          │
│  ✅ Can remove GPO link instantly if needed      │
└──────────────────────────────────────────────────┘
```

---

## 📊 Generate a Full GPO Report

Generate a comprehensive HTML report showing all GPOs applied to WS01.
This is a real-world skill used to audit and document GPO deployments.

On **WS01** run:

```powershell
# Generate the full GPO results report
gpresult /h "C:\GPOReport.html" /f

# Open it automatically
Start-Process "C:\GPOReport.html"
```

The report opens in your browser and shows:

- Every GPO applied to WS01 (computer and user)
- Every setting enforced by each GPO
- Which GPOs were not applied and why
- Security group memberships
- WMI filter results

On **DC01** you can also generate a domain-wide GPO report:

```powershell
# Generate a report for a specific computer from DC01
Get-GPResultantSetOfPolicy `
    -Computer "WS01" `
    -ReportType Html `
    -Path "C:\Scripts\WS01-GPOReport.html"

Start-Process "C:\Scripts\WS01-GPOReport.html"
```

---

## 📸 Take a Snapshot

USB Storage restriction is deployed and verified.

In VirtualBox go to:
```
Machine → Take Snapshot
```

Name it:
```
DC01 - GPO3 Disable USB Storage Configured
```

Click **OK**.

Also take a snapshot on **WS01**:

```
Machine → Take Snapshot
Name: WS01 - GPO3 USB Restriction Applied
```

---

## 🛠️ Troubleshooting

### Issue: "Disable USB Storage" not showing in gpresult /r on WS01

The GPO is not reaching WS01 — either the link is wrong or WS01
is not in the _COMPUTERS OU.

**Fix — Step 1: Confirm WS01 is in the _COMPUTERS OU on DC01:**

```powershell
Get-ADComputer -Identity "WS01" | Select-Object DistinguishedName
```

Must show: `CN=WS01,OU=_COMPUTERS,DC=Lab,DC=local`

If it shows `CN=Computers` instead:

```powershell
Get-ADComputer -Identity "WS01" |
    Move-ADObject -TargetPath "OU=_COMPUTERS,DC=Lab,DC=local"
```

**Fix — Step 2: Confirm GPO is linked to _COMPUTERS:**

In Group Policy Management expand `_COMPUTERS` in the left panel.
Confirm "Disable USB Storage" appears underneath it.

---

### Issue: USB drive is still accessible after gpupdate /force

Computer Configuration GPOs require a full restart to apply —
not just a gpupdate.

**Fix:**

```powershell
# Restart WS01
Restart-Computer -Force
```

After the restart log back in and test the USB drive again.

---

### Issue: GPO shows as applied in gpresult but USB still works

The registry keys may not have been written correctly, or a later
policy is overriding this one.

**Fix:**

```powershell
# Force a complete policy refresh with a reboot flag
gpupdate /force /boot

# After reboot check the registry directly
Get-ItemProperty `
    -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\RemovableStorageDevices\*"
```

---

### Issue: "Delete Link" option not available when right-clicking the GPO

You may be right-clicking on the GPO object itself rather than the link.

**Fix:**
1. Click on **"Lab.local"** in the left panel
2. In the right panel click the **"Linked Group Policy Objects"** tab
3. Right-click **"Disable USB Storage"** in this tab's list
4. The **"Delete Link"** option will be available here

---

### Issue: GPO linked to _COMPUTERS but Get-GPInheritance shows it as blocked

GPO inheritance may be blocked on the _COMPUTERS OU.

**Fix:**

```powershell
# Check if inheritance is blocked
Get-GPInheritance -Target "OU=_COMPUTERS,DC=Lab,DC=local" |
    Select-Object GpoInheritanceBlocked
```

If `GpoInheritanceBlocked: Yes` — in Group Policy Management right-click
**_COMPUTERS** and untick **"Block Inheritance"**.

---

## ✅ Verification Checklist

```
GPO Creation
[ ] New GPO created named "Disable USB Storage"
[ ] GPO created under Lab.local domain

GPO Configuration
[ ] Navigated to Computer Configuration → Administrative Templates
    → System → Removable Storage Access
[ ] "All Removable Storage classes: Deny all access" set to Enabled
[ ] Setting shows "Enabled" in the right panel
[ ] Group Policy Management Editor closed

GPO Linking
[ ] Domain-level link removed from Lab.local
[ ] GPO linked to _COMPUTERS OU correctly
[ ] "Disable USB Storage" visible under _COMPUTERS in GPMC left panel

WS01 Verification
[ ] WS01 is in OU=_COMPUTERS,DC=Lab,DC=local (not CN=Computers)
[ ] gpupdate /force ran on WS01
[ ] WS01 restarted after gpupdate
[ ] gpresult /r shows "Disable USB Storage" under Applied GPOs
[ ] Registry shows Deny_Read: 1 and Deny_Write: 1

USB Test
[ ] USB drive blocked with "operation cancelled due to restrictions" message
[ ] USB keyboard/mouse still functioning correctly

GPO Report
[ ] gpresult /h report generated and opened successfully
[ ] Disable USB Storage visible in the HTML report

Snapshots
[ ] DC01 snapshot: "DC01 - GPO3 Disable USB Storage Configured"
[ ] WS01 snapshot: "WS01 - GPO3 USB Restriction Applied"

Ready to proceed
[ ] USB restriction is live and tested on WS01
[ ] GPO is correctly linked to _COMPUTERS OU only
[ ] Ready to configure the Desktop Wallpaper GPO — the final step
```

---

## ➡️ Next Step

USB Storage is blocked on all domain workstations.
This is the final GPO — Desktop Wallpaper pushed to all domain users,
visually confirming your entire Group Policy infrastructure is working.

**[17 - GPO 4 Desktop Wallpaper →](../17%20-%20GPO%204%20-%20Desktop%20Wallpaper/README.md)**

---

## ⬅️ Previous Step

**[← 15 - GPO 2 Account Lockout Policy](../15%20-%20GPO%202%20-%20Account%20Lockout/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
