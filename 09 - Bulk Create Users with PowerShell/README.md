# 09 - Bulk Create Users with PowerShell

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Intermediate-orange?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/VM-DC01-purple?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Tool-PowerShell-2E6DBD?style=for-the-badge&logo=powershell)

> Manually creating 110 user accounts through the GUI would take hours.
> PowerShell does it in under 60 seconds.
> This step demonstrates one of the most valuable real-world IT administration
> skills — automating repetitive tasks with scripting.
> The script creates realistic users with names, departments, job titles,
> email addresses, and correct OU placement — all automatically.

---

## 📑 Table of Contents

- [What the Script Does](#-what-the-script-does)
- [Understanding the Script Components](#-understanding-the-script-components)
- [Create the Scripts Folder](#-create-the-scripts-folder)
- [Create the Script File](#-create-the-script-file)
- [The Full Script](#-the-full-script)
- [Run the Script](#-run-the-script)
- [Verify the Results](#-verify-the-results)
- [Useful AD User Management Commands](#-useful-ad-user-management-commands)
- [Take a Snapshot](#-take-a-snapshot)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)

---

## 🧠 What the Script Does

The PowerShell script automates the creation of 110 Active Directory user accounts with the following properties for each user:

```
┌─────────────────────────────────────────────────────────────┐
│                  Each User Gets                             │
│                                                             │
│   👤 Display Name        e.g. Ronald Gomez                  │
│   🔑 SAM Account Name    e.g. rgomez  (login username)      │
│   📧 UPN                 e.g. rgomez@lab.local              │
│   📧 Email Address       e.g. rgomez@lab.local              │
│   🏢 Department          e.g. IT                            │
│   💼 Job Title           e.g. Security Analyst              │
│   📁 OU Placement        e.g. OU=IT,OU=_USERS,DC=Lab,DC=local│
│   🔒 Password            Welcome1@Lab (must change on login) │
│   ✅ Account Enabled     Yes — ready to use immediately      │
└─────────────────────────────────────────────────────────────┘
```

### How Usernames Are Generated

Usernames follow the standard enterprise convention of **first initial + last name**:

```
Ronald Gomez    →   rgomez
Betty Jackson   →   bjackson
Paul Campbell   →   pcampbell
```

The script automatically handles **duplicate usernames** by appending a number:

```
John Smith   →   jsmith
Jane Smith   →   jsmith2
Jeff Smith   →   jsmith3
```

---

## 📖 Understanding the Script Components

The script is built from five key sections:

### Section 1 — Configuration Variables
Defines the domain, target OU, default password, and UPN suffix. Change these values to adapt the script for different environments.

### Section 2 — Name Pools
Two arrays of realistic first and last names. The script randomly picks from these to generate unique user combinations.

### Section 3 — Department Definitions
An array of department objects — each containing a department name, target OU, and job title. The script randomly assigns one to each user.

### Section 4 — Duplicate Prevention
Loads all existing SAM account names from AD into a HashSet before creating any users. This prevents conflicts with accounts that already exist.

### Section 5 — User Creation Loop
Runs until 110 users have been created. For each user it builds the username, checks for duplicates, selects a department, and calls `New-ADUser`.

---

## 📁 Create the Scripts Folder

Open **PowerShell as Administrator** on DC01 and run:

```powershell
# Create the scripts directory
New-Item -ItemType Directory -Path "C:\Scripts"

# Verify it was created
Get-Item "C:\Scripts"
```

Expected output:

```
    Directory: C:\

Mode                LastWriteTime    Length  Name
----                -------------    ------  ----
d----          17/04/2026  08:00            Scripts
```

---

## 📝 Create the Script File

Open Notepad with the script file pre-named and ready:

```powershell
notepad C:\Scripts\New-BulkADUsers.ps1
```

When Notepad asks **"Do you want to create a new file?"** click **Yes**.

Now copy and paste the full script from the section below into Notepad, then press **Ctrl+S** to save.

---

## 📜 The Full Script

```powershell
<#
.SYNOPSIS
    Bulk creates 110 Active Directory users for the Lab.local homelab domain.

.DESCRIPTION
    Generates realistic user accounts across IT, HR, Finance, and Management
    departments. All users are placed in the correct sub-OU under _USERS and
    are enabled with a standard default password.

.NOTES
    Domain  : Lab.local
    Run on  : DC01 as LAB\Administrator
    Script  : C:\Scripts\New-BulkADUsers.ps1
#>

#---------------------------------------------------------------------
# SECTION 1 — Configuration
#---------------------------------------------------------------------
$DomainDN        = "DC=Lab,DC=local"
$BaseOU          = "OU=_USERS,$DomainDN"
$DefaultPassword = ConvertTo-SecureString "Welcome1@Lab" -AsPlainText -Force
$UPNSuffix       = "lab.local"
$TotalToCreate   = 110

#---------------------------------------------------------------------
# SECTION 2 — First and Last Name Pools
#---------------------------------------------------------------------
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

#---------------------------------------------------------------------
# SECTION 3 — Department and Job Title Definitions
#---------------------------------------------------------------------
$Departments = @(
    @{ Name="IT";         OU="IT";         Title="Systems Administrator"  },
    @{ Name="IT";         OU="IT";         Title="Help Desk Technician"   },
    @{ Name="IT";         OU="IT";         Title="Network Engineer"        },
    @{ Name="IT";         OU="IT";         Title="Security Analyst"        },
    @{ Name="HR";         OU="HR";         Title="HR Manager"              },
    @{ Name="HR";         OU="HR";         Title="HR Coordinator"          },
    @{ Name="Finance";    OU="Finance";    Title="Financial Analyst"       },
    @{ Name="Finance";    OU="Finance";    Title="Accountant"              },
    @{ Name="Management"; OU="Management"; Title="Department Manager"      },
    @{ Name="Management"; OU="Management"; Title="Team Lead"               }
)

#---------------------------------------------------------------------
# SECTION 4 — Duplicate Prevention
#---------------------------------------------------------------------
Write-Host "Loading existing AD usernames..." -ForegroundColor Yellow

$UsedSamNames = [System.Collections.Generic.HashSet[string]]::new(
    [System.StringComparer]::OrdinalIgnoreCase)

Get-ADUser -Filter * -Properties SamAccountName |
    ForEach-Object { $null = $UsedSamNames.Add($_.SamAccountName) }

Write-Host "Found $($UsedSamNames.Count) existing accounts. Starting user creation...`n" `
    -ForegroundColor Yellow

#---------------------------------------------------------------------
# SECTION 5 — User Creation Loop
#---------------------------------------------------------------------
$UsersCreated = 0
$UsersSkipped = 0
$random       = New-Object System.Random

while ($UsersCreated -lt $TotalToCreate) {

    # Pick random first and last name
    $First = $FirstNames[$random.Next(0, $FirstNames.Count)]
    $Last  = $LastNames[$random.Next(0, $LastNames.Count)]

    # Build SAM account name: first initial + last name, lowercase, max 20 chars
    $BaseSam = ($First.Substring(0,1) + $Last).ToLower() -replace '[^a-z0-9]', ''
    if ($BaseSam.Length -gt 18) { $BaseSam = $BaseSam.Substring(0,18) }

    # Handle duplicates by appending a number
    $SamName = $BaseSam
    $suffix  = 2
    while ($UsedSamNames.Contains($SamName)) {
        $SamName = "$BaseSam$suffix"
        $suffix++
    }
    $null = $UsedSamNames.Add($SamName)

    # Pick a random department
    $Dept     = $Departments[$random.Next(0, $Departments.Count)]
    $TargetOU = "OU=$($Dept.OU),$BaseOU"

    # Build user attributes
    $UPN         = "$SamName@$UPNSuffix"
    $DisplayName = "$First $Last"
    $Description = "$($Dept.Title) - $($Dept.Name) Department"

    # Create the user account
    try {
        New-ADUser `
            -SamAccountName        $SamName `
            -UserPrincipalName     $UPN `
            -Name                  $DisplayName `
            -GivenName             $First `
            -Surname               $Last `
            -DisplayName           $DisplayName `
            -EmailAddress          "$SamName@$UPNSuffix" `
            -Department            $Dept.Name `
            -Title                 $Dept.Title `
            -Description           $Description `
            -Path                  $TargetOU `
            -AccountPassword       $DefaultPassword `
            -Enabled               $true `
            -PasswordNeverExpires   $false `
            -ChangePasswordAtLogon  $true `
            -ErrorAction Stop

        $UsersCreated++
        Write-Host "[$UsersCreated/$TotalToCreate] Created: $DisplayName ($SamName) — $($Dept.Name)" `
            -ForegroundColor Green

    } catch {
        Write-Warning "Failed to create $DisplayName ($SamName): $_"
        $UsersSkipped++
    }
}

#---------------------------------------------------------------------
# Summary
#---------------------------------------------------------------------
Write-Host "`n--- Bulk User Creation Complete ---" -ForegroundColor Yellow
Write-Host "Users created  : $UsersCreated"        -ForegroundColor Green
Write-Host "Errors/skipped : $UsersSkipped"        -ForegroundColor Red
Write-Host "Open ADUC to verify your users!"       -ForegroundColor Cyan
```

---

## ▶️ Run the Script

### Step 1 — Set the Execution Policy

By default PowerShell blocks unsigned scripts. Allow local scripts to run:

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

When prompted type **Y** and press Enter.

> 💡 **What does RemoteSigned mean?**
> - Scripts you create locally can run without a signature ✅
> - Scripts downloaded from the internet must be digitally signed ✅
> - This is the recommended setting for administrators — not too restrictive, not too open

---

### Step 2 — Run the Script

```powershell
& "C:\Scripts\New-BulkADUsers.ps1"
```

You will see green text scrolling as each user is created:

```
Loading existing AD usernames...
Found 1 existing accounts. Starting user creation...

[1/110]   Created: Mary Smith (msmith) — HR
[2/110]   Created: Robert Johnson (rjohnson) — IT
[3/110]   Created: Jennifer Williams (jwilliams) — Finance
[4/110]   Created: Michael Brown (mbrown) — Management
...
[108/110] Created: Betty Jackson (bjackson) — Management
[109/110] Created: Ronald Gomez (rgomez) — IT
[110/110] Created: Paul Campbell (pcampbell) — Finance

--- Bulk User Creation Complete ---
Users created  : 110
Errors/skipped : 0
Open ADUC to verify your users!
```

**Target result: Users created: 110 — Errors/skipped: 0**

---

## 🔍 Verify the Results

### Verify in PowerShell

```powershell
# Total user count in _USERS OU and all sub-OUs
$Total = (Get-ADUser -Filter * `
    -SearchBase "OU=_USERS,DC=Lab,DC=local" `
    -SearchScope Subtree).Count

Write-Host "Total users in _USERS: $Total"
```

```powershell
# Breakdown by department
Get-ADUser -Filter * `
    -SearchBase "OU=_USERS,DC=Lab,DC=local" `
    -SearchScope Subtree `
    -Properties Department |
    Group-Object Department |
    Select-Object Name, Count |
    Sort-Object Count -Descending |
    Format-Table -AutoSize
```

Expected output:

```
Name        Count
----        -----
IT             45
HR             22
Finance        22
Management     21
```

> Numbers will vary slightly since department assignment is random —
> but all four departments should have users and the total should be 110.

```powershell
# View a sample of created users with their details
Get-ADUser -Filter * `
    -SearchBase "OU=IT,OU=_USERS,DC=Lab,DC=local" `
    -Properties DisplayName, Title, Department, EmailAddress |
    Select-Object DisplayName, Title, Department, EmailAddress |
    Sort-Object DisplayName |
    Format-Table -AutoSize
```

---

### Verify in Active Directory Users and Computers

1. Open **Server Manager → Tools → Active Directory Users and Computers**
2. Expand **Lab.local → _USERS**
3. Click each sub-OU — you should see users in all four:

```
_USERS
├── Finance    → 20-25 users
├── HR         → 20-25 users
├── IT         → 40-50 users
└── Management → 18-25 users
```

Each user should show a person icon with their display name listed.

---

### Verify a Specific User's Properties

```powershell
# Look up a specific user — replace rgomez with any username from the script output
Get-ADUser -Identity "rgomez" -Properties * |
    Select-Object DisplayName, SamAccountName, UserPrincipalName,
                  Department, Title, EmailAddress, Enabled,
                  PasswordNeverExpires, ChangePasswordAtLogon
```

Expected output confirms all properties were set correctly:

```
DisplayName          : Ronald Gomez
SamAccountName       : rgomez
UserPrincipalName    : rgomez@lab.local
Department           : IT
Title                : Security Analyst
EmailAddress         : rgomez@lab.local
Enabled              : True
PasswordNeverExpires : False
ChangePasswordAtLogon: True
```

---

## 🛠️ Useful AD User Management Commands

Now that you have 110 users, here are the most useful day-to-day commands
for managing them — skills directly applicable to real-world IT roles:

### Reset a User Password

```powershell
Set-ADAccountPassword `
    -Identity "rgomez" `
    -NewPassword (ConvertTo-SecureString "NewP@ss2024!" -AsPlainText -Force) `
    -Reset
```

### Unlock a Locked Account

```powershell
Unlock-ADAccount -Identity "rgomez"
```

### Disable a User Account

```powershell
Disable-ADAccount -Identity "rgomez"
```

### Enable a User Account

```powershell
Enable-ADAccount -Identity "rgomez"
```

### Find All Locked Out Accounts

```powershell
Search-ADAccount -LockedOut | Select-Object Name, SamAccountName, LockedOut
```

### Find All Disabled Accounts

```powershell
Search-ADAccount -AccountDisabled | Select-Object Name, SamAccountName
```

### Move a User to a Different OU

```powershell
# Move Ronald Gomez from IT to Management
Get-ADUser -Identity "rgomez" |
    Move-ADObject -TargetPath "OU=Management,OU=_USERS,DC=Lab,DC=local"
```

### Find Users Who Have Never Logged In

```powershell
Get-ADUser -Filter * `
    -Properties LastLogonDate, Department |
    Where-Object { $_.LastLogonDate -eq $null } |
    Select-Object DisplayName, SamAccountName, Department |
    Format-Table -AutoSize
```

### Export All Users to CSV

```powershell
Get-ADUser -Filter * `
    -SearchBase "OU=_USERS,DC=Lab,DC=local" `
    -SearchScope Subtree `
    -Properties DisplayName, Department, Title, EmailAddress, Enabled |
    Select-Object DisplayName, SamAccountName, Department, Title, EmailAddress, Enabled |
    Export-Csv -Path "C:\Scripts\AllUsers.csv" -NoTypeInformation

Write-Host "Export saved to C:\Scripts\AllUsers.csv" -ForegroundColor Green
```

---

## 📸 Take a Snapshot

110 users created successfully — this is a major milestone.

In VirtualBox go to:
```
Machine → Take Snapshot
```

Name it:
```
DC01 - 110 AD Users Created - Pre Networking
```

Click **OK**.

> This snapshot is extremely valuable. If anything goes wrong with
> Group Policy or domain joining in later steps, you can roll back
> to this point and have a fully populated Active Directory
> with all 110 users intact — ready to start again from here.

---

## 🛠️ Troubleshooting

### Issue: "Set-ExecutionPolicy" is blocked by Group Policy

A Group Policy is preventing execution policy changes.

**Fix:**
```powershell
# Run the script bypassing the execution policy for this session only
PowerShell.exe -ExecutionPolicy Bypass -File "C:\Scripts\New-BulkADUsers.ps1"
```

---

### Issue: Script fails with "Access is denied" on New-ADUser

You are not running PowerShell as Administrator, or not logged in as a Domain Admin.

**Fix:**
1. Close PowerShell
2. Right-click the **Windows PowerShell** icon → **Run as Administrator**
3. Confirm you are logged in as `LAB\Administrator`
4. Re-run the script

---

### Issue: Script runs but shows many "Failed to create" warnings

The target OU path may be incorrect — the OU does not exist yet.

**Fix:**
```powershell
# Verify all OUs exist before running the script
Get-ADOrganizationalUnit -Filter * |
    Where-Object { $_.DistinguishedName -like "*_USERS*" } |
    Select-Object DistinguishedName
```

If any sub-OU is missing, go back to **[08 - Create OU Structure](../08-Create-OU-Structure/README.md)** and create it.

---

### Issue: Script creates fewer than 110 users and stops

The random name combinations may have been exhausted (unlikely with large name pools but possible).

**Fix:**
Add more names to either the `$FirstNames` or `$LastNames` arrays and re-run the script. The script will skip any usernames already taken and continue until it reaches the target.

---

### Issue: Users created but all landed in the wrong OU

The `$BaseOU` variable in Section 1 may have a typo.

**Fix:**
```powershell
# Check where users actually landed
Get-ADUser -Filter * -Properties DistinguishedName |
    Where-Object { $_.DistinguishedName -notlike "*_USERS*" -and
                   $_.SamAccountName -ne "Administrator" } |
    Select-Object DisplayName, DistinguishedName

# Move them to the correct OU
Get-ADUser -Filter * -SearchBase "CN=Users,DC=Lab,DC=local" |
    Move-ADObject -TargetPath "OU=IT,OU=_USERS,DC=Lab,DC=local"
```

---

### Issue: "ChangePasswordAtLogon" conflict error

A conflict exists between `PasswordNeverExpires` and `ChangePasswordAtLogon`.

**Fix:**
Ensure both settings in the script are:
```powershell
-PasswordNeverExpires   $false  # Must be false
-ChangePasswordAtLogon  $true   # Can only be true if above is false
```

These two settings cannot both be true simultaneously.

---

## ✅ Verification Checklist

```
Script Setup
[ ] C:\Scripts folder created on DC01
[ ] New-BulkADUsers.ps1 saved to C:\Scripts\
[ ] Execution policy set to RemoteSigned

Script Execution
[ ] Script ran without red error messages
[ ] Output shows [110/110] Created
[ ] Final summary shows "Users created: 110"
[ ] Final summary shows "Errors/skipped: 0"

PowerShell Verification
[ ] Total count in _USERS OU returns 110
[ ] Department breakdown shows all 4 departments have users
[ ] Sample user lookup shows correct properties
[ ] Enabled: True on all users
[ ] ChangePasswordAtLogon: True on all users

ADUC Verification
[ ] _USERS OU expanded and shows 4 sub-OUs
[ ] IT sub-OU contains users
[ ] HR sub-OU contains users
[ ] Finance sub-OU contains users
[ ] Management sub-OU contains users

Snapshot
[ ] Snapshot taken named "DC01 - 110 AD Users Created - Pre Networking"

Ready to proceed
[ ] 110 domain users exist and are verified
[ ] Users are correctly distributed across all 4 department OUs
[ ] Ready to move on to networking and WS01 setup
```

---

## ➡️ Next Step

The domain is populated with 110 users. Now we configure the
lab network so DC01 and WS01 can communicate with each other.

**[10 - Configure Lab Networking →](../10%20-%20Configure%20Networking/README.md)**

---

## ⬅️ Previous Step

**[← 08 - Create OU Structure](../08%20-%20Create%20OU%20Structure/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
