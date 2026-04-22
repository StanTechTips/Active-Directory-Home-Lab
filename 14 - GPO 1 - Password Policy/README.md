# 14 - GPO 1 — Password Policy

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Intermediate-orange?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/VM-DC01-purple?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Tool-Group_Policy-red?style=for-the-badge&logo=windows)

> A strong password policy is the foundation of every secure Active Directory
> environment. This GPO enforces password complexity, length, history,
> and expiry rules across the entire Lab.local domain — applying to all
> 110 users and any future accounts automatically.
> Every setting here mirrors what you will find in real enterprise environments.

---

## 📑 Table of Contents

- [What is a Group Policy Object?](#-what-is-a-group-policy-object)
- [Why Password Policy Belongs in Default Domain Policy](#-why-password-policy-belongs-in-default-domain-policy)
- [Password Policy Settings Explained](#-password-policy-settings-explained)
- [Open Group Policy Management](#-open-group-policy-management)
- [Configure the Password Policy](#-configure-the-password-policy)
- [Apply and Verify the Policy](#-apply-and-verify-the-policy)
- [Test the Policy is Working](#-test-the-policy-is-working)
- [Understanding GPO Processing](#-understanding-gpo-processing)
- [Take a Snapshot](#-take-a-snapshot)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)

---

## 🧠 What is a Group Policy Object?

A **Group Policy Object (GPO)** is a collection of settings that Windows
applies automatically to users and computers in Active Directory.
GPOs are the primary tool IT administrators use to enforce security
standards and configuration across an entire organisation.

```
┌─────────────────────────────────────────────────────────────┐
│              How Group Policy Works                          │
│                                                             │
│  DC01 stores GPOs          WS01 applies GPOs               │
│  ┌───────────────────┐     ┌───────────────────┐           │
│  │  Group Policy     │     │  gpupdate /force  │           │
│  │  Management       │────►│  or at login      │           │
│  │  Console (GPMC)   │     │  or every 90 mins │           │
│  └───────────────────┘     └───────────────────┘           │
│                                                             │
│  Stored in SYSVOL share    Applied to:                      │
│  \\DC01\SYSVOL\...         ✅ Computer accounts             │
│                            ✅ User accounts                 │
│                            ✅ Specific OUs                  │
└─────────────────────────────────────────────────────────────┘
```

**The two GPO configuration areas:**

| Configuration Area | Applies to | Applied when |
|-------------------|-----------|-------------|
| Computer Configuration | Computer objects | Computer starts up |
| User Configuration | User objects | User logs in |

---

## 💡 Why Password Policy Belongs in Default Domain Policy

Password policy in Active Directory is special — it can only be enforced
at the **domain level**, not at the OU level. This is a core AD design rule.

```
┌─────────────────────────────────────────────────────────────┐
│              GPO Linking and Password Policy                 │
│                                                             │
│   ✅ Password policy linked to domain root                  │
│      Applies to ALL users in the entire domain              │
│                                                             │
│   ❌ Password policy linked to an OU (e.g. _USERS)         │
│      Will NOT enforce — AD ignores password settings        │
│      at the OU level for domain accounts                    │
│                                                             │
│   ✅ Fine-Grained Password Policies (FGPPs)                 │
│      Advanced feature — allows different policies           │
│      per security group (e.g. admins need 16+ chars)        │
│      Not covered in this lab — standard domain policy used  │
└─────────────────────────────────────────────────────────────┘
```

> 💡 **The correct approach:**
> Configure password policy directly inside the **Default Domain Policy**
> which is already linked at the domain root and takes effect for all accounts.
> This is the standard practice in enterprise environments.

---

## 🔐 Password Policy Settings Explained

Here are the six settings we configure and what each one does in practice:

### Enforce Password History

| Setting | Value | What It Does |
|---------|-------|-------------|
| Enforce password history | 5 passwords | Prevents users reusing their last 5 passwords |

**Real world example:** Without this, users change their password then
immediately change it back to their old one. History enforcement stops this.

---

### Maximum Password Age

| Setting | Value | What It Does |
|---------|-------|-------------|
| Maximum password age | 90 days | Forces a password change every 90 days |

**Real world example:** If credentials are stolen, regular expiry limits
the window an attacker can use compromised credentials undetected.

---

### Minimum Password Age

| Setting | Value | What It Does |
|---------|-------|-------------|
| Minimum password age | 1 day | Prevents changing password multiple times in one day |

**Real world example:** Without this, a user could change their password
5 times in a row to cycle through the history and get back to their original password.
A 1-day minimum stops this bypass.

---

### Minimum Password Length

| Setting | Value | What It Does |
|---------|-------|-------------|
| Minimum password length | 10 characters | Rejects passwords shorter than 10 characters |

**Real world note:** NIST guidelines now recommend 12–16 characters as
the minimum for enterprise accounts. 10 is a solid baseline for a homelab.

---

### Password Complexity Requirements

| Setting | Value | What It Does |
|---------|-------|-------------|
| Complexity requirements | Enabled | Password must contain 3 of 4 character types |

**The four character types:**
- Uppercase letters (A–Z)
- Lowercase letters (a–z)
- Numbers (0–9)
- Special characters (!@#$%^&*)

**Example:** `lab123` fails complexity. `Lab@2024!` passes. ✅

---

### Reversible Encryption

| Setting | Value | What It Does |
|---------|-------|-------------|
| Reversible encryption | Disabled | Passwords stored as one-way hash — not recoverable |

**Why always disabled:** Reversible encryption stores passwords in a
recoverable format — essentially plain text. This is a serious security
vulnerability and should **never** be enabled unless a specific legacy
application absolutely requires it.

---

## 🚀 Open Group Policy Management

On **DC01**, open the Group Policy Management Console:

**Method 1 — Via Server Manager:**
```
Server Manager → Tools → Group Policy Management
```

**Method 2 — Via Run dialog:**
Press **Windows Key + R**, type `gpmc.msc` and press **Enter**.

The **Group Policy Management** console opens.

In the left panel expand the tree:

```
Group Policy Management
└── Forest: Lab.local
    └── Domains
        └── Lab.local
            ├── Default Domain Policy   ← This is what we edit
            ├── Domain Controllers
            └── Group Policy Objects
```

---

## ⚙️ Configure the Password Policy

### Step 1 — Open Default Domain Policy for Editing

Right-click **"Default Domain Policy"** → click **"Edit"**.

The **Group Policy Management Editor** opens in a new window.

---

### Step 2 — Navigate to Password Policy

In the left panel navigate through the following folders:

```
Default Domain Policy [DC01.Lab.local]
└── Computer Configuration
    └── Policies
        └── Windows Settings
            └── Security Settings
                └── Account Policies
                    └── Password Policy    ← Click here
```

Click on **"Password Policy"**.

Six settings appear in the right panel.

---

### Step 3 — Configure Each Setting

Double-click each setting and configure as follows:

---

**Setting 1 — Enforce password history**

Double-click → tick **"Define this policy setting"** → set value to:
```
5
```
Click **OK**.

---

**Setting 2 — Maximum password age**

Double-click → tick **"Define this policy setting"** → set value to:
```
90
```
Click **OK**.

---

**Setting 3 — Minimum password age**

Double-click → tick **"Define this policy setting"** → set value to:
```
1
```
Click **OK**.

---

**Setting 4 — Minimum password length**

Double-click → tick **"Define this policy setting"** → set value to:
```
10
```
Click **OK**.

---

**Setting 5 — Password must meet complexity requirements**

Double-click → tick **"Define this policy setting"** → select:
```
✅ Enabled
```
Click **OK**.

---

**Setting 6 — Store passwords using reversible encryption**

Double-click → tick **"Define this policy setting"** → select:
```
❌ Disabled
```
Click **OK**.

---

### Step 4 — Confirm All Six Settings

After configuring all six, the right panel should show:

```
Policy                                         Policy Setting
------                                         --------------
Enforce password history                       5 passwords remembered
Maximum password age                           90 days
Minimum password age                           1 days
Minimum password length                        10 characters
Password must meet complexity requirements     Enabled
Store passwords using reversible encryption    Disabled
```

---

### Step 5 — Close the Editor

Close the **Group Policy Management Editor** window.

Settings are saved automatically — no save button needed.

---

## ✅ Apply and Verify the Policy

### Force the Policy to Apply Immediately

On **DC01** open PowerShell and run:

```powershell
gpupdate /force
```

Expected output:

```
Updating policy...

Computer Policy update has completed successfully.
User Policy update has completed successfully.
```

---

### Verify the Password Policy is Active

```powershell
Get-ADDefaultDomainPasswordPolicy
```

Expected output — confirm all six values match:

```
ComplexityEnabled           : True          ✅
DistinguishedName           : DC=Lab,DC=local
LockoutDuration             : 00:00:00
LockoutObservationWindow    : 00:00:00
LockoutThreshold            : 0
MaxPasswordAge              : 90.00:00:00   ✅
MinPasswordAge              : 1.00:00:00    ✅
MinPasswordLength           : 10            ✅
PasswordHistoryCount        : 5             ✅
ReversibleEncryptionEnabled : False         ✅
```

> ⚠️ **If the values are still showing the old defaults** (e.g. MinPasswordLength: 7):
> The Default Domain Policy may have a lower priority than another GPO.
> See the Troubleshooting section below.

---

## 🧪 Test the Policy is Working

### Test 1 — Try Setting a Weak Password (Should Fail)

On **DC01** try to set a weak password on a test user:

```powershell
# This should FAIL — password is too short
Set-ADAccountPassword `
    -Identity "rgomez" `
    -NewPassword (ConvertTo-SecureString "pass1" -AsPlainText -Force) `
    -Reset
```

Expected result — error message:

```
Set-ADAccountPassword : The password does not meet the length, complexity,
or history requirement of the domain.
```

✅ The policy is blocking weak passwords correctly.

---

### Test 2 — Try Setting a Strong Password (Should Succeed)

```powershell
# This should SUCCEED — meets all requirements
Set-ADAccountPassword `
    -Identity "rgomez" `
    -NewPassword (ConvertTo-SecureString "Secure@Pass2024!" -AsPlainText -Force) `
    -Reset
```

Expected result — no error, command completes silently. ✅

---

### Test 3 — Verify on WS01

On **WS01** run:

```powershell
gpupdate /force
```

Then check the applied policy:

```powershell
gpresult /r
```

Look under **"Computer Settings → Applied Group Policy Objects"** for:

```
Default Domain Policy   ✅
```

---

## 📖 Understanding GPO Processing

### How Often GPOs Apply

| Event | GPO Refresh |
|-------|------------|
| Computer startup | Computer Configuration GPOs apply |
| User login | User Configuration GPOs apply |
| Every 90 minutes | Background refresh on domain members |
| `gpupdate /force` | Immediate forced refresh |
| Domain Controller | Every 5 minutes (more frequent than members) |

---

### GPO Processing Order (LSDOU)

When multiple GPOs exist, they are applied in this order:

```
L — Local Group Policy        (applied first — lowest priority)
S — Site GPOs                 (applied second)
D — Domain GPOs               (applied third)
O — Organisational Unit GPOs  (applied last — highest priority)
```

> 💡 **Last applied wins.**
> If a setting is configured in both the Domain GPO and an OU GPO,
> the OU GPO setting wins because it is applied last.
> This is why password policy must be at domain level —
> it needs to be the authoritative setting for all accounts.

---

### GPO Inheritance

```
Lab.local (domain)
├── Default Domain Policy → applies to EVERYTHING below
│
├── _COMPUTERS OU
│   └── Disable USB Storage GPO → applies to computers only
│
└── _USERS OU
    └── Desktop Wallpaper GPO → applies to users only
        ├── IT sub-OU     → inherits Desktop Wallpaper
        ├── HR sub-OU     → inherits Desktop Wallpaper
        ├── Finance sub-OU → inherits Desktop Wallpaper
        └── Management sub-OU → inherits Desktop Wallpaper
```

---

## 📸 Take a Snapshot

Password Policy is configured and verified.

In VirtualBox go to:
```
Machine → Take Snapshot
```

Name it:
```
DC01 - GPO1 Password Policy Configured
```

Click **OK**.

---

## 🛠️ Troubleshooting

### Issue: Get-ADDefaultDomainPasswordPolicy shows old values after gpupdate

The Default Domain Policy changes have not propagated yet, or another GPO
is overriding specific settings.

**Fix — Check GPO precedence:**

In Group Policy Management, click **Lab.local** in the left panel.
Click the **"Linked Group Policy Objects"** tab.

Look at the **Link Order** column. Lower numbers = higher priority.

If **Default Domain Policy** does not have Link Order 1, right-click it
→ **Move Link to Top**.

Then run:

```powershell
gpupdate /force
Get-ADDefaultDomainPasswordPolicy
```

---

### Issue: "Access denied" when opening Default Domain Policy for editing

You are not logged in as a Domain Administrator.

**Fix:**

```powershell
# Verify current logged-in account
whoami

# Must show LAB\Administrator — if not, switch user in Server Manager
```

---

### Issue: Password policy settings showing "Not Defined" after saving

The settings were closed without confirming — they were not saved.

**Fix:**
1. Right-click **Default Domain Policy** → **Edit**
2. Navigate back to Password Policy
3. For each setting — double-click → tick **"Define this policy setting"**
4. Set the value → Click **OK**
5. Confirm all six show values (not "Not Defined") in the right panel
6. Close the editor

---

### Issue: Complexity requirements enabled but short passwords still accepted

The change has not propagated to all domain controllers yet (only relevant
if you have multiple DCs — in this lab with one DC this should not occur).

**Fix:**

```powershell
# Force replication on DC01
repadmin /syncall /AdeP

# Then force policy update
gpupdate /force

# Verify
Get-ADDefaultDomainPasswordPolicy | Select-Object MinPasswordLength, ComplexityEnabled
```

---

### Issue: Test user password change succeeds with a weak password

The wrong domain policy or a local policy may be taking effect.

**Fix:**

```powershell
# Check what password policy the specific user is subject to
Get-ADUserResultantPasswordPolicy -Identity "rgomez"
```

If this returns empty it means the user is subject to the Default Domain
Password Policy — which should now enforce the correct settings.
Run `gpupdate /force` again and retry the test.

---

## ✅ Verification Checklist

```
Group Policy Management Console
[ ] Group Policy Management opened successfully
[ ] Default Domain Policy located under Lab.local
[ ] Default Domain Policy editor opened

Password Policy Configuration
[ ] Enforce password history: 5 passwords
[ ] Maximum password age: 90 days
[ ] Minimum password age: 1 day
[ ] Minimum password length: 10 characters
[ ] Password complexity requirements: Enabled
[ ] Reversible encryption: Disabled
[ ] All six settings show defined values — not "Not Defined"

Policy Applied
[ ] gpupdate /force ran successfully on DC01
[ ] Get-ADDefaultDomainPasswordPolicy confirms all six values
[ ] ComplexityEnabled: True
[ ] MaxPasswordAge: 90 days
[ ] MinPasswordLength: 10
[ ] PasswordHistoryCount: 5
[ ] ReversibleEncryptionEnabled: False

Policy Tested
[ ] Weak password (e.g. "pass1") rejected with policy error
[ ] Strong password (e.g. "Secure@Pass2024!") accepted successfully
[ ] gpresult /r on WS01 shows Default Domain Policy applied

Snapshot
[ ] Snapshot taken named "DC01 - GPO1 Password Policy Configured"

Ready to proceed
[ ] Password policy is live and enforced across Lab.local
[ ] All 110 domain users are subject to this policy
[ ] Ready to configure Account Lockout Policy
```

---

## ➡️ Next Step

Password Policy is enforced across the domain.
Now we configure Account Lockout to protect against
brute force password attacks.

**[15 - GPO 2 Account Lockout Policy →](../15%20-%20GPO%202%20-%20Account%20Lockout/README.md)**

---

## ⬅️ Previous Step

**[← 13 - Join WS01 to the Domain](../13%20-%20Join%20WS01%20to%20the%20Domain/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
