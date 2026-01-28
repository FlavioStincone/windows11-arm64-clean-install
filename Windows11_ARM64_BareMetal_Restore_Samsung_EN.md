# TECHNICAL GUIDE

## Clean Install Recovery of Windows 11 (ARM64)  
### Samsung Galaxy Book (Go / Edge / S)

**Subject:**  
Bare-metal recovery procedure for Samsung devices with **ARM64** architecture,  
in cases of **corrupted, replaced or fully formatted UFS storage**,  
where the standard Windows installer fails to complete the installation.

> **WARNING – ADVANCED GUIDE**  
> This procedure is intended exclusively for technicians or advanced users.  
> Requires familiarity with **DISM, WinPE, manual partitioning, and UEFI**.  
> Improper use can cause **total data loss**.

**Document version:** 2.3  
**Date:** 01/28/2026

---

## 1. PREREQUISITES AND TOOLS

A working support workstation (**Host PC**) with Windows is required.

### 1.1 Required Hardware

- **Host PC:** Windows 10/11 x64  
- **Target PC:** Samsung Galaxy Book ARM (Go, Edge, S, etc.)  
- **USB 1 – Boot (WinPE):** ≥ 8 GB *(will be formatted)*  
- **USB 2 – Data:** ≥ 16 GB, **NTFS**  
- **Recommended accessories:**
  - Wired USB mouse  
  - Android smartphone (USB tethering) or USB LAN adapter

---

### 1.2 Software to download on Host PC

1. **Windows ADK – Windows 11**  
   - Install the **Deployment Tools** component  
2. **Windows PE add-on for ADK**  
   - Install **after** the ADK  
3. **Official Windows 11 ARM64 ISO**  
   - Download from **Microsoft website** (section *Download Windows 11 on ARM64 devices*)

---

### 1.3 Driver acquisition (**CRITICAL STEP**)

Drivers are **not included** in the generic ARM64 ISO.  
They must be downloaded from **Samsung official support site**:

1. Visit https://www.samsung.com/support  
2. Search for the exact device model (e.g., `NP740XME-KA1IT`)  
3. Download the **Driver Pack** or **System Software Installer**  
4. Extract and organize drivers as follows:

    C:\Drivers_WinPE   -> Storage (UFS) / Chipset / USB (ESSENTIAL)  
    C:\Drivers         -> Complete driver package

Without **UFS / USB** drivers, the disk **will not be visible** in WinPE.

---

## 2. INITIALIZING THE ADK ENVIRONMENT (HOST PC)

Before using DISM, open an **Administrator Command Prompt** and run:

    cd "C:\Program Files (x86)\Windows Kits\10\Assessment and Deployment Kit\Deployment Tools"  
    DandISetEnv.bat

Verify the message:

    Environment variables have been set

---

## 3. WINDOWS IMAGE PREPARATION (DATA USB)

### 3.1 `install.wim`

1. Mount the ARM64 ISO  
2. Copy from `sources` → `install.wim` or `install.esd`  
3. If the file is `.esd`, convert it to `.wim`:

    dism /export-image ^  
     /sourceimagefile:C:\wim\install.esd ^  
     /sourceindex:1 ^  
     /destinationimagefile:C:\wim\install.wim ^  
     /compress:max /checkintegrity

---

### 3.2 Edition and index verification

    dism /get-wiminfo /wimfile:C:\wim\install.wim

Choose the index corresponding to the desired edition (e.g., Pro, Home, Education).

---

### 3.3 Driver injection

    md C:\wimmount  
    dism /mount-wim /wimfile:C:\wim\install.wim /index:1 /mountdir:C:\wimmount  
    dism /image:C:\wimmount /add-driver /driver:C:\Drivers /recurse /forceunsigned  
    dism /unmount-wim /mountdir:C:\wimmount /commit

Copy the modified `install.wim` file to the **root of USB 2**.

---

### 3.4 Automation scripts

#### `CreatePartitions.txt`

    select disk 0  
    clean  
    convert gpt  
    create partition efi size=260  
    format quick fs=fat32 label="System"  
    assign letter=S  
    create partition msr size=128  
    create partition primary  
    format quick fs=ntfs label="Windows"  
    assign letter=W

#### `ApplyImage.bat`

    @echo off  
    set INDEX=%2  
    if "%INDEX%"=="" set INDEX=1  
    echo [INFO] Applying Windows image...  
    dism /apply-image /imagefile:%1 /index:%INDEX% /applydir:W:\  
    echo [INFO] Creating UEFI bootloader...  
    W:\Windows\System32\bcdboot W:\Windows /s S:  
    echo [SUCCESS] Installation completed.  
    pause

---

## 4. CREATING WINPE USB (BOOT)

    Dism /Cleanup-Wim  
    rd /s /q C:\WinPE_arm64  

    copype arm64 C:\WinPE_arm64  

    dism /mount-image ^  
     /imagefile:"C:\WinPE_arm64\media\sources\boot.wim" ^  
     /index:1 ^  
     /mountdir:"C:\WinPE_arm64\mount"  

    dism /image:"C:\WinPE_arm64\mount" ^  
     /add-driver /driver:"C:\Drivers_WinPE" /recurse /forceunsigned  

    dism /unmount-image /mountdir:"C:\WinPE_arm64\mount" /commit  

    MakeWinPEMedia /UFD C:\WinPE_arm64 E:

Replace `E:` with the drive letter of **USB 1 (WinPE)**.

---

## 5. INSTALLATION ON TARGET DEVICE

### 5.1 UEFI / BIOS settings

- Disable **Secure Boot** and **Fast Boot**  
- Set boot priority to **USB (WinPE)**  
- Save with **F10**

---

### 5.2 Installation from WinPE

    diskpart /s D:\CreatePartitions.txt  
    D:\ApplyImage.bat D:\install.wim 1

At the end:

    exit

Remove USB sticks and reboot.

---

## 6. POST-INSTALLATION CONFIGURATION

### 6.1 Network bypass (Windows 11 only)

    Shift + F10  
    OOBE\BYPASSNRO

### 6.2 Complete recovery

- Connect to Internet (USB tethering recommended)  
- Update via **Windows Update**  
- Verify functionality of:  
  - UFS / eMMC Storage  
  - Touchpad and keyboard  
  - Sleep / standby

---

## END OF PROCEDURE

---

## DISCLAIMER AND LICENSE

This guide is provided "as is," without any warranty.  
The author is not responsible for any damage, data loss, or malfunctions.

Distributed under the **MIT License** (see `LICENSE` file in the repository).

---