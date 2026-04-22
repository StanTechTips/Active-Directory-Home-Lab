# 15 - GPO 2 — Account Lockout Policy

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Intermediate-orange?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/VM-DC01-purple?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Tool-Group_Policy-red?style=for-the-badge&logo=windows)

> Account lockout is one of the most critical security controls in any
> Active Directory environment. Without it, an attacker can make unlimited
> password guesses against any domain account — silently and indefinitely.
> This GPO locks any account after 5 failed attempts, buying time to
> detect and respond to brute force attacks before they succeed.

---

## 📑 Table of Contents

- [Why Account Lockout Matters](#-why-account-lockout-matters)
- [The Three Lockout Settings Explained](#-the-three-lockout-settings-explained)
- [Configure the Account Lockout Policy](#-configure-the-account-lockout-policy)
- [Apply and Verify the Policy](#-apply-and-verify-the-policy)
- [Test the Lockout Policy is Working](#-test-the-lockout-policy-is-working)
- [Manage Locked Accounts](#-manage-locked-accounts)
- [Finding and Auditing Lockout Events](#-finding-and-auditing-lockout-events)
- [Take a Snapshot](#-take-a-snapshot)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)

---

## 🛡️ Why Account Lockout Matters

Without an account lockout policy an attacker who knows a valid username
can attempt passwords indefinitely until they find the right one.

```
┌─────────────────────────────────────────────────────────────┐
│              Without Account Lockout                         │
│                                                             │
│   Attacker tries:  password1    ❌ Fail — try again         │
│                    password2    ❌ Fail — try again         │
│                    password3    ❌ Fail — try again         │
│                    ...          ❌ Fail — try again         │
│                    Lab@2024!    ✅ Success — full access     │
│                                                             │
│   Result: Domain compromised with no detection              │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│              With Account Lockout (this GPO)                 │
│                                                             │
│   Attacker tries:  password1    ❌ Fail  (attempt 1 of 5)  │
│                    password2    ❌ Fail  (attempt 2 of 5)   │
│                    password3    ❌ Fail  (attempt 3 of 5)   │
│                    password4    ❌ Fail  (attempt 4 of 5)   │
│                    password5    ❌ Fail  (attempt 5 of 5)   │
│                                 🔒 ACCOUNT LOCKED           │
│                                                             │
│   Result: Attack stopped — IT alerted — 30 min delay        │
└─────────────────────────────────────────────────────────────┘
```

**Real world impact:** Account lockout is a fundamental control required
by security frameworks including ISO 27001, Cyber Essentials, and NIST SP 800-53.
You will encounter and manage this policy in virtually every IT role.

---

## 🔐 The Three Lockout Settings Explained

Account lockout is controlled by exactly three settings — all configured
in the same location as the password policy:

### Setting 1 — Account Lockout Threshold

| Setting | Value | What It Does |
|---------|-------|-------------|
| Account lockout threshold | `5` invalid attempts | Account locks after 5 consecutive wrong passwords |

**Choosing the right value:**

| Value | Security | User Impact | Recommended For |
|-------|----------|-------------|----------------|
| 0 | ❌ No lockout at all | None | Never recommended |
| 3 | Very high | Users locked out frequently | High security environments |
| **5** | **High** | **Reasonable tolerance** | **Standard enterprise ✅** |
| 10 | Medium | Low friction for users | Less security-conscious environments |

> 💡 **Why 5?** It tolerates a user mistyping their password a few times
> while still blocking automated brute force tools that try thousands of
> passwords per second.

---

### Setting 2 — Account Lockout Duration

| Setting | Value | What It Does |
|---------|-------|-------------|
| Account lockout duration | `30` minutes | Locked account automatically unlocks after 30 minutes |

**Two approaches:**

| Duration | Behaviour | Best For |
|----------|-----------|----------|
| 0 minutes | Account stays locked until admin manually unlocks | Maximum security — requires IT intervention |
| **30 minutes** | **Auto-unlocks after 30 minutes** | **Balance of security and usability ✅** |

> 💡 **Setting 0 = permanent lock** — the helpdesk must manually unlock
> every locked account. This is more secure but creates significant
> helpdesk burden. 30 minutes is the standard enterprise balance.

---

### Setting 3 — Reset Account Lockout Counter After

| Setting | Value | What It Does |
|---------|-------|-------------|
| Reset account lockout counter after | `30` minutes | Failed attempt counter resets to 0 after 30 minutes of no failures |

**How the counter works:**

```
10:00 — User fails login attempt 1  (counter: 1/5)
10:05 — User fails login attempt 2  (counter: 2/5)
10:06 — User succeeds               (counter resets to 0)

OR

10:00 — User fails login attempt 1  (counter: 1/5)
10:30 — 30 minutes pass with no attempts
10:31 — Counter automatically resets to 0
10:32 — User fails login attempt 1  (counter: 1/5 — fresh start)
```

> 💡 **This value should always equal the lockout duration.**
> Setting both to 30 minutes creates a consistent, predictable behaviour
> that is easy to communicate to users and the helpdesk.

---

## ⚙️ Configure the Account Lockout Policy

### Step 1 — Open Default Domain Policy for Editing

In **Group Policy Management** right-click **"Default Domain Policy"** → **"Edit"**.

The Group Policy Management Editor opens.

---

### Step 2 — Navigate to Account Lockout Policy

In the left panel navigate to:

```
Default Domain Policy [DC01.Lab.local]
└── Computer Configuration
    └── Policies
        └── Windows Settings
            └── Security Settings
                └── Account Policies
                    └── Account Lockout Policy    ← Click here
```

Click on **"Account Lockout Policy"**.

Three settings appear in the right panel — all showing **"Not Defined"**.

---

### Step 3 — Configure Account Lockout Threshold First

> ⚠️ **Always configure the threshold setting first.**
> When you set the threshold, Windows automatically suggests values
> for the other two settings. This saves time and prevents
> inconsistent configurations.

Double-click **"Account lockout threshold"**.

Tick **"Define this policy setting"**.

Set the value to:
```
5
```

Click **OK**.

A dialog box immediately appears:

```
┌─────────────────────────────────────────────────────────┐
│  Suggested Value Changes                                 │
│                                                         │
│  Because the value of Account lockout threshold is      │
│  now 5 invalid logon attempts, the settings for the     │
│  following items will be changed to the suggested       │
│  values:                                                │
│                                                         │
│  Account lockout duration:          30 minutes          │
│  Reset account lockout counter after: 30 minutes        │
│                                                         │
│                    [ OK ]    [ Cancel ]                  │
└─────────────────────────────────────────────────────────┘
```

Click **OK** to accept the suggested values.

Windows automatically sets both duration and counter to 30 minutes. ✅

---

### Step 4 — Verify All Three Settings

The right panel should now show:

```
Policy                                    Policy Setting
------                                    --------------
Account lockout duration                  30 minutes
Account lockout threshold                 5 invalid logon attempts
Reset account lockout counter after       30 minutes
```

---

### Step 5 — Confirm the Duration Value

Double-click **"Account lockout duration"** to confirm it shows:

```
✅ Define this policy setting
Value: 30 minutes
```

Click **OK**.

---

### Step 6 — Confirm the Reset Counter Value

Double-click **"Reset account lockout counter after"** to confirm it shows:

```
✅ Define this policy setting
Value: 30 minutes
```

Click **OK**.

---

### Step 7 — Close the Editor

Close the **Group Policy Management Editor**.

All three settings are saved automatically.

---

## ✅ Apply and Verify the Policy

### Force the Policy to Apply

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

### Verify All Three Lockout Values

```powershell
Get-ADDefaultDomainPasswordPolicy |
    Select-Object LockoutThreshold, LockoutDuration, LockoutObservationWindow
```

Expected output:

```
LockoutThreshold  LockoutDuration  LockoutObservationWindow
----------------  ---------------  ------------------------
5                 00:30:00         00:30:00
```

All three values confirmed:
- `LockoutThreshold: 5` ✅
- `LockoutDuration: 00:30:00` (30 minutes) ✅
- `LockoutObservationWindow: 00:30:00` (30 minutes) ✅

---

### Full Domain Policy Summary

Run this to see the complete password and lockout policy together:

```powershell
Get-ADDefaultDomainPasswordPolicy | Format-List
```

Expected output — the complete domain policy at a glance:

```
ComplexityEnabled           : True          ← GPO 1 ✅
LockoutDuration             : 00:30:00      ← GPO 2 ✅
LockoutObservationWindow    : 00:30:00      ← GPO 2 ✅
LockoutThreshold            : 5             ← GPO 2 ✅
MaxPasswordAge              : 90.00:00:00   ← GPO 1 ✅
MinPasswordAge              : 1.00:00:00    ← GPO 1 ✅
MinPasswordLength           : 10            ← GPO 1 ✅
PasswordHistoryCount        : 5             ← GPO 1 ✅
ReversibleEncryptionEnabled : False         ← GPO 1 ✅
```

---

## 🧪 Test the Lockout Policy is Working

### Test 1 — Trigger a Lockout on a Test User

On **WS01** at the login screen, select **Other user** and deliberately
enter the wrong password 5 times for a domain user:

```
Username:  LAB\rgomez
Password:  wrongpassword1   ← Attempt 1
Password:  wrongpassword2   ← Attempt 2
Password:  wrongpassword3   ← Attempt 3
Password:  wrongpassword4   ← Attempt 4
Password:  wrongpassword5   ← Attempt 5
```

After the 5th attempt you will see:

```
Your account has been locked out.
Please contact your system administrator.
```

✅ The lockout policy is working correctly.

---

### Test 2 — Confirm the Account is Locked on DC01

On **DC01** open PowerShell and run:

```powershell
Get-ADUser -Identity "rgomez" -Properties LockedOut, BadLogonCount, LastBadPasswordAttempt |
    Select-Object Name, LockedOut, BadLogonCount, LastBadPasswordAttempt
```

Expected output:

```
Name          LockedOut  BadLogonCount  LastBadPasswordAttempt
----          ---------  -------------  ----------------------
Ronald Gomez  True       5              20/04/2026 09:15:42   ✅
```

`LockedOut: True` confirms the account is locked. ✅

---

### Test 3 — Manually Unlock the Account

```powershell
# Unlock the account immediately
Unlock-ADAccount -Identity "rgomez"

# Verify it is unlocked
Get-ADUser -Identity "rgomez" -Properties LockedOut |
    Select-Object Name, LockedOut
```

Expected output:

```
Name          LockedOut
----          ---------
Ronald Gomez  False     ✅
```

---

## 🔑 Manage Locked Accounts

These are the most important day-to-day commands for managing
account lockouts — skills used daily by helpdesk and sysadmin teams:

### Find All Currently Locked Accounts

```powershell
Search-ADAccount -LockedOut |
    Select-Object Name, SamAccountName, LockedOut, LastLogonDate |
    Format-Table -AutoSize
```

---

### Unlock a Single Account

```powershell
Unlock-ADAccount -Identity "rgomez"
```

---

### Unlock All Locked Accounts in the Domain

```powershell
# Use with caution — unlocks ALL locked accounts
Search-ADAccount -LockedOut |
    Unlock-ADAccount

Write-Host "All locked accounts have been unlocked." -ForegroundColor Green
```

---

### Check Lockout Details for a Specific User

```powershell
Get-ADUser -Identity "rgomez" `
    -Properties LockedOut, BadLogonCount, BadPasswordTime,
                LastBadPasswordAttempt, LastLogonDate |
    Select-Object Name, SamAccountName, LockedOut,
                  BadLogonCount, LastBadPasswordAttempt, LastLogonDate
```

---

### Reset Password and Unlock Simultaneously

```powershell
# Unlock and reset password in one block
Set-ADAccountPassword `
    -Identity "rgomez" `
    -NewPassword (ConvertTo-SecureString "NewP@ss2024!" -AsPlainText -Force) `
    -Reset

Unlock-ADAccount -Identity "rgomez"

Set-ADUser -Identity "rgomez" -ChangePasswordAtLogon $true

Write-Host "Password reset and account unlocked for rgomez" -ForegroundColor Green
```

---

## 🔍 Finding and Auditing Lockout Events

Windows records every lockout event in the Security Event Log.
This is how IT administrators investigate suspicious activity.

### Find Lockout Events in the Event Log

On **DC01** run:

```powershell
# Search for account lockout events (Event ID 4740)
Get-WinEvent -FilterHashtable @{
    LogName   = "Security"
    Id        = 4740
    StartTime = (Get-Date).AddDays(-1)
} |
Select-Object TimeCreated, Message |
Format-List
```

**Key Security Event IDs to know:**

| Event ID | Description |
|----------|-------------|
| 4625 | Failed logon attempt |
| 4740 | Account was locked out |
| 4767 | Account was unlocked |
| 4723 | User attempted to change password |
| 4724 | Admin reset a user password |
| 4728 | User added to a security group |
| 4648 | Logon using explicit credentials |

---

### Enable Audit Policy for Logon Events

To make the above events appear in the Security log, enable auditing:

In **Group Policy Management** right-click **Default Domain Policy** → **Edit**.

Navigate to:

```
Computer Configuration → Policies → Windows Settings
→ Security Settings → Local Policies → Audit Policy
```

Configure:

| Policy | Setting |
|--------|---------|
| Audit account logon events | Success, Failure |
| Audit account management | Success, Failure |
| Audit logon events | Success, Failure |

---

## 📸 Take a Snapshot

Account Lockout Policy is configured and tested.

In VirtualBox go to:
```
Machine → Take Snapshot
```

Name it:
```
DC01 - GPO2 Account Lockout Configured
```

Click **OK**.

---

## 🛠️ Troubleshooting

### Issue: LockoutThreshold still shows 0 after gpupdate

The Default Domain Policy changes are not applying correctly.

**Fix:**

```powershell
# Check which GPO is providing the lockout setting
gpresult /h C:\GPOReport.html /f
Start-Process C:\GPOReport.html
```

Open the HTML report and look under **Computer Configuration → Account Policies**.
Identify which GPO is winning for these settings.

If another GPO is overriding the Default Domain Policy:

In Group Policy Management → Lab.local → Linked GPO Objects tab
→ confirm Default Domain Policy has **Link Order 1** (highest priority).

---

### Issue: Account does not lock after 5 failed attempts on WS01

Group Policy may not have applied to WS01 yet.

**Fix — on WS01:**

```powershell
gpupdate /force /boot
```

The `/boot` flag forces a restart — Computer Configuration policies
including lockout require a full restart to apply on the workstation.

---

### Issue: "Unlock-ADAccount is not recognised"

The Active Directory module is not loaded.

**Fix:**

```powershell
Import-Module ActiveDirectory
Unlock-ADAccount -Identity "rgomez"
```

---

### Issue: Get-WinEvent returns no events for Event ID 4740

Audit policy for account management is not enabled.

**Fix:**

Enable the audit policy as described in the **Finding and Auditing Lockout Events**
section above, then run `gpupdate /force` on DC01 and trigger a lockout again.

---

### Issue: Locked account auto-unlocked before the 30 minutes elapsed

The lockout duration setting may not have applied or is set to 0.

**Fix:**

```powershell
# Verify the lockout duration
Get-ADDefaultDomainPasswordPolicy | Select-Object LockoutDuration

# If showing 00:00:00 (0 minutes) it means permanent lockout was set
# Go back and set Account Lockout Duration to 30 in the GPO
```

---

## ✅ Verification Checklist

```
Group Policy Configuration
[ ] Default Domain Policy opened for editing
[ ] Navigated to Account Lockout Policy correctly
[ ] Account lockout threshold set to 5
[ ] Suggested values popup accepted — auto-set both to 30 minutes
[ ] Account lockout duration confirmed: 30 minutes
[ ] Reset account lockout counter confirmed: 30 minutes
[ ] Editor closed — settings saved

Policy Applied
[ ] gpupdate /force ran successfully on DC01
[ ] Get-ADDefaultDomainPasswordPolicy shows LockoutThreshold: 5
[ ] Get-ADDefaultDomainPasswordPolicy shows LockoutDuration: 00:30:00
[ ] Get-ADDefaultDomainPasswordPolicy shows LockoutObservationWindow: 00:30:00

Policy Tested
[ ] 5 wrong passwords entered for test user on WS01
[ ] "Account locked out" message appeared on WS01
[ ] Get-ADUser shows LockedOut: True on DC01
[ ] Unlock-ADAccount successfully unlocked the account
[ ] Get-ADUser shows LockedOut: False after unlock

Account Management Commands Verified
[ ] Search-ADAccount -LockedOut works correctly
[ ] Unlock-ADAccount works correctly
[ ] Password reset and unlock combined command tested

Snapshot
[ ] Snapshot taken named "DC01 - GPO2 Account Lockout Configured"

Ready to proceed
[ ] Account lockout is live and tested
[ ] Helpdesk unlock commands confirmed working
[ ] Both GPO 1 and GPO 2 are active and verified
[ ] Ready to configure USB Storage restriction
```

---

## ➡️ Next Step

Password Policy and Account Lockout are both live.
Now we deploy a security GPO that prevents USB storage
devices from being used on domain workstations.

**[16 - GPO 3 Disable USB Storage →](../16-GPO3-Disable-USB/README.md)**

---

## ⬅️ Previous Step

**[← 14 - GPO 1 Password Policy](../14-GPO1-Password-Policy/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
