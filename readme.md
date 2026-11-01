# Silently Deploying VMware Tools via Microsoft Intune (Win32 App)

## 📌 Objective
Demonstrate how to package, upload, and silently deploy a legacy executable to a Windows 11 client using Microsoft Intune, effectively bypassing User Account Control (UAC) prompts for standard users on an Entra-joined device.

## 🏗️ Environment & Prerequisites
* **Cloud:** Microsoft 365 Business Premium tenant (Entra ID P1 + Intune Plan 1)
* **Endpoint:** Windows 11 Pro Virtual Machine (Entra-joined)
* **Tools:** Microsoft Win32 Content Prep Tool (`IntuneWinAppUtil.exe`)
* **Payload:** VMware Tools (`setup.exe`)

---

## 🛠️ Step 1: Packaging the Payload
To deploy an `.exe` via Intune, it must first be wrapped in an `.intunewin` container.
1. Extracted the `windows.iso` payload from the host machine into a staging folder `C:\IntunePack\Source`. Ensure the `Program Files` dependency folders are included alongside `setup.exe`.
2. Ran the Microsoft Win32 Content Prep Tool to compress the directory into `setup.intunewin`.

*[Insert Screenshot of the Command Prompt showing IntuneWinAppUtil.exe completing the package]*

---

## ☁️ Step 2: Configuring the Intune Win32 App
Uploaded the `.intunewin` package to the Microsoft Intune admin center and configured the installation behavior.

**Installation Commands:**
* **Install command:** `setup.exe /S /v"/qn REBOOT=R"`
  * *Why:* `/S` suppresses the InstallShield wrapper. `/v"/qn REBOOT=R"` passes silent mode and suppresses the reboot to the underlying MSI.
* **Uninstall command:** `setup.exe /S /v"/qn REMOVE=ALL"`
* **Install behavior:** `System` 
  * *Impact:* This is critical. Running as System bypasses the UAC prompt that would normally block standard users from installing software.

*[Insert image_10c4dd.png here - showing the Program configuration tab]*

---

## 🔍 Step 3: Setting Detection Rules
Intune requires a detection rule to verify the installation succeeded and prevent continuous installation loops. 
* **Rule type:** File
* **Path:** `C:\Program Files\VMware\VMware Tools`
* **File:** `vmtoolsd.exe`
* **Detection method:** File or folder exists

---

## 🚀 Step 4: Deployment & Triage
Assigned the application to the test Security Group. 

**Verification:**
Instead of waiting for the scheduled Intune check-in, I forced a manual sync. Standard users cannot force a sync via the Windows Settings GUI, so I restarted the VM to trigger the Intune Management Extension service on boot.

*[Insert image_0ff2fe.png here - showing the Intune portal "Waiting for install status"]*

To verify the background installation locally while the Intune portal was catching up, I monitored the local Intune logs:
* **Log Path:** `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\IntuneManagementExtension.log`

**Result:** The application installed silently in the background, populating the local directory and enabling host-to-guest clipboard sharing without requiring any user interaction.
