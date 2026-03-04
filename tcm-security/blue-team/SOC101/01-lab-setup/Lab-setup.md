# Windows 11 Virtual Lab Setup (VirtualBox)

## 1. Download VirtualBox

- Go to: https://www.virtualbox.org/wiki/Downloads  
- Download the **Windows hosts** version  
- Install Oracle VM VirtualBox

---

## 2. Install Prerequisites

Install **Microsoft Visual C++ Redistributable** (required dependency).

---

## 3. Download Windows 11 ISO

- Visit:  
  https://www.microsoft.com/en-us/evalcenter/download-windows-11-enterprise  
- Download the Windows 11 Enterprise ISO file

---

## 4. Create the Virtual Machine

Open VirtualBox and create a new VM:

- Set the **machine name**
- Select the downloaded **ISO**
- ⚠️ Uncheck **"Unattended Installation"**
- Enable **UEFI**
- Assign resources:
  - Minimum 4GB RAM
  - Minimum 2 CPUs
- Create a virtual hard disk

---

## 5. Windows 11 Requirements Error

During installation, Windows may display a message stating that the system does not meet the minimum requirements (TPM, Secure Boot, etc.).

---

## 6. Bypass Windows 11 Requirements

When the error appears:

1. Press `Shift + F10`
2. Type `regedit`
3. Navigate to:   HKEY_LOCAL_MACHINE\SYSTEM\Setup
4. Create a new key named:  Inside `LabConfig`, create the following **DWORD (32-bit)** values and set them to `1`:


6. Close Registry Editor and continue the installation.
---

## 7. Disable Windows Defender (Lab Environment Only)

⚠️ Warning: Disabling Windows Defender reduces system security. Use only in isolated lab environments.

### Disable Tamper Protection

1. Open **Virus & Threat Protection**
2. Click **Manage Settings**
3. Turn off **Tamper Protection**

---

### PowerShell Commands (Run as Administrator)


```powershell
Disable real-time protection:
Set-MpPreference -DisableRealtimeMonitoring $true


Disable network file scanning:
Set-MpPreference -DisableScanningNetworkFiles $true

Disable Block At First Sight:
Set-MpPreference -DisableBlockAtFirstSeen $true

Disable Windows Defender via Registry:
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableAntiSpyware /t REG_DWORD /d 1 /f
