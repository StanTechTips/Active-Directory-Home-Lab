# 08 - Create Organisational Unit Structure

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Beginner-blue?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/VM-DC01-purple?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Tool-ADUC-0078D6?style=for-the-badge&logo=windows)

> Organisational Units are the folders of Active Directory.
> A well-designed OU structure is the foundation of clean user management,
> targeted Group Policy, and efficient administration.
> This step builds the structure before any users or computers are added.

---

## 📑 Table of Contents

- [What is an Organisational Unit?](#-what-is-an-organisational-unit)
- [Why OU Design Matters](#-why-ou-design-matters)
- [The OU Structure for This Lab](#-the-ou-structure-for-this-lab)
- [Method 1 — Create OUs via ADUC (GUI)](#-method-1--create-ous-via-aduc-gui)
- [Method 2 — Create OUs via PowerShell (Faster)](#-method-2--create-ous-via-powershell-faster)
- [Verify the Structure](#-verify-the-structure)
- [Understanding OU vs Default Containers](#-understanding-ou-vs-default-containers)
- [Take a Snapshot](#-take-a-snapshot)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)

---

## 🧠 What is an Organisational Unit?

An **Organisational Unit (OU)** is a container object inside Active Directory used to organise users, computers, groups, and other OUs into a logical hierarchy.

```
┌─────────────────────────────────────────────────────────────┐
│              What OUs Do in Active Directory                 │
│                                                             │
│   📁 Organisation                                           │
│      Group objects logically — by department, location,     │
│      or function — so the directory is easy to navigate     │
│                                                             │
│   🎯 Group Policy Targeting                                 │
│      GPOs are linked to OUs — only objects inside the OU    │
│      receive that policy. Link a GPO to _COMPUTERS and      │
│      only domain computers get it, not users.               │
│                                                             │
│   🔐 Delegated Administration                               │
│      Grant a helpdesk team permission to reset passwords    │
│      only in the HR OU — without giving them domain admin   │
│                                                             │
│   🔍 Easier Management                                      │
│      Find all IT users instantly. Move a user between       │
│      departments by dragging them to a different OU.        │
└─────────────────────────────────────────────────────────────┘
```

Think of OUs as **folders in a filing cabinet** — the filing cabinet is your domain, and everything inside it needs a logical place to live.

---

## 💡 Why OU Design Matters

Poor OU design is one of the most common mistakes in real-world Active Directory deployments. Here is why it matters:

| Scenario | Bad OU Design | Good OU Design |
|----------|--------------|----------------|
| Apply USB policy to all computers | Can't target easily — have to apply to entire domain | Link GPO to _COMPUTERS OU — applies only to computers |
| Reset passwords for HR users only | Have to search entire directory | All HR users are in _USERS\HR — find them instantly |
| New starter joins Finance team | Dump in default Users container | Add directly to _USERS\Finance — correct policies apply immediately |
| Helpdesk needs to manage IT accounts only | Give helpdesk domain admin — too much access | Delegate control to _USERS\IT OU only |

> 💡 **Rule of thumb:** Design your OU structure around how you manage objects,
> not around how your organisation chart looks.
> OUs that mirror GPO requirements are more valuable than OUs that mirror the org chart.

---

## 🗺️ The OU Structure for This Lab

Here is the complete OU structure we are building:

```
Lab.local
├── 📁 _COMPUTERS
│     └── (Domain-joined workstations go here)
│
├── 📁 _GROUPS
│     └── (Security and distribution groups go here)
│
└── 📁 _USERS
      ├── 📁 IT
      │     └── (IT department users — 
      │           Systems Admins, Help Desk, Network Engineers)
      │
      ├── 📁 HR
      │     └── (HR department users —
      │           HR Managers, HR Coordinators)
      │
      ├── 📁 Finance
      │     └── (Finance department users —
      │           Financial Analysts, Accountants)
      │
      └── 📁 Management
            └── (Management users —
                  Department Managers, Team Leads)
```

### Why the Underscore Prefix?

The three top-level OUs are named `_COMPUTERS`, `_GROUPS`, and `_USERS` — with a leading underscore. This is a widely used convention in enterprise environments for one simple reason:

```
Without underscore:          With underscore:
┌─────────────────────┐      ┌─────────────────────┐
│ Builtin             │      │ _COMPUTERS          │  ← Sorts to top
│ Computers           │      │ _GROUPS             │  ← Sorts to top
│ Domain Controllers  │      │ _USERS              │  ← Sorts to top
│ Finance             │      │ Builtin             │
│ ForeignSecurity...  │      │ Computers           │
│ HR                  │      │ Domain Controllers  │
│ IT                  │      │ ForeignSecurity...  │
│ Management          │      │ Users               │
│ Users               │      │                     │
└─────────────────────┘      └─────────────────────┘
         Messy                      Clean ✅
```

The underscore sorts your custom OUs to the very top of the alphabetical list, above all the default AD containers — making them immediately visible and easy to navigate.

---

## 🖥️ Method 1 — Create OUs via ADUC (GUI)

### Open Active Directory Users and Computers

In Server Manager click **Tools → Active Directory Users and Computers**.

The ADUC console opens. In the left panel you will see your domain `Lab.local` with the default containers below it.

---

### Create _COMPUTERS

**1.** Right-click **Lab.local** in the left panel.

**2.** Hover over **"New"** → click **"Organizational Unit"**.

**3.** In the Name field type:
```
_COMPUTERS
```

**4.** Confirm **"Protect container from accidental deletion"** is ticked ✅

**5.** Click **OK**.

`_COMPUTERS` now appears in the left panel under Lab.local.

---

### Create _GROUPS

**1.** Right-click **Lab.local** → **New** → **Organizational Unit**.

**2.** Type:
```
_GROUPS
```

**3.** Click **OK**.

---

### Create _USERS

**1.** Right-click **Lab.local** → **New** → **Organizational Unit**.

**2.** Type:
```
_USERS
```

**3.** Click **OK**.

---

### Create Sub-OUs inside _USERS

Now create the four department OUs **inside** `_USERS`:

**IT Sub-OU:**

**1.** Right-click **_USERS** in the left panel → **New** → **Organizational Unit**.

**2.** Type:
```
IT
```
Click **OK**.

---

**HR Sub-OU:**

**1.** Right-click **_USERS** → **New** → **Organizational Unit**.

**2.** Type:
```
HR
```
Click **OK**.

---

**Finance Sub-OU:**

**1.** Right-click **_USERS** → **New** → **Organizational Unit**.

**2.** Type:
```
Finance
```
Click **OK**.

---

**Management Sub-OU:**

**1.** Right-click **_USERS** → **New** → **Organizational Unit**.

**2.** Type:
```
Management
```
Click **OK**.

---

### Confirm the Structure in ADUC

Click the arrow next to **_USERS** to expand it.
You should now see all four department sub-OUs:

```
Lab.local
├── _COMPUTERS
├── _GROUPS
└── _USERS
    ├── Finance
    ├── HR
    ├── IT
    └── Management
```

---

## ⚡ Method 2 — Create OUs via PowerShell (Faster)

If you prefer the command line, this single PowerShell script creates the entire structure in seconds.

Open **PowerShell as Administrator** on DC01 and run:

```powershell
# Define the domain Distinguished Name
$DomainDN = "DC=Lab,DC=local"

# Create the three top-level OUs
$TopLevelOUs = @("_COMPUTERS", "_GROUPS", "_USERS")

foreach ($OU in $TopLevelOUs) {
    New-ADOrganizationalUnit `
        -Name $OU `
        -Path $DomainDN `
        -ProtectedFromAccidentalDeletion $true
    Write-Host "Created OU: $OU" -ForegroundColor Green
}

# Create the four department sub-OUs inside _USERS
$DepartmentOUs = @("IT", "HR", "Finance", "Management")

foreach ($Dept in $DepartmentOUs) {
    New-ADOrganizationalUnit `
        -Name $Dept `
        -Path "OU=_USERS,$DomainDN" `
        -ProtectedFromAccidentalDeletion $true
    Write-Host "Created sub-OU: $Dept under _USERS" -ForegroundColor Cyan
}

Write-Host "`nOU structure created successfully!" -ForegroundColor Yellow
```

Expected output:

```
Created OU: _COMPUTERS
Created OU: _GROUPS
Created OU: _USERS
Created sub-OU: IT under _USERS
Created sub-OU: HR under _USERS
Created sub-OU: Finance under _USERS
Created sub-OU: Management under _USERS

OU structure created successfully!
```

---

## 🔍 Verify the Structure

After creating the OUs via either method, verify everything is correct in PowerShell:

```powershell
# List all OUs in the domain
Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName | Sort-Object DistinguishedName
```

Expected output:

```
Name          DistinguishedName
----          -----------------
_COMPUTERS    OU=_COMPUTERS,DC=Lab,DC=local
_GROUPS       OU=_GROUPS,DC=Lab,DC=local
_USERS        OU=_USERS,DC=Lab,DC=local
Finance       OU=Finance,OU=_USERS,DC=Lab,DC=local
HR            OU=HR,OU=_USERS,DC=Lab,DC=local
IT            OU=IT,OU=_USERS,DC=Lab,DC=local
Management    OU=Management,OU=_USERS,DC=Lab,DC=local
Domain Controllers  OU=Domain Controllers,DC=Lab,DC=local
```

> 💡 **Understanding Distinguished Names (DNs):**
> The `DistinguishedName` column shows the full path of each OU in LDAP notation.
> Reading right to left — `OU=IT,OU=_USERS,DC=Lab,DC=local` means:
> the IT OU, inside the _USERS OU, inside the Lab.local domain.
> You will use these paths in PowerShell scripts throughout the rest of the lab.

---

## 📖 Understanding OU vs Default Containers

Active Directory has two types of containers that look similar but behave differently:

| Feature | Organisational Unit (OU) | Default Container |
|---------|--------------------------|------------------|
| Can link GPOs to it | ✅ Yes | ❌ No |
| Can delegate control | ✅ Yes | ⚠️ Limited |
| Can create sub-containers | ✅ Yes | ❌ No |
| Accidental deletion protection | ✅ Optional | N/A |
| Examples | _USERS, _COMPUTERS, IT, HR | Computers, Users, Builtin |
| Icon in ADUC | 📁 Folder with book | 📦 Plain folder |

> ⚠️ **Important:** The default **Computers** container (not your `_COMPUTERS` OU)
> cannot have GPOs linked to it. This is why we created `_COMPUTERS` as a proper OU.
> When WS01 joins the domain it will initially land in the default Computers container —
> we will move it to `_COMPUTERS` in Step 13 so GPOs can apply to it correctly.

---

## 🔒 Accidental Deletion Protection Explained

When you create an OU with **"Protect container from accidental deletion"** ticked, Active Directory adds a special access control entry that prevents the OU from being deleted via the ADUC console without first disabling the protection.

To delete a protected OU you must first:

**In ADUC:**
1. Click **View → Advanced Features** (to show the Security tab)
2. Right-click the OU → **Properties → Object tab**
3. Untick **"Protect object from accidental deletion"**
4. Then you can delete it

**In PowerShell:**
```powershell
# Remove protection first
Set-ADOrganizationalUnit -Identity "OU=_USERS,DC=Lab,DC=local" -ProtectedFromAccidentalDeletion $false

# Then delete
Remove-ADOrganizationalUnit -Identity "OU=_USERS,DC=Lab,DC=local" -Confirm:$false
```

> 💡 This protection has saved many administrators from accidentally deleting
> an OU containing thousands of user accounts. Always leave it enabled.

---

## 📸 Take a Snapshot

The OU structure is built and ready for users. Take a snapshot before the bulk user creation in the next step.

In VirtualBox go to:
```
Machine → Take Snapshot
```

Name it:
```
DC01 - OU Structure Created - Pre User Import
```

Click **OK**.

---

## 🛠️ Troubleshooting

### Issue: Cannot right-click Lab.local to create a new OU

You may not have the correct permissions or the view is collapsed.

**Fix:**
1. Make sure you are logged in as `LAB\Administrator`
2. In ADUC click the **arrow** next to Lab.local to expand it first
3. Then right-click directly on `Lab.local` — not on a container inside it
4. If the option is still missing click **View → Advanced Features** and try again

---

### Issue: PowerShell returns "New-ADOrganizationalUnit is not recognised"

The Active Directory module is not loaded.

**Fix:**
```powershell
Import-Module ActiveDirectory
# Then re-run the script
```

---

### Issue: "An object with this name already exists" error when running the PowerShell script

An OU with the same name was already created (possibly partially).

**Fix:**
```powershell
# Check what already exists
Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName

# Remove a specific OU if needed (disable protection first)
Set-ADOrganizationalUnit -Identity "OU=IT,OU=_USERS,DC=Lab,DC=local" -ProtectedFromAccidentalDeletion $false
Remove-ADOrganizationalUnit -Identity "OU=IT,OU=_USERS,DC=Lab,DC=local" -Confirm:$false

# Then recreate it
New-ADOrganizationalUnit -Name "IT" -Path "OU=_USERS,DC=Lab,DC=local" -ProtectedFromAccidentalDeletion $true
```

---

### Issue: Sub-OUs were created at the wrong level (under Lab.local instead of under _USERS)

The wrong container was right-clicked when creating the sub-OUs.

**Fix:**
```powershell
# Move the misplaced OU to the correct location
Move-ADObject `
    -Identity "OU=IT,DC=Lab,DC=local" `
    -TargetPath "OU=_USERS,DC=Lab,DC=local"
```

---

### Issue: OUs are not visible in ADUC after creation

ADUC may need to be refreshed.

**Fix:**
Press **F5** to refresh the ADUC view, or right-click the domain → **Refresh**.

---

## ✅ Verification Checklist

```
Top-Level OUs Created
[ ] _COMPUTERS OU created directly under Lab.local
[ ] _GROUPS OU created directly under Lab.local
[ ] _USERS OU created directly under Lab.local
[ ] All three have underscore prefix and sort to the top

Department Sub-OUs Created (all inside _USERS)
[ ] IT sub-OU created inside _USERS
[ ] HR sub-OU created inside _USERS
[ ] Finance sub-OU created inside _USERS
[ ] Management sub-OU created inside _USERS

PowerShell Verification
[ ] Get-ADOrganizationalUnit shows all 7 OUs
[ ] Distinguished Names confirm correct nesting
[ ] Finance, HR, IT, Management all show OU=_USERS in their path

ADUC Visual Confirmation
[ ] _COMPUTERS visible directly under Lab.local
[ ] _GROUPS visible directly under Lab.local
[ ] _USERS visible directly under Lab.local
[ ] Expanding _USERS shows all four department OUs

Snapshot
[ ] Snapshot taken named "DC01 - OU Structure Created - Pre User Import"

Ready to proceed
[ ] All 7 OUs created and verified
[ ] Structure matches the design in this guide
[ ] Ready to receive bulk user accounts in the next step
```

---

## ➡️ Next Step

The OU structure is ready. Time to populate it with 110 domain users
using a PowerShell automation script.

**[09 - Bulk Create Users with PowerShell →](../09-Bulk-Create-Users/README.md)**

---

## ⬅️ Previous Step

**[← 07 - Promote to Domain Controller](../07-Promote-to-Domain-Controller/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
