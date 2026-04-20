# 🖥️ Active Directory Home Lab

A fully functional enterprise-grade Active Directory home lab built using Oracle VirtualBox, Windows Server 2022, and Windows 11. This project simulates a real-world IT environment including a Domain Controller, bulk user provisioning via PowerShell, and Group Policy management.

---

## 📋 Description

This home lab project was built to develop and demonstrate hands-on skills in:

- Windows Server administration
- Active Directory Domain Services (AD DS)
- DNS configuration and management
- PowerShell scripting and automation
- Group Policy Object (GPO) creation and deployment
- Virtual networking and VM management

The lab mirrors the core infrastructure found in most small to medium-sized businesses, making it directly relevant to roles such as System Administrator, IT Support Engineer, and Help Desk Technician.

---

## 🗂️ Table of Contents

- [01 - Pre-Requirements](#01---pre-requirements)
- [02 - Install Oracle VirtualBox](#02---install-oracle-virtualbox)
- [03 - Create the Domain Controller VM (DC01)](#03---create-the-domain-controller-vm-dc01)
- [04 - Install Windows Server 2022](#04---install-windows-server-2022)
- [05 - Configure DC01 Post-Installation](#05---configure-dc01-post-installation)
- [06 - Install Active Directory Domain Services](#06---install-active-directory-domain-services)
- [07 - Promote Server to Domain Controller](#07---promote-server-to-domain-controller)
- [08 - Create Organisational Unit Structure](#08---create-organisational-unit-structure)
- [09 - Bulk Create 110 Users with PowerShell](#09---bulk-create-110-users-with-powershell)
- [10 - Configure Networking for the Lab](#10---configure-networking-for-the-lab)
- [11 - Create the Windows 11 Workstation VM (WS01)](#11---create-the-windows-11-workstation-vm-ws01)
- [12 - Install Windows 11 on WS01](#12---install-windows-11-on-ws01)
- [13 - Join WS01 to the Domain](#13---join-ws01-to-the-domain)
- [14 - GPO 1 - Password Policy](#14---gpo-1---password-policy)
- [15 - GPO 2 - Account Lockout Policy](#15---gpo-2---account-lockout-policy)
- [16 - GPO 3 - Disable USB Storage](#16---gpo-3---disable-usb-storage)
- [17 - GPO 4 - Desktop Wallpaper](#17---gpo-4---desktop-wallpaper)
- [Lab Architecture Diagram](#lab-architecture-diagram)

---

## 01 - Pre-Requirements

Before starting this project ensure your physical host machine meets the following requirements.

### Hardware Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| RAM | 16 GB | 32 GB |
| CPU | Quad-core with VT-x/AMD-V | 6+ core modern CPU |
| Storage | 100 GB free (SSD) | 200 GB free (NVMe SSD) |
| OS | Windows 10/11, macOS, or Linux | Windows 11 |

### Verify Virtualisation is Enabled

On Windows open Task Manager → Performance → CPU and check that **Virtualisation: Enabled** is shown. If it says Disabled, reboot into BIOS/UEFI and enable Intel VT-x or AMD-V under CPU settings.

### Software and ISO Downloads Required

| Download | URL |
|----------|-----|
| Oracle VirtualBox | https://www.virtualbox.org/wiki/Downloads |
| VirtualBox Extension Pack | https://www.virtualbox.org/wiki/Downloads |
| Windows Server 2022 ISO | https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022 |
| Windows 11 ISO | https://www.microsoft.com/en-gb/software-download/windows11 |

> **Note:** Both Windows Server 2022 and Windows 11 are free evaluation editions valid for 180 days — more than enough for this lab.

---

## 02 - Install Oracle VirtualBox

1. Download the VirtualBox installer for your host OS from the official Oracle website
2. Run the installer and accept all defaults
3. When warned about network interruption during install click **Yes** — this installs a virtual network adapter and is expected
4. After installation open VirtualBox and go to **File → Tools → Extension Manager**
5. Click the add icon and browse to the downloaded `.vbox-extpack` file to install the Extension Pack
6. The Extension Pack version must match your VirtualBox version exactly

---

## 03 - Create the Domain Controller VM (DC01)

1. In VirtualBox click **"Create a new virtual machine"**
2. On the Name and OS screen set the following — **leave ISO Image blank**:

| Setting | Value |
|---------|-------|
| Name | `DC01` |
| ISO Image | Leave blank |
| Type | Microsoft Windows |
| Version | Windows 2022 (64-bit) |

3. On the Hardware screen set **RAM to 4096 MB** and **CPUs to 2**
4. On the Hard Disk screen create a new VHD **60 GB dynamically allocated**
5. Click Finish

> ⚠️ **Important:** Always leave the ISO blank when creating the VM. Selecting it during creation triggers the Unattended Installer which causes a licence error. Attach the ISO manually after creation.

### Attach the ISO Manually

1. Click **DC01 → Settings → Storage**
2. Click the **Empty** optical drive
3. Click the blue disc icon → **Choose a disk file**
4. Select your Windows Server 2022 ISO
5. Click OK

### Configure Additional Settings

Go to **DC01 → Settings** and configure:

- **System → Motherboard:** Enable EFI ✅, TPM 2.0
- **Display → Screen:** Video Memory 128 MB
- **Network → Adapter 1:** NAT (for internet access)
- **Network → Adapter 2:** Internal Network — Name: `LabNet`

---

## 04 - Install Windows Server 2022

1. Start DC01 and boot from the ISO
2. Select language, time, and keyboard layout then click **Next → Install now**
3. On the edition selection screen choose **Windows Server 2022 Standard Evaluation (Desktop Experience)**
4. Accept the licence terms
5. Choose **Custom: Install Windows only (advanced)**
6. Select the unallocated disk space and click **Next**
7. The installation will complete and the VM will reboot several times automatically
8. Set a strong Administrator password when prompted — for example `Lab@Admin2024!`

> **Tip:** Install VirtualBox Guest Additions after reaching the desktop for clipboard sharing and better screen resolution. Go to **Devices → Insert Guest Additions CD Image** inside the VM.

---

## 05 - Configure DC01 Post-Installation

### Rename the Computer to DC01

1. Open Server Manager → Local Server
2. Click the computer name hyperlink
3. Click **Change** and set the name to `DC01`
4. Restart when prompted

### Set a Static IP Address on the Internal Adapter

Open Network Connections. Identify **Ethernet 2** (the Internal Network adapter — it will show as disconnected). Right-click → Properties → IPv4 → Properties and set:

| Field | Value |
|-------|-------|
| IP Address | `192.168.100.10` |
| Subnet Mask | `255.255.255.0` |
| Default Gateway | Leave blank |
| Preferred DNS | `127.0.0.1` |

Leave the NAT adapter (Ethernet) set to **Obtain an IP address automatically**.

### Verify in PowerShell

```powershell
Get-NetIPConfiguration
```

You should see `192.168.100.10` on Ethernet 2 and a `10.0.2.x` address on the NAT adapter.

---

## 06 - Install Active Directory Domain Services

1. Open **Server Manager → Manage → Add Roles and Features**
2. Click Next through to **Server Roles**
3. Tick **Active Directory Domain Services**
4. When the popup appears click **Add Features**
5. Click Next through Features and the AD DS info page
6. On the Confirmation screen click **Install**
7. Wait for installation to complete — do not close the wizard

---

## 07 - Promote Server to Domain Controller

After the AD DS role installs, a yellow flag appears in Server Manager. Click it then click **"Promote this server to a domain controller"**.

Work through the wizard with these settings:

| Screen | Setting | Value |
|--------|---------|-------|
| Deployment Configuration | Forest | Add a new forest |
| Root domain name | | `Lab.local` |
| Domain Controller Options | Forest/Domain functional level | Windows Server 2016 |
| | DNS Server | ✅ Enabled |
| | Global Catalog | ✅ Enabled |
| | DSRM Password | `DSRM@Lab2024!` (write this down) |
| Additional Options | NetBIOS name | `LAB` (auto-filled) |
| Paths | All paths | Leave as default |

Click **Install** — the server will automatically restart and join its own domain.

### Verify Active Directory is Working

```powershell
# Confirm domain details
Get-ADDomain

# Confirm domain password policy
Get-ADDefaultDomainPasswordPolicy
```

---

## 08 - Create Organisational Unit Structure

Open **Active Directory Users and Computers** (Server Manager → Tools → ADUC).

Right-click **Lab.local** → New → Organizational Unit and create the following structure:

```
Lab.local
├── _COMPUTERS
├── _GROUPS
└── _USERS
    ├── IT
    ├── HR
    ├── Finance
    └── Management
```

The underscore prefix makes these OUs sort to the top of the list for easy navigation.

---

## 09 - Bulk Create 110 Users with PowerShell

### Create the Scripts Folder

```powershell
New-Item -ItemType Directory -Path C:\Scripts
```

### Create and Run the Script

Open Notepad with:

```powershell
notepad C:\Scripts\New-BulkADUsers.ps1
```

Paste the following script and save:

```powershell
# Configuration
$DomainDN        = "DC=Lab,DC=local"
$BaseOU          = "OU=_USERS,$DomainDN"
$DefaultPassword = ConvertTo-SecureString "Welcome1@Lab" -AsPlainText -Force
$UPNSuffix       = "lab.local"

$FirstNames = @(
    "James","Mary","John","Patricia","Robert","Jennifer","Michael","Linda",
    "William","Barbara","David","Elizabeth","Richard","Susan","Joseph","Jessica",
    "Thomas","Sarah","Charles","Karen","Christopher","Lisa","Daniel","Nancy",
    "Matthew","Margaret","Anthony","Betty","Mark","Sandra","Donald","Ashley",
    "Steven","Dorothy","Paul","Kimberly","Andrew","Emily","Kenneth","Donna",
    "Joshua","Michelle","Kevin","Carol","Brian","Amanda","George","Melissa",
    "Edward","Deborah","Ronald","Stephanie","Timothy","Rebecca","Jason","Sharon",
    "Jeffrey","Laura","Ryan","Cynthia","Jacob","Kathleen","Gary","Amy",
    "Nicholas","Angela","Eric","Shirley","Jonathan","Anna","Stephen","Brenda",
    "Larry","Pamela","Justin","Emma","Scott","Nicole","Brandon","Helen",
    "Benjamin","Samantha","Samuel","Katherine","Raymond","Christine","Gregory",
    "Debra","Frank","Rachel","Alexander","Carolyn","Patrick","Janet","Jack"
)

$LastNames = @(
    "Smith","Johnson","Williams","Brown","Jones","Garcia","Miller","Davis",
    "Rodriguez","Martinez","Hernandez","Lopez","Gonzalez","Wilson","Anderson",
    "Thomas","Taylor","Moore","Jackson","Martin","Lee","Perez","Thompson",
    "White","Harris","Sanchez","Clark","Ramirez","Lewis","Robinson","Walker",
    "Young","Allen","King","Wright","Scott","Torres","Nguyen","Hill","Flores",
    "Green","Adams","Nelson","Baker","Hall","Rivera","Campbell","Mitchell",
    "Carter","Roberts","Phillips","Evans","Turner","Parker","Collins",
    "Edwards","Stewart","Morris","Murphy","Cook","Rogers","Morgan","Peterson",
    "Cooper","Reed","Bailey","Bell","Gomez","Kelly","Howard","Ward","Cox"
)

$Departments = @(
    @{ Name="IT";         OU="IT";         Title="Systems Administrator" },
    @{ Name="IT";         OU="IT";         Title="Help Desk Technician" },
    @{ Name="IT";         OU="IT";         Title="Network Engineer" },
    @{ Name="IT";         OU="IT";         Title="Security Analyst" },
    @{ Name="HR";         OU="HR";         Title="HR Manager" },
    @{ Name="HR";         OU="HR";         Title="HR Coordinator" },
    @{ Name="Finance";    OU="Finance";    Title="Financial Analyst" },
    @{ Name="Finance";    OU="Finance";    Title="Accountant" },
    @{ Name="Management"; OU="Management"; Title="Department Manager" },
    @{ Name="Management"; OU="Management"; Title="Team Lead" }
)

$UsedSamNames = [System.Collections.Generic.HashSet[string]]::new(
    [System.StringComparer]::OrdinalIgnoreCase)
Get-ADUser -Filter * -Properties SamAccountName |
    ForEach-Object { $null = $UsedSamNames.Add($_.SamAccountName) }

$UsersCreated = 0
$UsersSkipped = 0
$TotalToCreate = 110
$random = New-Object System.Random

while ($UsersCreated -lt $TotalToCreate) {

    $First = $FirstNames[$random.Next(0, $FirstNames.Count)]
    $Last  = $LastNames[$random.Next(0, $LastNames.Count)]

    $BaseSam = ($First.Substring(0,1) + $Last).ToLower() -replace '[^a-z0-9]', ''
    if ($BaseSam.Length -gt 18) { $BaseSam = $BaseSam.Substring(0,18) }

    $SamName = $BaseSam
    $suffix  = 2
    while ($UsedSamNames.Contains($SamName)) {
        $SamName = $BaseSam + $suffix
        $suffix++
    }
    $null = $UsedSamNames.Add($SamName)

    $Dept        = $Departments[$random.Next(0, $Departments.Count)]
    $TargetOU    = "OU=$($Dept.OU),$BaseOU"
    $UPN         = "$SamName@$UPNSuffix"
    $DisplayName = "$First $Last"

    try {
        New-ADUser `
            -SamAccountName       $SamName `
            -UserPrincipalName    $UPN `
            -Name                 $DisplayName `
            -GivenName            $First `
            -Surname              $Last `
            -DisplayName          $DisplayName `
            -EmailAddress         "$SamName@$UPNSuffix" `
            -Department           $Dept.Name `
            -Title                $Dept.Title `
            -Description          "$($Dept.Title) - $($Dept.Name)" `
            -Path                 $TargetOU `
            -AccountPassword      $DefaultPassword `
            -Enabled              $true `
            -PasswordNeverExpires $false `
            -ChangePasswordAtLogon $true `
            -ErrorAction Stop

        $UsersCreated++
        Write-Host "[$UsersCreated/$TotalToCreate] Created: $DisplayName ($SamName) - $($Dept.Name)" `
            -ForegroundColor Green

    } catch {
        Write-Warning "Failed to create $DisplayName ($SamName): $_"
        $UsersSkipped++
    }
}

Write-Host "`n--- Done ---" -ForegroundColor Yellow
Write-Host "Users created : $UsersCreated" -ForegroundColor Green
Write-Host "Errors/skipped: $UsersSkipped" -ForegroundColor Red
```

### Run the Script

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
& "C:\Scripts\New-BulkADUsers.ps1"
```

### Verify Users Were Created

```powershell
Get-ADUser -Filter * -SearchBase "OU=_USERS,DC=Lab,DC=local" `
    -SearchScope Subtree `
    -Properties Department |
    Group-Object Department |
    Select-Object Name, Count |
    Format-Table -AutoSize
```

---

## 10 - Configure Networking for the Lab

### Network Architecture

| VM | Adapter | Type | IP Address |
|----|---------|------|------------|
| DC01 | Adapter 1 | NAT | 10.0.2.15 (auto) |
| DC01 | Adapter 2 | Internal Network — LabNet | 192.168.100.10 (static) |
| WS01 | Adapter 1 | Internal Network — LabNet | 192.168.100.20 (static) |

> **Critical:** Both VMs must use the exact same Internal Network name — `LabNet`. The name is case-sensitive.

### Set Static IP on WS01 via PowerShell

```powershell
# Set static IP
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress "192.168.100.20" -PrefixLength 24 -DefaultGateway "192.168.100.10"

# Point DNS to DC01
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "192.168.100.10"
```

---

## 11 - Create the Windows 11 Workstation VM (WS01)

Create a new VM with the following settings — again **leave ISO blank**:

| Setting | Value |
|---------|-------|
| Name | `WS01` |
| ISO Image | Leave blank |
| Type | Microsoft Windows |
| Version | Windows 11 (64-bit) |
| RAM | 4096 MB |
| CPUs | 2 |
| Storage | 50 GB dynamically allocated |

### Configure Settings After Creation

Go to **WS01 → Settings** and configure:

- **System → Motherboard:** Enable EFI ✅, TPM 2.0, Secure Boot ❌ (untick this)
- **Display → Screen:** Video Memory 128 MB
- **Network → Adapter 1:** Internal Network — Name: `LabNet`
- **Storage:** Attach Windows 11 ISO to the optical drive

---

## 12 - Install Windows 11 on WS01

1. Start WS01 — it will boot from the ISO automatically since the hard disk is empty
2. Select language and keyboard layout then click **Next → Install now**
3. Select **Windows 11 Pro** ← required for domain joining
4. Accept the licence terms and choose **Custom: Install Windows only**
5. On the network connection screen press **Shift + F10** to open a command prompt and run:

```
OOBE\BYPASSNRO
```

6. The VM restarts — on the network screen this time click **"I don't have internet"**
7. Click **"Continue with limited setup"**
8. Create a local account — Username: `LocalAdmin`, Password: `Local@Admin1`

> **Why bypass the network requirement?** WS01 will be domain-joined — we want a local admin account as backup, not a Microsoft account. Domain accounts handle all logins.

---

## 13 - Join WS01 to the Domain

### Prerequisites Check

Before joining, verify connectivity from WS01:

```powershell
# Test connectivity to DC01
ping 192.168.100.10

# Test DNS resolution
nslookup Lab.local 192.168.100.10

# Test LDAP port is reachable
Test-NetConnection -ComputerName 192.168.100.10 -Port 389
```

All three must succeed before proceeding.

### Join the Domain

```powershell
Add-Computer -DomainName "Lab.local" -Credential "LAB\Administrator" -Restart
```

Enter the DC01 Administrator password when prompted. WS01 will restart automatically.

### Verify the Join on DC01

```powershell
Get-ADComputer -Filter * | Select-Object Name, DNSHostName
```

WS01 should appear in the list. Also check **Active Directory Users and Computers → _COMPUTERS** OU.

### Log In with a Domain User

After the restart, at the login screen click **Other user** and enter:

```
Username: LAB\rgomez
Password: Welcome1@Lab
```

The user will be prompted to change their password on first login — this is correct behaviour.

---

## 14 - GPO 1 - Password Policy

Open **Group Policy Management** on DC01 (Server Manager → Tools → Group Policy Management).

### Edit the Default Domain Policy

Right-click **Default Domain Policy** → **Edit** and navigate to:

```
Computer Configuration → Policies → Windows Settings
→ Security Settings → Account Policies → Password Policy
```

Configure each setting:

| Setting | Value |
|---------|-------|
| Enforce password history | 5 passwords |
| Maximum password age | 90 days |
| Minimum password age | 1 day |
| Minimum password length | 10 characters |
| Password must meet complexity requirements | Enabled |
| Store passwords using reversible encryption | Disabled |

### Apply and Verify

```powershell
gpupdate /force
Get-ADDefaultDomainPasswordPolicy
```

Expected output confirms:
- `MinPasswordLength: 10`
- `MaxPasswordAge: 90 days`
- `ComplexityEnabled: True`
- `PasswordHistoryCount: 5`

---

## 15 - GPO 2 - Account Lockout Policy

Still in **Default Domain Policy → Edit**, navigate to:

```
Computer Configuration → Policies → Windows Settings
→ Security Settings → Account Policies → Account Lockout Policy
```

Configure each setting:

| Setting | Value |
|---------|-------|
| Account lockout threshold | 5 invalid attempts |
| Account lockout duration | 30 minutes |
| Reset account lockout counter after | 30 minutes |

### Apply and Verify

```powershell
gpupdate /force
Get-ADDefaultDomainPasswordPolicy | Select-Object LockoutThreshold, LockoutDuration, LockoutObservationWindow
```

### Useful Account Management Commands

```powershell
# Find locked out accounts
Search-ADAccount -LockedOut

# Unlock a specific account
Unlock-ADAccount -Identity jsmith

# Reset a password
Set-ADAccountPassword -Identity jsmith -NewPassword (ConvertTo-SecureString "NewP@ss1!" -AsPlainText -Force) -Reset
```

---

## 16 - GPO 3 - Disable USB Storage

### Create the GPO

In Group Policy Management right-click **Lab.local** → **Create a GPO in this domain and Link it here** → name it `Disable USB Storage`.

### Edit the GPO

Right-click **Disable USB Storage** → **Edit** and navigate to:

```
Computer Configuration → Policies → Administrative Templates
→ System → Removable Storage Access
```

Double-click **"All Removable Storage classes: Deny all access"** → select **Enabled** → click OK.

### Link to _COMPUTERS OU

Right-click **_COMPUTERS** in the left panel → **Link an Existing GPO** → select **Disable USB Storage** → click OK.

### Apply and Verify on WS01

```powershell
gpupdate /force
gpresult /r
```

Look for **Disable USB Storage** listed under **Applied Group Policy Objects** in the Computer Settings section.

---

## 17 - GPO 4 - Desktop Wallpaper

### Create a Shared Wallpaper Folder on DC01

```powershell
# Create the folder
New-Item -ItemType Directory -Path "C:\Wallpaper"

# Share it so domain computers can access it
New-SmbShare -Name "Wallpaper" -Path "C:\Wallpaper" -FullAccess "Everyone"
```

### Generate a Wallpaper Image via PowerShell

```powershell
Add-Type -AssemblyName System.Drawing

$width    = 1920
$height   = 1080
$bitmap   = New-Object System.Drawing.Bitmap($width, $height)
$graphics = [System.Drawing.Graphics]::FromImage($bitmap)

$bgColour  = [System.Drawing.Color]::FromArgb(0, 35, 102)
$graphics.Clear($bgColour)

$font      = New-Object System.Drawing.Font("Arial", 48, [System.Drawing.FontStyle]::Bold)
$fontSmall = New-Object System.Drawing.Font("Arial", 28)
$brush     = New-Object System.Drawing.SolidBrush([System.Drawing.Color]::White)

$graphics.DrawString("Lab.local Domain", $font, $brush, 600, 400)
$graphics.DrawString("Managed by Group Policy — Unauthorised access prohibited", $fontSmall, $brush, 350, 500)
$graphics.DrawString("DC01 — Windows Server 2022 Active Directory", $fontSmall, $brush, 430, 560)

$bitmap.Save("C:\Wallpaper\wallpaper.jpg", [System.Drawing.Imaging.ImageFormat]::Jpeg)
$graphics.Dispose()
$bitmap.Dispose()

Write-Host "Wallpaper created!" -ForegroundColor Green
```

### Create the GPO

In Group Policy Management right-click **Lab.local** → **Create a GPO in this domain and Link it here** → name it `Desktop Wallpaper`.

### Edit the GPO

Right-click **Desktop Wallpaper** → **Edit** and navigate to:

```
User Configuration → Policies → Administrative Templates
→ Desktop → Desktop
```

Double-click **"Desktop Wallpaper"** and configure:

| Setting | Value |
|---------|-------|
| Status | Enabled |
| Wallpaper Name | `\\DC01\Wallpaper\wallpaper.jpg` |
| Wallpaper Style | Fill |

### Link to _USERS OU

Right-click **_USERS** → **Link an Existing GPO** → select **Desktop Wallpaper** → OK.

### Apply and Test

On WS01:

```powershell
gpupdate /force
```

Log off and log back in as a domain user. The dark blue Lab.local wallpaper should appear automatically.

### Generate a Full GPO Report

```powershell
gpresult /h C:\gporeport.html
start C:\gporeport.html
```

---

## Lab Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                Oracle VirtualBox Host (Physical PC)          │
│                                                             │
│  ┌──────────────────────┐    ┌──────────────────────────┐   │
│  │        DC01          │    │          WS01            │   │
│  │  Windows Server 2022 │    │      Windows 11 Pro      │   │
│  │                      │    │                          │   │
│  │  ┌────────────────┐  │    │  ┌────────────────────┐  │   │
│  │  │  AD DS / DNS   │  │    │  │  Domain-joined     │  │   │
│  │  │  LDAP/Kerberos │  │    │  │  GPOs applied      │  │   │
│  │  └────────────────┘  │    │  └────────────────────┘  │   │
│  │                      │    │                          │   │
│  │  Adapter 1: NAT      │    │  Adapter 1: LabNet       │   │
│  │  10.0.2.15           │    │  192.168.100.20          │   │
│  │  Adapter 2: LabNet   │    │  DNS → 192.168.100.10    │   │
│  │  192.168.100.10      │    │                          │   │
│  └──────────┬───────────┘    └────────────┬─────────────┘   │
│             │                             │                  │
│             └─────────────┬───────────────┘                  │
│                           │                                  │
│              ┌────────────▼────────────┐                     │
│              │  LabNet Internal Network │                     │
│              │   192.168.100.0/24      │                     │
│              │   Domain: Lab.local     │                     │
│              └─────────────────────────┘                     │
└─────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Technologies Used

- Oracle VirtualBox 7.x
- Windows Server 2022 Standard Evaluation
- Windows 11 Pro
- Active Directory Domain Services (AD DS)
- DNS Server
- Group Policy Management
- PowerShell 5.1

---

## 📁 Repository Structure

```
Active-Directory-Home-Lab/
├── README.md
└── Scripts/
    └── New-BulkADUsers.ps1
```

---

## 👤 Author

Built as a hands-on IT home lab project to develop real-world Active Directory administration skills.

---

## 📄 Licence

This project is for educational purposes only. Windows Server 2022 and Windows 11 evaluation licences are provided by Microsoft and are valid for 180 days.
