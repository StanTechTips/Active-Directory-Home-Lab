# 05 - Configure DC01

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Beginner-blue?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/VM-DC01-purple?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Tool-Server_Manager-0078D6?style=for-the-badge&logo=windows)

> Before installing Active Directory, DC01 needs two critical configurations —
> a proper computer name and a static IP address.
> These must be done first. Active Directory relies on both being stable and permanent.

---

## 📑 Table of Contents

- [Why These Steps Come First](#-why-these-steps-come-first)
- [Step 1 — Rename the Computer to DC01](#-step-1--rename-the-computer-to-dc01)
- [Step 2 — Set a Static IP Address](#-step-2--set-a-static-ip-address)
- [Step 3 — Verify Both Adapters](#-step-3--verify-both-adapters)
- [Step 4 — Rename Network Adapters (Optional but Recommended)](#-step-4--rename-network-adapters-optional-but-recommended)
- [Understanding the Network Design](#-understanding-the-network-design)
- [Take a Snapshot](#-take-a-snapshot)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)

---

## 🧠 Why These Steps Come First

Active Directory is deeply integrated with both the **computer name** and the **IP address** of the server it runs on. If either changes after AD is installed, serious problems can occur — DNS records break, Kerberos authentication fails, and domain-joined computers lose trust with the domain.

```
┌─────────────────────────────────────────────────────────────┐
│              Why order matters                               │
│                                                             │
│   ✅ Correct order:                                          │
│   Rename PC → Set Static IP → Install AD → Promote DC       │
│                                                             │
│   ❌ Wrong order:                                            │
│   Install AD → Rename PC  ← breaks DNS, Kerberos, SPN       │
│   Install AD → Change IP  ← breaks replication, DNS         │
└─────────────────────────────────────────────────────────────┘
```

**Always complete this step before touching Active Directory.**

---

## 🖥️ Step 1 — Rename the Computer to DC01

Windows Server 2022 assigns a random computer name during installation (something like `WIN-K3J8DL2`). We need to rename it to `DC01` before installing AD.

### Method 1 — Via Server Manager (GUI)

**1.** Open **Server Manager** — it opens automatically, or click the flag icon in the taskbar.

**2.** Click **"Local Server"** in the left panel.

**3.** Find the **"Computer name"** row near the top of the Properties section. It will show the random auto-generated name as a blue hyperlink.

**4.** Click the blue hyperlink — the **System Properties** window opens.

**5.** Click the **"Change..."** button.

**6.** In the **"Computer name"** field delete the random name and type:
```
DC01
```

**7.** Click **OK**.

**8.** A popup appears:
```
You must restart your computer to apply these changes.
```
Click **OK** → Click **Close** → Click **Restart Now**.

**9.** After the restart log back in as Administrator.

**10.** Open Server Manager → Local Server and confirm the computer name now shows **DC01**.

---

### Method 2 — Via PowerShell (Faster)

Open **PowerShell as Administrator** (right-click Start → Windows PowerShell (Admin)) and run:

```powershell
# Rename the computer and restart automatically
Rename-Computer -NewName "DC01" -Restart -Force
```

After the automatic restart, log back in and verify:

```powershell
# Confirm the new name
$env:COMPUTERNAME
```

Expected output:
```
DC01
```

---

## 🌐 Step 2 — Set a Static IP Address

A Domain Controller **must** have a static IP address. If the IP changes the DNS records for the domain will point to the wrong address and every domain-joined computer will lose connectivity to the domain.

### Which Adapter Gets the Static IP?

DC01 has **two network adapters** — you need to set the static IP on the correct one:

| Adapter | Type | IP Assignment | Purpose |
|---------|------|--------------|---------|
| Ethernet (Adapter 1) | NAT | Automatic (DHCP) — leave alone | Internet access |
| Ethernet 2 (Adapter 2) | Internal Network | **Static — configure this one** | Domain network |

> ⚠️ **Common mistake:** Setting the static IP on the NAT adapter instead of the Internal Network adapter.
> The NAT adapter should always remain on automatic/DHCP.
> Only Ethernet 2 (the Internal Network adapter) gets the static IP.

### Method 1 — Via Network Connections (GUI)

**1.** In Server Manager → Local Server, find the **"Ethernet"** row and click the blue hyperlink next to it.

**2.** The **Network Connections** window opens showing both adapters.

**3.** Identify **Ethernet 2** — it will show as **"Network cable unplugged"** or **"Unidentified network"**. This is the Internal Network adapter.

**4.** Right-click **Ethernet 2** → **Properties**.

**5.** Double-click **"Internet Protocol Version 4 (TCP/IPv4)"**.

**6.** Select **"Use the following IP address"** and enter:

| Field | Value |
|-------|-------|
| IP address | `192.168.100.10` |
| Subnet mask | `255.255.255.0` |
| Default gateway | *(leave completely blank)* |
| Preferred DNS server | `127.0.0.1` |
| Alternate DNS server | *(leave completely blank)* |

**7.** Click **OK → OK → Close**.

> 💡 **Why DNS points to 127.0.0.1 (loopback)?**
> Once Active Directory is installed DC01 runs its own DNS server.
> Pointing DC01's DNS to itself (127.0.0.1 = loopback = this computer)
> means DC01 resolves domain names using its own DNS service.
> This is the correct and standard configuration for a Domain Controller.

---

### Method 2 — Via PowerShell

Open **PowerShell as Administrator** and run:

```powershell
# Find the index number of the Internal Network adapter
Get-NetAdapter | Format-Table Name, InterfaceIndex, Status, LinkSpeed
```

Look for the adapter that shows **"Disconnected"** status — that is your Internal Network adapter (Ethernet 2). Note its **InterfaceIndex** number.

Then run (replace `19` with your actual InterfaceIndex if different):

```powershell
# Set the static IP address
New-NetIPAddress `
    -InterfaceIndex 19 `
    -IPAddress "192.168.100.10" `
    -PrefixLength 24

# Set DNS to loopback
Set-DnsClientServerAddress `
    -InterfaceIndex 19 `
    -ServerAddresses "127.0.0.1"
```

---

## 🔍 Step 3 — Verify Both Adapters

After setting the static IP, verify both adapters are configured correctly.

Run in PowerShell:

```powershell
Get-NetIPConfiguration
```

### Expected Output

```
InterfaceAlias       : Ethernet
InterfaceDescription : Intel(R) PRO/1000 MT Desktop Adapter
NetProfile.Name      : Lab.local (or Network)
IPv4Address          : 10.0.2.15          ← NAT adapter — auto assigned ✅
IPv4DefaultGateway   : 10.0.2.2
DNSServer            : 10.0.2.3

InterfaceAlias       : Ethernet 2
InterfaceDescription : Intel(R) PRO/1000 MT Desktop Adapter #2
NetAdapter.Status    : Disconnected       ← Normal — WS01 doesn't exist yet ✅
IPv4Address          : 192.168.100.10     ← Static IP correctly set ✅
DNSServer            : 127.0.0.1          ← Pointing to itself ✅
```

> 💡 **"Disconnected" on Ethernet 2 is completely normal at this stage.**
> The Internal Network adapter shows disconnected because WS01 does not exist yet —
> there is no other device on the LabNet network.
> The moment WS01 comes online this status resolves automatically.

### Verify the Static IP is Saved

```powershell
# Check the IP is saved even on a disconnected adapter
Get-NetIPAddress -InterfaceAlias "Ethernet 2" | Select-Object IPAddress, PrefixLength, PrefixOrigin
```

Expected output:

```
IPAddress        PrefixLength  PrefixOrigin
---------        ------------  ------------
192.168.100.10   24            Manual        ← "Manual" confirms it is static ✅
```

`PrefixOrigin: Manual` confirms the IP was manually set — not assigned by DHCP.

---

## 🏷️ Step 4 — Rename Network Adapters (Optional but Recommended)

Renaming the adapters makes them much easier to identify throughout the rest of the lab, especially when running PowerShell commands.

```powershell
# Rename the NAT adapter
Rename-NetAdapter -Name "Ethernet" -NewName "NAT-Internet"

# Rename the Internal Network adapter
Rename-NetAdapter -Name "Ethernet 2" -NewName "LAN-LabNet"
```

After renaming your `Get-NetIPConfiguration` output will clearly show:

```
InterfaceAlias : NAT-Internet    ← Much clearer ✅
InterfaceAlias : LAN-LabNet      ← Much clearer ✅
```

> 💡 If you rename the adapters, remember to use the new names
> in any PowerShell commands going forward — for example:
> `Set-DnsClientServerAddress -InterfaceAlias "LAN-LabNet" -ServerAddresses "127.0.0.1"`

---

## 🗺️ Understanding the Network Design

Here is why the network is designed this way and what each component does:

```
┌─────────────────────────────────────────────────────────────┐
│                    DC01 Network Design                       │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                      DC01                           │   │
│  │                                                     │   │
│  │   NAT-Internet              LAN-LabNet              │   │
│  │   10.0.2.15 (auto)          192.168.100.10 (static) │   │
│  │   DNS: ISP servers          DNS: 127.0.0.1          │   │
│  └──────────┬──────────────────────────┬───────────────┘   │
│             │                          │                    │
│             ▼                          ▼                    │
│      VirtualBox NAT              Internal Network           │
│      (internet access)           LabNet 192.168.100.0/24    │
│             │                          │                    │
│             ▼                          ▼                    │
│         🌐 Internet              WS01 (192.168.100.20)      │
│                                  joins domain via this      │
└─────────────────────────────────────────────────────────────┘
```

| Adapter | IP | Purpose | Why This Design |
|---------|-----|---------|----------------|
| NAT-Internet | 10.0.2.15 (auto) | Internet access | Allows downloading files inside the VM during setup |
| LAN-LabNet | 192.168.100.10 (static) | Domain network | WS01 uses this IP to find and communicate with DC01 |

**Why static and not DHCP for the domain network?**

Every domain-joined computer remembers DC01's IP address. If DC01's IP ever changes:
- DNS records for `Lab.local` break
- Kerberos authentication fails for all users
- Group Policy cannot be delivered to workstations
- The entire domain becomes unreachable

Static = stable = reliable domain.

---

## 📸 Take a Snapshot

DC01 is now renamed and has a static IP configured. This is an important save point before installing Active Directory.

In VirtualBox go to:
```
Machine → Take Snapshot
```

Name it:
```
DC01 - Renamed and Static IP Configured - Pre AD Install
```

Click **OK**.

> If the Active Directory installation in the next step goes wrong for any reason,
> you can roll back to this snapshot and have a clean, correctly named server
> with the right IP — ready to try again instantly.

---

## 🛠️ Troubleshooting

### Issue: Computer name still shows the random name after restart

The rename did not apply correctly.

**Fix:**
```powershell
# Check current name
$env:COMPUTERNAME

# If still wrong, rename again
Rename-Computer -NewName "DC01" -Restart -Force
```

---

### Issue: Static IP set but Get-NetIPConfiguration shows no IPv4 address on Ethernet 2

The adapter is disconnected so Windows may not display the IP in the standard view.

**Fix:**
```powershell
# This command shows IPs even on disconnected adapters
Get-NetIPAddress -InterfaceAlias "Ethernet 2"
```

Look for `PrefixOrigin: Manual` on the `192.168.100.10` entry — this confirms it is saved correctly even though the adapter shows as disconnected.

---

### Issue: Both adapters show the same IP range or the NAT adapter has the static IP

The static IP was set on the wrong adapter.

**Fix:**
```powershell
# Remove the wrong static IP from the NAT adapter
# First identify which interface has the wrong IP
Get-NetIPAddress -AddressFamily IPv4

# Remove the incorrect static IP (replace InterfaceIndex with the correct number)
Remove-NetIPAddress -InterfaceIndex 14 -IPAddress "192.168.100.10" -Confirm:$false

# Set it back to DHCP
Set-NetIPInterface -InterfaceIndex 14 -Dhcp Enabled

# Now set the static IP on the correct adapter (Ethernet 2)
New-NetIPAddress -InterfaceIndex 19 -IPAddress "192.168.100.10" -PrefixLength 24
```

---

### Issue: Cannot access the internet after configuring the static IP

The NAT adapter DNS settings may have been accidentally changed.

**Fix:**
```powershell
# Reset the NAT adapter to full automatic
Set-NetIPInterface -InterfaceAlias "Ethernet" -Dhcp Enabled
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ResetServerAddresses

# Verify internet is back
Test-NetConnection -ComputerName "8.8.8.8" -InformationLevel Quiet
```

---

### Issue: Ping from WS01 to 192.168.100.10 fails even after WS01 is set up

The Internal Network adapter names do not match between the two VMs.

**Fix:**
1. Power off both VMs
2. Check **DC01 → Settings → Network → Adapter 2** — note the exact network name
3. Check **WS01 → Settings → Network → Adapter 1** — must be identical
4. Both must say `LabNet` with the same capitalisation
5. Correct whichever one is wrong and restart both VMs

---

## ✅ Verification Checklist

```
Computer Name
[ ] Server renamed to DC01 successfully
[ ] Restart completed after rename
[ ] Server Manager → Local Server shows Computer name: DC01
[ ] PowerShell $env:COMPUTERNAME returns "DC01"

Static IP Configuration
[ ] Static IP set on Ethernet 2 (Internal Network adapter) only
[ ] NAT adapter (Ethernet) left on automatic DHCP
[ ] IP address: 192.168.100.10 confirmed
[ ] Subnet mask: 255.255.255.0 confirmed
[ ] Preferred DNS: 127.0.0.1 confirmed
[ ] Get-NetIPAddress shows PrefixOrigin: Manual on 192.168.100.10

Network Verification
[ ] Get-NetIPConfiguration shows both adapters
[ ] NAT adapter shows 10.0.2.x address (auto)
[ ] Internal adapter shows 192.168.100.10 (static)
[ ] Internet still accessible via NAT adapter

Snapshot
[ ] Snapshot taken named "DC01 - Renamed and Static IP Configured - Pre AD Install"

Ready to proceed
[ ] Computer name is DC01
[ ] Static IP 192.168.100.10 is set and saved
[ ] Both adapters verified in PowerShell
```

---

## ➡️ Next Step

DC01 is correctly named and has a stable IP address.
Time to install the Active Directory Domain Services role.

**[06 - Install Active Directory Domain Services →](../06-Install-Active-Directory/README.md)**

---

## ⬅️ Previous Step

**[← 04 - Install Windows Server 2022](../04-Install-Windows-Server-2022/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
