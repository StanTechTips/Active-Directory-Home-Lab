# 10 - Configure Lab Networking

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Intermediate-orange?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/VM-DC01_%26_WS01-purple?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Network-192.168.100.0%2F24-blue?style=for-the-badge)

> Networking is the invisible glue that holds this entire lab together.
> Without correct network configuration DC01 and WS01 cannot communicate —
> which means no domain joining, no Group Policy delivery, and no AD authentication.
> This step explains the complete network design and verifies everything
> is correctly configured on both VMs before WS01 is created.

---

## 📑 Table of Contents

- [The Lab Network Design](#-the-lab-network-design)
- [How VirtualBox Networking Works](#-how-virtualbox-networking-works)
- [DC01 Network Configuration](#-dc01-network-configuration)
- [Verify DC01 Networking](#-verify-dc01-networking)
- [What WS01 Will Need](#-what-ws01-will-need)
- [The LabNet Internal Network Explained](#-the-labnet-internal-network-explained)
- [Common Networking Mistakes](#-common-networking-mistakes)
- [Take a Snapshot](#-take-a-snapshot)
- [Troubleshooting](#-troubleshooting)
- [Verification Checklist](#-verification-checklist)

---

## 🗺️ The Lab Network Design

Here is the complete network architecture for the entire lab:

```
┌─────────────────────────────────────────────────────────────────┐
│                   Physical Host PC                               │
│                                                                 │
│  ┌──────────────────────────┐   ┌──────────────────────────┐   │
│  │          DC01            │   │          WS01            │   │
│  │   Windows Server 2022    │   │      Windows 11 Pro      │   │
│  │                          │   │                          │   │
│  │  ┌────────────────────┐  │   │  ┌────────────────────┐  │   │
│  │  │  Adapter 1 — NAT   │  │   │  │ Adapter 1          │  │   │
│  │  │  10.0.2.15 (auto)  │  │   │  │ Internal Network   │  │   │
│  │  │  Internet access   │  │   │  │ 192.168.100.20     │  │   │
│  │  └────────────────────┘  │   │  │ DNS: 192.168.100.10│  │   │
│  │                          │   │  └────────────────────┘  │   │
│  │  ┌────────────────────┐  │   └────────────┬─────────────┘   │
│  │  │  Adapter 2         │  │                │                  │
│  │  │  Internal Network  │  │                │                  │
│  │  │  192.168.100.10    │  │                │                  │
│  │  │  DNS: 127.0.0.1    │  │                │                  │
│  │  └─────────┬──────────┘  │                │                  │
│  └────────────┼─────────────┘                │                  │
│               │                              │                  │
│               └──────────────┬───────────────┘                  │
│                              │                                   │
│              ┌───────────────▼────────────────┐                  │
│              │     LabNet Internal Network     │                  │
│              │      192.168.100.0 / 24         │                  │
│              │      Domain: Lab.local          │                  │
│              └─────────────────────────────────┘                  │
│                              │                                   │
│              ┌───────────────▼────────────────┐                  │
│              │     VirtualBox NAT (DC01 only)  │                  │
│              │         🌐 Internet             │                  │
└─────────────────────────────────────────────────────────────────┘
```

### IP Address Summary

| VM | Adapter | Network Type | IP Address | Purpose |
|----|---------|-------------|-----------|---------|
| DC01 | Adapter 1 | NAT | 10.0.2.15 (auto) | Internet access for downloads |
| DC01 | Adapter 2 | Internal — LabNet | 192.168.100.10 (static) | Domain Controller address |
| WS01 | Adapter 1 | Internal — LabNet | 192.168.100.20 (static) | Workstation domain address |

---

## 🔧 How VirtualBox Networking Works

VirtualBox offers several network adapter types. This lab uses two of them:

### NAT (Network Address Translation)

```
VM  ──►  VirtualBox NAT Engine  ──►  Host PC  ──►  Internet
         (translates addresses)
```

| Property | Value |
|----------|-------|
| VM gets internet access | ✅ Yes |
| VM is visible to other VMs | ❌ No |
| VM is visible from host | ❌ No (by default) |
| IP auto-assigned | ✅ Yes — always 10.0.2.15 |
| Used for | Internet access on DC01 only |

---

### Internal Network

```
VM1 (DC01)  ◄──────────────────►  VM2 (WS01)
            VirtualBox Internal
            Network Switch
            (LabNet)
```

| Property | Value |
|----------|-------|
| VMs can communicate with each other | ✅ Yes |
| VMs can reach the internet | ❌ No |
| Host PC can see these VMs | ❌ No |
| IP auto-assigned | ❌ No — must be set manually |
| Used for | DC01 ↔ WS01 domain communication |

> 💡 **Why no internet on WS01?**
> WS01 only needs to reach DC01 to join the domain and receive Group Policy.
> It does not need internet access for any step in this lab.
> Keeping WS01 on the internal network only is also more secure
> and mirrors how locked-down corporate workstations are often configured.

---

## ⚙️ DC01 Network Configuration

DC01 needs both adapters configured correctly. Here is the current required state:

### Adapter 1 — NAT (Internet)

| Setting | Required Value |
|---------|---------------|
| VirtualBox type | NAT |
| IP assignment | Automatic (DHCP from VirtualBox) |
| Expected IP | 10.0.2.15 |
| DNS | Provided automatically by VirtualBox |
| Action needed | None — leave completely on automatic |

---

### Adapter 2 — Internal Network (Domain)

| Setting | Required Value |
|---------|---------------|
| VirtualBox type | Internal Network |
| VirtualBox network name | `LabNet` (case-sensitive) |
| IP assignment | Static — set manually |
| IP address | `192.168.100.10` |
| Subnet mask | `255.255.255.0` |
| Default gateway | Leave blank |
| Preferred DNS | `127.0.0.1` |
| Action needed | Verify static IP is still set |

### Verify DC01 Adapter Settings in VirtualBox

**With DC01 powered off**, go to **DC01 → Settings → Network** and confirm:

**Adapter 1 tab:**
```
Enable Network Adapter:  ✅
Attached to:             NAT
```

**Adapter 2 tab:**
```
Enable Network Adapter:  ✅
Attached to:             Internal Network
Name:                    LabNet
```

> ⚠️ **Critical:** The Internal Network name must be exactly `LabNet`.
> Capital L, capital N, no spaces.
> This name must be **identical** on both DC01 and WS01.
> A mismatch — even just different capitalisation — puts the VMs
> on completely separate virtual networks that cannot communicate.

---

## 🔍 Verify DC01 Networking

With DC01 running, open **PowerShell as Administrator** and run these checks:

### Check 1 — Both Adapters are Present

```powershell
Get-NetAdapter | Select-Object Name, InterfaceIndex, Status, MacAddress
```

Expected output — two adapters:

```
Name        InterfaceIndex  Status       MacAddress
----        --------------  ------       ----------
Ethernet    14              Up           08-00-27-XX-XX-XX
Ethernet 2  19              Disconnected 08-00-27-XX-XX-XX
```

> `Disconnected` on Ethernet 2 is normal — WS01 does not exist yet.

---

### Check 2 — Static IP is Correctly Set

```powershell
Get-NetIPAddress -AddressFamily IPv4 | Select-Object InterfaceAlias, IPAddress, PrefixOrigin
```

Expected output:

```
InterfaceAlias  IPAddress       PrefixOrigin
--------------  ---------       ------------
Ethernet        10.0.2.15       Dhcp          ← NAT adapter ✅
Ethernet 2      192.168.100.10  Manual        ← Static IP ✅
```

`Manual` on the internal adapter confirms the static IP is correctly saved.

---

### Check 3 — DNS is Pointing to Loopback

```powershell
Get-DnsClientServerAddress -AddressFamily IPv4 |
    Select-Object InterfaceAlias, ServerAddresses
```

Expected output:

```
InterfaceAlias  ServerAddresses
--------------  ---------------
Ethernet        {10.0.2.3, ...}    ← ISP DNS from VirtualBox NAT ✅
Ethernet 2      {127.0.0.1}        ← Loopback — DC's own DNS ✅
```

---

### Check 4 — DNS Service is Running

```powershell
Get-Service -Name DNS | Select-Object Name, Status, StartType
```

Expected output:

```
Name  Status   StartType
----  ------   ---------
DNS   Running  Automatic
```

---

### Check 5 — Domain Resolves Correctly

```powershell
Resolve-DnsName -Name "DC01.Lab.local"
```

Expected output:

```
Name              Type  TTL   Section    IPAddress
----              ----  ---   -------    ---------
DC01.Lab.local    A     1200  Answer     192.168.100.10
```

DC01's hostname resolves to `192.168.100.10` — DNS is working correctly. ✅

---

## 🖥️ What WS01 Will Need

When WS01 is created in the next step it needs the following network configuration:

### VirtualBox Settings (set before first boot)

```
Adapter 1:
  Enable Network Adapter:  ✅
  Attached to:             Internal Network
  Name:                    LabNet    ← Must match DC01 exactly
```

### Windows Network Settings (set after Windows 11 is installed)

```
IP address:       192.168.100.20
Subnet mask:      255.255.255.0
Default gateway:  192.168.100.10    ← Points to DC01
Preferred DNS:    192.168.100.10    ← Points to DC01's DNS
```

> ⚠️ **The DNS setting is the most critical.**
> WS01's DNS server MUST point to DC01's IP address (`192.168.100.10`).
> Without this, WS01 cannot resolve `Lab.local` and domain joining will fail
> with the error "The specified domain either does not exist or could not be contacted."

### Why WS01's Gateway Points to DC01

WS01 uses DC01 as its default gateway on the internal network. Even though DC01
is not a proper router, this tells WS01 where to send traffic it cannot deliver locally.
More importantly it ensures all DNS queries go to DC01 first.

---

## 🌐 The LabNet Internal Network Explained

The `LabNet` internal network is a completely isolated virtual switch inside VirtualBox.
It exists only inside your host PC and is invisible to the outside world.

```
What LabNet is:
┌─────────────────────────────────────────────────────────────┐
│                     LabNet Switch                           │
│                   192.168.100.0/24                          │
│                                                             │
│   Port 1: DC01 Adapter 2    192.168.100.10                  │
│   Port 2: WS01 Adapter 1    192.168.100.20                  │
│                                                             │
│   ← No connection to the internet                           │
│   ← No connection to your home network                      │
│   ← Invisible to your router                                │
│   ← Only these two VMs can communicate on it                │
└─────────────────────────────────────────────────────────────┘
```

### Subnet Design Explained

| Address | Role |
|---------|------|
| `192.168.100.0` | Network address — not usable |
| `192.168.100.1` | Reserved for potential gateway use |
| `192.168.100.10` | DC01 — Domain Controller |
| `192.168.100.20` | WS01 — Workstation |
| `192.168.100.21–254` | Available for future VMs |
| `192.168.100.255` | Broadcast address — not usable |

> 💡 The `/24` subnet mask (`255.255.255.0`) gives us 254 usable addresses.
> This lab only uses two — plenty of room to add more VMs later
> such as a second workstation, a file server, or a DHCP server.

---

## ⚠️ Common Networking Mistakes

These are the most frequently seen networking errors in this lab and how to avoid them:

### Mistake 1 — Wrong Internal Network Name

```
DC01 Adapter 2:  LabNet    ← Correct
WS01 Adapter 1:  labnet    ← Wrong — different capitalisation
```

**Result:** VMs cannot ping each other even though both show as connected.
**Fix:** Both must be exactly `LabNet` — capital L, capital N.

---

### Mistake 2 — Static IP on the NAT Adapter

```
Ethernet (NAT):     192.168.100.10  ← Wrong — static IP on wrong adapter
Ethernet 2 (LAN):   No IP set       ← Wrong
```

**Result:** Internet on DC01 breaks. WS01 cannot find the DC.
**Fix:** NAT adapter stays on automatic. Static IP goes on Ethernet 2 only.

---

### Mistake 3 — WS01 DNS Pointing to Wrong Server

```
WS01 DNS:  8.8.8.8  ← Wrong — Google's DNS knows nothing about Lab.local
WS01 DNS:  192.168.100.10  ← Correct — DC01's DNS knows Lab.local
```

**Result:** Domain join fails with "domain does not exist" error.
**Fix:** WS01 Preferred DNS must always be `192.168.100.10`.

---

### Mistake 4 — VMs on Different Network Types

```
DC01 Adapter 2:  Internal Network  ← Correct
WS01 Adapter 1:  Host-Only Network ← Wrong — different network type
```

**Result:** VMs cannot communicate even on the same IP range.
**Fix:** Both must use **Internal Network** — not Host-Only, not Bridged.

---

### Mistake 5 — Forgetting to Set Static IP After WS01 Install

```
WS01 IP:  169.254.x.x  ← Self-assigned — DHCP failed, no static set
```

**Result:** WS01 gets an APIPA address and cannot reach DC01.
**Fix:** Set static IP `192.168.100.20` on WS01 after Windows 11 installs.

---

## 📸 Take a Snapshot

DC01 networking is verified and ready. Take a snapshot before creating WS01.

In VirtualBox go to:
```
Machine → Take Snapshot
```

Name it:
```
DC01 - Networking Verified - Ready for WS01
```

Click **OK**.

---

## 🛠️ Troubleshooting

### Issue: Ethernet 2 shows as "Disconnected" in Get-NetAdapter

This is normal at this stage — WS01 does not exist yet.

**Expected behaviour:** The adapter will show as **Up** the moment WS01 is
running on the same LabNet internal network.

---

### Issue: Static IP disappeared from Ethernet 2 after a restart

VirtualBox occasionally resets network adapter settings.

**Fix:**
```powershell
# Reapply the static IP
New-NetIPAddress `
    -InterfaceAlias "Ethernet 2" `
    -IPAddress "192.168.100.10" `
    -PrefixLength 24

# Reapply DNS
Set-DnsClientServerAddress `
    -InterfaceAlias "Ethernet 2" `
    -ServerAddresses "127.0.0.1"

# Verify
Get-NetIPAddress -InterfaceAlias "Ethernet 2"
```

---

### Issue: Resolve-DnsName returns "DNS name does not exist"

The DNS service may not have fully initialised after promotion.

**Fix:**
```powershell
# Restart the DNS service
Restart-Service -Name DNS

# Wait 10 seconds then retry
Start-Sleep -Seconds 10
Resolve-DnsName -Name "DC01.Lab.local"
```

---

### Issue: Get-Service DNS shows "Stopped"

The DNS service is not running.

**Fix:**
```powershell
# Start DNS and set it to start automatically
Start-Service -Name DNS
Set-Service -Name DNS -StartupType Automatic

# Verify
Get-Service -Name DNS
```

---

### Issue: Both adapters show the same IP subnet after configuring

The static IP was accidentally applied to both adapters.

**Fix:**
```powershell
# Find which interface has the wrong IP
Get-NetIPAddress -AddressFamily IPv4 | Select-Object InterfaceAlias, IPAddress

# Remove the IP from the NAT adapter (replace index 14 with your NAT adapter's index)
Remove-NetIPAddress -InterfaceIndex 14 -IPAddress "192.168.100.10" -Confirm:$false

# Reset NAT adapter to DHCP
Set-NetIPInterface -InterfaceIndex 14 -Dhcp Enabled
```

---

## ✅ Verification Checklist

```
VirtualBox Settings — DC01
[ ] Adapter 1 set to NAT
[ ] Adapter 2 enabled and set to Internal Network
[ ] Adapter 2 network name is exactly "LabNet" (capital L, capital N)

DC01 Network Configuration
[ ] Ethernet (NAT) has IP 10.0.2.15 — assigned automatically
[ ] Ethernet 2 (Internal) has IP 192.168.100.10 — set manually
[ ] Ethernet 2 shows PrefixOrigin: Manual (static confirmed)
[ ] Ethernet 2 DNS points to 127.0.0.1 (loopback)

Service Checks
[ ] DNS service is Running and set to Automatic startup
[ ] Resolve-DnsName DC01.Lab.local returns 192.168.100.10

Understanding Confirmed
[ ] LabNet name is noted — must use exact same name on WS01
[ ] WS01 will need IP 192.168.100.20 with DNS 192.168.100.10
[ ] Common networking mistakes reviewed and understood

Snapshot
[ ] Snapshot taken named "DC01 - Networking Verified - Ready for WS01"

Ready to proceed
[ ] DC01 networking is fully configured and verified
[ ] Ready to create the WS01 VM in the next step
```

---

## ➡️ Next Step

DC01 networking is solid. Time to build the Windows 11 workstation VM.

**[11 - Create the WS01 VM →](../11-Create-WS01-VM/README.md)**

---

## ⬅️ Previous Step

**[← 09 - Bulk Create Users with PowerShell](../09-Bulk-Create-Users/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
