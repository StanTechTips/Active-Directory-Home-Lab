# 01 - Pre-Requirements

![Static Badge](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Difficulty-Beginner-blue?style=for-the-badge)
![Static Badge](https://img.shields.io/badge/Platform-Windows-0078D6?style=for-the-badge&logo=windows)

> Before touching a single VM, make sure your host machine and downloads are ready.
> Skipping this step is the number one cause of problems later in the lab.

---

## 📑 Table of Contents

- [Hardware Requirements](#-hardware-requirements)
- [Check Virtualisation is Enabled](#-check-virtualisation-is-enabled)
- [Software Downloads](#-software-downloads)
- [Disk Space Planning](#-disk-space-planning)
- [Pre-Flight Checklist](#-pre-flight-checklist)

---

## 💻 Hardware Requirements

This lab runs **two virtual machines simultaneously** on your physical PC.
Your host machine needs to be able to handle both at the same time comfortably.

| Component | Minimum | Recommended | Why It Matters |
|-----------|---------|-------------|----------------|
| RAM | 16 GB | 32 GB | DC01 needs 4 GB + WS01 needs 4 GB + your host OS needs at least 4 GB |
| CPU | Quad-core with VT-x / AMD-V | 6-core or higher | Virtualisation extensions are mandatory — without them VMs won't boot |
| Storage | 100 GB free (SSD) | 200 GB free (NVMe SSD) | HDD will make Windows Server boot extremely slowly — SSD is strongly recommended |
| Host OS | Windows 10 / 11, macOS, Linux | Windows 11 | Any modern OS works as the VirtualBox host |

> ⚠️ **Running on a laptop?** Make sure your charger is plugged in. Running two VMs on battery will throttle your CPU and everything will feel sluggish.

---

## ⚙️ Check Virtualisation is Enabled

Virtualisation extensions must be enabled in your BIOS before VirtualBox can run 64-bit VMs.

### Check on Windows (30 seconds)

1. Press **Ctrl + Shift + Esc** to open Task Manager
2. Click the **Performance** tab
3. Click **CPU** in the left panel
4. Look at the bottom right of the CPU graph

```
Virtualisation:  Enabled   ✅  You are good to go
Virtualisation:  Disabled  ❌  Follow the steps below
```

### Enable Virtualisation in BIOS (if disabled)

1. Restart your PC
2. Press the BIOS key repeatedly as it boots — usually **Del**, **F2**, **F10**, or **F12** (check your motherboard manual)
3. Navigate to **CPU Configuration** or **Advanced Settings**
4. Find the setting called one of the following and set it to **Enabled**:
   - Intel: `Intel VT-x` or `Intel Virtualisation Technology`
   - AMD: `AMD-V` or `SVM Mode`
5. Press **F10** to save and exit

> 💡 After enabling VT-x/AMD-V in BIOS, go back and recheck Task Manager — it should now show **Enabled**.

---

## 📥 Software Downloads

Download all four items below **before** starting the lab. Having everything ready avoids stopping halfway through a VM setup.

### 1. Oracle VirtualBox

The free hypervisor that runs all your VMs.

| Item | Download Link |
|------|--------------|
| VirtualBox Installer | https://www.virtualbox.org/wiki/Downloads |
| VirtualBox Extension Pack | https://www.virtualbox.org/wiki/Downloads |

> ⚠️ **The Extension Pack version must exactly match your VirtualBox version.** Download both from the same page at the same time.

**What the Extension Pack adds:**
- USB 2.0 and 3.0 device support
- VirtualBox Remote Desktop Protocol (VRDP)
- Host webcam passthrough
- Disk image encryption

---

### 2. Windows Server 2022 ISO

The free 180-day evaluation from Microsoft. No licence key needed.

| Item | Details |
|------|---------|
| Download URL | https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022 |
| File Format | **ISO** (not VHD) |
| File Size | ~4.7 GB |
| Validity | 180 days — fully functional evaluation |
| Edition to use | Windows Server 2022 Standard Evaluation (Desktop Experience) |

**How to download:**
1. Click the link above
2. Fill in the short registration form (name and email — any values work)
3. Select **ISO** as the download format
4. Select your language and click Download

---

### 3. Windows 11 ISO

The client workstation OS that will join your domain.

| Item | Details |
|------|---------|
| Download URL | https://www.microsoft.com/en-gb/software-download/windows11 |
| File Size | ~6.5 GB |
| Edition to use | Windows 11 Pro — **do not use Home edition** |

> ❌ **Why not Windows 11 Home?**
> Home edition cannot join Active Directory domains.
> Only **Pro**, **Education**, and **Enterprise** editions support domain joining.
> Always choose **Windows 11 Pro** for this lab.

**How to download:**
1. Click the link above
2. Scroll to **"Download Windows 11 Disk Image (ISO)"**
3. Select **Windows 11 (multi-edition ISO)**
4. Click Download → select your language → click **64-bit Download**

---

## 📦 Disk Space Planning

Here is exactly how much space each component will use on your hard drive:

| Component | Initial Size | After Full Setup | Notes |
|-----------|-------------|-----------------|-------|
| Windows Server 2022 ISO | 4.7 GB | — | Can delete after install |
| Windows 11 ISO | 6.5 GB | — | Can delete after install |
| DC01 Virtual Hard Disk | 60 GB allocated | ~25 GB actual | Dynamically allocated — grows as needed |
| WS01 Virtual Hard Disk | 50 GB allocated | ~20 GB actual | Dynamically allocated — grows as needed |
| VM Snapshots | Variable | ~5–15 GB each | Snapshots save state — take them generously |
| **Total recommended free** | **— ** | **~100 GB minimum** | **150 GB+ preferred for snapshots** |

> 💡 **Dynamically allocated** means the `.vhd` file on your host PC only uses as much real disk space as data has actually been written to it — it does not immediately consume the full allocated size.

---

## ✅ Pre-Flight Checklist

Work through this list before moving to Step 02. Every box should be ticked before proceeding.

```
Hardware
[ ] Host PC has at least 16 GB RAM
[ ] CPU has virtualisation extensions (VT-x or AMD-V)
[ ] Virtualisation shows as "Enabled" in Task Manager
[ ] At least 100 GB of free SSD storage available

Downloads
[ ] Oracle VirtualBox installer downloaded
[ ] VirtualBox Extension Pack downloaded (same version as VirtualBox)
[ ] Windows Server 2022 ISO downloaded (~4.7 GB)
[ ] Windows 11 ISO downloaded (~6.5 GB)

Ready to proceed
[ ] All four files are saved in an easy-to-find location (e.g. C:\LabFiles\)
[ ] Charger plugged in if on a laptop
```

---

## 🗂️ Suggested Folder Structure on Your Host PC

Keep all your lab files organised in one place from the start:

```
C:\LabFiles\
├── ISOs\
│   ├── SERVER_EVAL_x64FRE_en-us.iso
│   └── Win11_25H2_EnglishInternational_x64_v2.iso
├── VMs\
│   ├── DC01\
│   └── WS01\
└── Scripts\
    └── New-BulkADUsers.ps1
```

---

## ➡️ Next Step

Once every box in the checklist is ticked you are ready to move on.

**[02 - Install Oracle VirtualBox →](../02%20-%20Install%20Oracle%20Virtual%20Box/README.md)**

---

*Part of the [Active Directory Home Lab](../README.md) project*
