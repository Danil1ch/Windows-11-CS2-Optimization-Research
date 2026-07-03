# PowerShell / CMD / Registry Commands — Windows 11 + CS2 Optimization

**Purpose:** a separate command reference from the Windows 11 25H2 + CS2 microstutter investigation.  
Each command includes what it does, when to use it, whether it is optional/risky, and rollback notes where possible.

**Test system:** Ryzen 7 5700X · RTX 3070 Ti · MSI MPG B550 Gaming Plus · Windows 11 Pro 25H2 build 26200 · CS2 · dual monitors 200 Hz + 100 Hz · ATTACK SHARK X3 wireless mouse.

> Do **not** run everything blindly.  
> For most users, the safest parts are: **Diagnostics**, **MPO fix**, **Defender exclusions**, **power checks**, and **measurement tools**.  
> **UAC disable**, **full Defender disable**, mass UWP removal and aggressive Windows debloat are **optional / risky / personal preference**.

---

## 0. How to run commands

Most commands require an elevated terminal:

```powershell
Win + X → Windows Terminal (Admin)
```

CMD tools such as `reg`, `schtasks`, `DISM`, `vssadmin`, `bcdedit` also work from Windows Terminal.

After registry, services, MPO, HAGS, Defender, scheduled task, BIOS or driver changes, reboot before judging results.

---

## 1. Baseline diagnostics before tweaks

### 1.1. List power plans

```powershell
powercfg /L
```

**What it does:** lists all power plans and marks the active one with `*`.

**Why:** verify that the desired high-performance plan is active.

---

### 1.2. Check CPU power settings in the active plan

```powershell
powercfg /Q SCHEME_CURRENT SUB_PROCESSOR
```

**What it does:** prints processor power management settings.

**What to check:**
- Minimum processor state;
- Maximum processor state.

**In this investigation:** AC power was 100% / 100%.

---

### 1.3. Check Core Parking parameters

```powershell
powercfg /Q SCHEME_CURRENT SUB_PROCESSOR CPMINCORES
powercfg /Q SCHEME_CURRENT SUB_PROCESSOR CPMAXCORES
powercfg /Q | findstr /i "CPMINCORES CPMAXCORES"
```

**What it does:** attempts to find explicit CPU core parking settings.

**Result here:** no useful `CPMINCORES/CPMAXCORES` entries were exposed, so QuickCPU/ParkControl were not used blindly.

---

### 1.4. Check VBS / Device Guard

```powershell
Get-CimInstance -Namespace root\Microsoft\Windows\DeviceGuard -ClassName Win32_DeviceGuard
```

Full output:

```powershell
Get-CimInstance -Namespace root\Microsoft\Windows\DeviceGuard -ClassName Win32_DeviceGuard | Format-List *
```

**What it does:** checks Virtualization-Based Security, Credential Guard, HVCI / Memory Integrity and related security features.

**Fields to inspect:**
- `VirtualizationBasedSecurityStatus`;
- `SecurityServicesRunning`;
- `SecurityServicesConfigured`.

---

### 1.5. Check Windows boot configuration

```powershell
bcdedit /enum
```

**What it does:** prints Windows Boot Manager / current OS loader settings.

**Why:** look for explicit Hyper-V or hypervisor launch parameters.

---

### 1.6. Check Hyper-V / virtualization optional components

```powershell
Get-WindowsOptionalFeature -Online | Where-Object {$_.FeatureName -match "Hyper|Virtual|Sandbox|Containers"}
```

**What it does:** shows the state of Hyper-V, VirtualMachinePlatform, HypervisorPlatform, Sandbox and Containers.

---

### 1.7. Inspect Windows driver store

```powershell
pnputil /enum-drivers
```

**What it does:** lists installed OEM INF drivers.

**Why:** check for old AMD/NVIDIA/Realtek duplicates.

**Important:** old driver packages were not removed because the expected FPS gain is tiny and the risk of deleting a fallback driver is not worth it.

---

### 1.8. Check TRIM for SSDs

```cmd
fsutil behavior query DisableDeleteNotify
```

**Normal result:**

```text
NTFS DisableDeleteNotify = 0
ReFS DisableDeleteNotify = 0
```

`0` means TRIM is enabled.

---

## 2. Power and sleep

### 2.1. Create Ultimate Performance power plan

```powershell
powercfg -duplicatescheme e9a42b02-d5df-448d-aa00-03f14749eb61
```

**What it does:** creates the Ultimate Performance / Maximum Performance power plan.

---

### 2.2. Activate a power plan by GUID

Example from this system:

```powershell
powercfg /setactive 4635cd9f-a6b8-45cf-bc4d-e5f34042c528
```

Alternative for the built-in High Performance alias:

```powershell
powercfg /setactive SCHEME_MIN
```

---

### 2.3. Disable hibernation and Fast Startup

```powershell
powercfg -h off
```

**What it does:** disables hibernation and removes `hiberfil.sys`.

**Pros:**
- frees disk space;
- disables Fast Startup;
- forces cleaner driver initialization after shutdown.

**Cons:**
- hibernation is unavailable;
- Fast Startup is unavailable.

**Rollback:**

```powershell
powercfg -h on
```

---

### 2.4. Check available sleep states

```powershell
powercfg /a
```

---

### 2.5. Disable wake by Realtek network adapter

```powershell
powercfg /devicedisablewake "Realtek PCIe GbE Family Controller"
```

**What it does:** prevents the Realtek NIC from waking the PC.

---

### 2.6. Check devices allowed to wake the PC

```powershell
powercfg /devicequery wake_armed
```

---

### 2.7. Check devices that can be configured for wake

```powershell
powercfg /devicequery wake_programmable
```

---

### 2.8. Check active sleep blockers

```powershell
powercfg /requests
```

---

### 2.9. Disable USB Selective Suspend

Reliable GUI path:

```text
Control Panel → Power Options → Change plan settings → Change advanced power settings → USB settings → USB selective suspend setting → Disabled
```

Check through CLI:

```powershell
powercfg /Q SCHEME_CURRENT SUB_USB
```

---

## 3. Disk cleanup and component store

### 3.1. Check Reserved Storage

```powershell
DISM /Online /Get-ReservedStorageState
```

---

### 3.2. Disable Reserved Storage

```powershell
DISM /Online /Set-ReservedStorageState /State:Disabled
```

**What it does:** disables Windows Reserved Storage.

**Result here:** around 8 GB freed.

**Rollback:**

```powershell
DISM /Online /Set-ReservedStorageState /State:Enabled
```

---

### 3.3. Delete all shadow copies

```cmd
vssadmin delete shadows /all
```

**What it does:** deletes all existing shadow copies / restore shadow data.

**Warning:** old restore points and shadow copies are lost.

---

### 3.4. Disable System Restore on C:

```powershell
Disable-ComputerRestore -Drive "C:\"
```

**Rollback:**

```powershell
Enable-ComputerRestore -Drive "C:\"
```

---

### 3.5. List restore points

```powershell
Get-ComputerRestorePoint
```

---

### 3.6. Analyze component store

```cmd
Dism /Online /Cleanup-Image /AnalyzeComponentStore
```

---

### 3.7. Clean WinSxS / component store

```cmd
DISM /Online /Cleanup-Image /StartComponentCleanup
```

**Note:** if you get `0x800f0806`, reboot and run it again.

---

### 3.8. Check CompactOS

```cmd
compact /compactos:query
```

**What it does:** checks whether Windows is compressed through CompactOS.

---

## 4. OneDrive

### 4.1. Check OneDrive presence

```powershell
Get-Command OneDrive*
where.exe OneDrive.exe
```

---

### 4.2. Disable OneDrive file sync by policy

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\OneDrive" /v DisableFileSyncNGSC /t REG_DWORD /d 1 /f
```

**Rollback:**

```cmd
reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows\OneDrive" /v DisableFileSyncNGSC /f
```

---

### 4.3. Remove OneDriveSetup from current user startup

```cmd
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v OneDriveSetup /f
```

---

## 5. Edge / EdgeUpdate / startup

### 5.1. Stop Edge Update services

```powershell
Stop-Service edgeupdate -ErrorAction SilentlyContinue
Stop-Service edgeupdatem -ErrorAction SilentlyContinue
```

---

### 5.2. Disable Edge Update services

```powershell
Set-Service edgeupdate -StartupType Disabled -ErrorAction SilentlyContinue
Set-Service edgeupdatem -StartupType Disabled -ErrorAction SilentlyContinue
```

**Rollback:**

```powershell
Set-Service edgeupdate -StartupType Manual -ErrorAction SilentlyContinue
Set-Service edgeupdatem -StartupType Manual -ErrorAction SilentlyContinue
```

---

### 5.3. Remove Microsoft Edge desktop shortcuts

```powershell
Remove-Item "$env:PUBLIC\Desktop\Microsoft Edge.lnk" -Force -ErrorAction SilentlyContinue
Remove-Item "$env:USERPROFILE\Desktop\Microsoft Edge.lnk" -Force -ErrorAction SilentlyContinue
```

---

### 5.4. Check Run startup keys

```cmd
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Run"
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Run"
reg query "HKLM\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Run"
```

---

### 5.5. Remove Edge AutoLaunch from startup

```cmd
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v MicrosoftEdgeAutoLaunch_D7AF3B185A21C1505ABE7863144C14FF /f
```

---

### 5.6. Find Edge scheduled tasks

```powershell
Get-ScheduledTask | Where-Object {$_.TaskName -like "*Edge*" -or $_.TaskPath -like "*Edge*"} | Select TaskName,TaskPath,State
```

---

### 5.7. Disable EdgeUpdate tasks

```powershell
Disable-ScheduledTask -TaskName "MicrosoftEdgeUpdateTaskMachineCore{C3347F68-E4A6-4C80-B2C5-7E5E9803EFBE}"
Disable-ScheduledTask -TaskName "MicrosoftEdgeUpdateTaskMachineUA{450E9791-858D-4398-95DB-8CFA62EB5365}"
```

---

### 5.8. Kill MicrosoftEdgeUpdate.exe

```cmd
taskkill /F /IM MicrosoftEdgeUpdate.exe
```

---

### 5.9. Check EdgeUpdate folder

```powershell
dir "C:\Program Files (x86)\Microsoft\EdgeUpdate"
```

---

## 6. Startup items

### 6.1. Show startup items

```powershell
Get-CimInstance Win32_StartupCommand | Select-Object Name, Command
```

---

### 6.2. Remove SecurityHealth from startup

```cmd
reg delete "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v SecurityHealth /f
```

**Important:** this does not fully disable Defender.

---

### 6.3. Remove OneDriveSetup from startup

```cmd
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v OneDriveSetup /f
```

---

## 7. Telemetry / privacy / consumer features

### 7.1. Disable Advertising ID

```cmd
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\AdvertisingInfo" /v Enabled /t REG_DWORD /d 0 /f
```

---

### 7.2. Disable Activity History upload

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\System" /v PublishUserActivities /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\System" /v UploadUserActivities /t REG_DWORD /d 0 /f
```

---

### 7.3. Disable Windows Consumer Features

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\CloudContent" /v DisableWindowsConsumerFeatures /t REG_DWORD /d 1 /f
```

---

### 7.4. Disable Tailored Experiences

```cmd
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Privacy" /v TailoredExperiencesWithDiagnosticDataEnabled /t REG_DWORD /d 0 /f
```

---

### 7.5. Limit Data Collection

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v AllowTelemetry /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v DisableTelemetryOptInSettingsUx /t REG_DWORD /d 1 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v DoNotShowFeedbackNotifications /t REG_DWORD /d 1 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v DisableOneSettingsDownloads /t REG_DWORD /d 1 /f
```

---

### 7.6. Disable DiagTrack and dmwappushservice

```powershell
Stop-Service DiagTrack -ErrorAction SilentlyContinue
Set-Service DiagTrack -StartupType Disabled -ErrorAction SilentlyContinue

Stop-Service dmwappushservice -ErrorAction SilentlyContinue
Set-Service dmwappushservice -StartupType Disabled -ErrorAction SilentlyContinue
```

---

## 8. Windows services disabled on this system

> Do not blindly disable all of these on other PCs.

### 8.1. Windows Search / Indexer

```powershell
Stop-Service WSearch
Set-Service WSearch -StartupType Disabled
```

**Downside:** file search can become slower.

---

### 8.2. Print Spooler

```powershell
Stop-Service Spooler
Set-Service Spooler -StartupType Disabled
```

**Do not disable** if you need printers or PDF printers.

---

### 8.3. PrintDeviceConfigurationService

```powershell
Stop-Service PrintDeviceConfigurationService
Set-Service PrintDeviceConfigurationService -StartupType Disabled
```

---

### 8.4. Remote Registry

```powershell
Stop-Service RemoteRegistry
Set-Service RemoteRegistry -StartupType Disabled
```

---

### 8.5. RetailDemo

```powershell
Stop-Service RetailDemo
Set-Service RetailDemo -StartupType Disabled
```

---

### 8.6. SysMain

```powershell
Stop-Service SysMain
Set-Service SysMain -StartupType Disabled
```

**Comment:** may be fine on SSDs, but do not expect a direct FPS boost.

---

### 8.7. Geolocation Service

```powershell
Stop-Service lfsvc
Set-Service lfsvc -StartupType Disabled
```

---

### 8.8. Data Usage Service

```powershell
Stop-Service DusmSvc
Set-Service DusmSvc -StartupType Disabled
```

---

### 8.9. Radio Management Service

```powershell
Stop-Service RmSvc
Set-Service RmSvc -StartupType Disabled
```

---

### 8.10. Hyper-V Host Compute Service

```powershell
Stop-Service HvHost
Set-Service HvHost -StartupType Disabled
```

**Do not disable** if you need Hyper-V, WSL2 or virtual machines.

---

### 8.11. InventorySvc

```powershell
Stop-Service InventorySvc
Set-Service InventorySvc -StartupType Disabled
```

---

### 8.12. MapsBroker

```powershell
Stop-Service MapsBroker
Set-Service MapsBroker -StartupType Disabled
```

---

### 8.13. CDPSvc

```powershell
Stop-Service CDPSvc
Set-Service CDPSvc -StartupType Disabled
```

**What it affects:** Connected Devices Platform, Nearby Share, some phone/PC linking features.

---

### 8.14. CDPUserSvc

```powershell
Get-Service CDPUserSvc* | Stop-Service
```

Attempted but not recommended as required:

```powershell
Get-Service CDPUserSvc* | Set-Service -StartupType Disabled
```

**Result here:** invalid parameter error.

---

### 8.15. WSAIFabricSvc

```powershell
Stop-Service WSAIFabricSvc
Set-Service WSAIFabricSvc -StartupType Disabled
```

---

### 8.16. Delivery Optimization / DoSvc

Attempted:

```powershell
Stop-Service DoSvc
Set-Service DoSvc -StartupType Disabled
```

**Result here:** access denied. Do not present it as a required working tweak.

---

### 8.17. HappService — only if Happ is installed and VPN runs on router

```powershell
Stop-Service HappService -ErrorAction SilentlyContinue
Set-Service HappService -StartupType Manual -ErrorAction SilentlyContinue
```

**What it does:** switches Happ Proxy Client Service to manual startup.

**Do not apply** if you rely on Happ all the time.

---

## 9. Scheduled tasks

### 9.1. CEIP / Feedback

```cmd
schtasks /Change /TN "\Microsoft\Windows\Customer Experience Improvement Program\Consolidator" /Disable
schtasks /Change /TN "\Microsoft\Windows\Customer Experience Improvement Program\UsbCeip" /Disable
schtasks /Change /TN "\Microsoft\Windows\Feedback\Siuf\DmClient" /Disable
schtasks /Change /TN "\Microsoft\Windows\Feedback\Siuf\DmClientOnScenarioDownload" /Disable
```

---

### 9.2. StartupAppTask

```cmd
schtasks /Change /TN "\Microsoft\Windows\Application Experience\StartupAppTask" /Disable
```

---

### 9.3. Microsoft Compatibility Appraiser Exp

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Application Experience\" -TaskName "Microsoft Compatibility Appraiser Exp"
```

---

### 9.4. WinSAT

```cmd
schtasks /Change /TN "\Microsoft\Windows\Maintenance\WinSAT" /Disable
```

---

### 9.5. Family Safety

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Shell\" -TaskName "FamilySafetyMonitor"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Shell\" -TaskName "FamilySafetyRefreshTask"
```

---

### 9.6. MapsToastTask

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Maps\" -TaskName "MapsToastTask"
```

---

### 9.7. RetailDemo cleanup

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\RetailDemo\" -TaskName "CleanupOfflineContent"
```

---

### 9.8. SpeechModelDownloadTask

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Speech\" -TaskName "SpeechModelDownloadTask"
```

---

### 9.9. Flighting / A-B testing tasks

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Flighting\FeatureConfig\" -TaskName "BootstrapUsageDataReporting"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Flighting\FeatureConfig\" -TaskName "GovernedFeatureUsageProcessing"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Flighting\FeatureConfig\" -TaskName "ReconcileConfigs"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Flighting\FeatureConfig\" -TaskName "ReconcileFeatures"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Flighting\FeatureConfig\" -TaskName "SafeguardsReconciliation"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Flighting\FeatureConfig\" -TaskName "UsageDataFlushing"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Flighting\FeatureConfig\" -TaskName "UsageDataReceiver"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Flighting\FeatureConfig\" -TaskName "UsageDataReporting"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Flighting\OneSettings\" -TaskName "RefreshCache"
```

---

### 9.10. Sustainability

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Sustainability\" -TaskName "PowerGridForecastTask"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Sustainability\" -TaskName "SustainabilityTelemetry"
```

---

### 9.11. AccountHealth

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\AccountHealth\" -TaskName "RecoverabilityToastTask"
```

---

### 9.12. WindowsAI tasks — attempted, access denied

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\WindowsAI\ClickToDo\" -TaskName "ModelCachingIdle"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\WindowsAI\ClickToDo\" -TaskName "ModelCachingLimit"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\WindowsAI\ClickToDo\" -TaskName "ModelCachingUpdate"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\WindowsAI\Recall\" -TaskName "PolicyConfiguration"
```

**Result here:** access denied. Do not list as required.

---

## 10. UWP / Appx removal

> Optional debloat. Not required for CS2 smoothness. Removing Microsoft Store is not recommended for everyone.

### 10.1. Remove Microsoft Store

```powershell
Get-AppxPackage *WindowsStore* | Remove-AppxPackage
```

### 10.2. Remove Web Experience

```powershell
Get-AppxPackage *WebExperience* | Remove-AppxPackage
```

### 10.3. Remove Phone Link

```powershell
Get-AppxPackage Microsoft.YourPhone | Remove-AppxPackage
```

### 10.4. Remove Outlook for Windows

```powershell
Get-AppxPackage Microsoft.OutlookForWindows | Remove-AppxPackage
```

### 10.5. Remove Clipchamp

```powershell
Get-AppxPackage Clipchamp.Clipchamp | Remove-AppxPackage
```

### 10.6. Remove Dev Home

```powershell
Get-AppxPackage Microsoft.Windows.DevHome | Remove-AppxPackage
```

### 10.7. Remove Get Help

```powershell
Get-AppxPackage Microsoft.GetHelp | Remove-AppxPackage
```

### 10.8. Remove Bing Search

```powershell
Get-AppxPackage Microsoft.BingSearch | Remove-AppxPackage
```

### 10.9. Remove Power Automate Desktop

```powershell
Get-AppxPackage Microsoft.PowerAutomateDesktop | Remove-AppxPackage
```

### 10.10. Remove CrossDevice

```powershell
Get-AppxPackage MicrosoftWindows.CrossDevice | Remove-AppxPackage
```

### 10.11. Xbox Appx — partial

```powershell
Get-AppxPackage *Xbox* | Remove-AppxPackage
```

**Note:** `Microsoft.XboxGameCallableUI` is a system component and may not be removed.

### 10.12. NarratorQuickStart — do not list as working

```powershell
Get-AppxPackage Microsoft.Windows.NarratorQuickStart | Remove-AppxPackage
```

**Result here:** system app removal error.

---

## 11. Windows Optional Features / Capabilities

### 11.1. Disable Work Folders

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName WorkFolders-Client -NoRestart
```

### 11.2. Disable Internet Printing Client

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName Printing-Foundation-InternetPrinting-Client -NoRestart
```

### 11.3. Disable WCF TCP Port Sharing

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName WCF-TCP-PortSharing45 -NoRestart
```

### 11.4. Remove Hello Face

```powershell
Remove-WindowsCapability -Online -Name Hello.Face.20134~~~~0.0.1.0
```

**Do not remove** if you use Windows Hello Face.

### 11.5. Remove Russian handwriting input

```powershell
Remove-WindowsCapability -Online -Name Language.Handwriting~~~ru-RU~0.0.1.0
```

### 11.6. Remove Math Recognizer

```powershell
Remove-WindowsCapability -Online -Name MathRecognizer~~~~0.0.1.0
```

### 11.7. Remove Steps Recorder

```powershell
Remove-WindowsCapability -Online -Name App.StepsRecorder~~~~0.0.1.0
```

---

## 12. Defender — safer option: exclusions

> Recommended for most users: do not destroy Defender, add exclusions for Steam/games/projects instead.

### 12.1. Add Steam folder exclusion

```powershell
Add-MpPreference -ExclusionPath "C:\Program Files (x86)\Steam"
```

### 12.2. Add SteamLibrary exclusion

```powershell
Add-MpPreference -ExclusionPath "D:\SteamLibrary"
```

Replace the path with your actual Steam library.

### 12.3. Add projects folder exclusion

```powershell
Add-MpPreference -ExclusionPath "$env:USERPROFILE\Desktop\projects"
```

Replace the path with your actual projects folder.

### 12.4. Check Defender exclusions

```powershell
Get-MpPreference | Select-Object -ExpandProperty ExclusionPath
```

---

## 13. Defender — optional / risky

> Not for everyone. Full Defender disabling reduces security.  
> On modern Windows 11, Tamper Protection may revert some of this.

### 13.1. Manually disable Tamper Protection

```text
Windows Security → Virus & threat protection → Manage settings → Tamper Protection → Off
```

### 13.2. Disable real-time monitoring

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
```

### 13.3. Disable behavior monitoring

```powershell
Set-MpPreference -DisableBehaviorMonitoring $true
```

### 13.4. Disable IOAV protection

```powershell
Set-MpPreference -DisableIOAVProtection $true
```

### 13.5. Disable script scanning

```powershell
Set-MpPreference -DisableScriptScanning $true
```

### 13.6. Policy key: DisableAntiSpyware

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableAntiSpyware /t REG_DWORD /d 1 /f
```

**Rollback:**

```cmd
reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableAntiSpyware /f
```

### 13.7. Policy key: DisableRealtimeMonitoring

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection" /v DisableRealtimeMonitoring /t REG_DWORD /d 1 /f
```

**Rollback:**

```cmd
reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection" /v DisableRealtimeMonitoring /f
```

---

## 14. UAC — optional / risky

> Not required for CS2. Disabling UAC reduces security and can affect UWP/system behavior.

### 14.1. Disable UAC through EnableLUA

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v EnableLUA /t REG_DWORD /d 0 /f
```

**Requires reboot.**

**Rollback:**

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v EnableLUA /t REG_DWORD /d 1 /f
```

### 14.2. Disable admin consent prompt

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v ConsentPromptBehaviorAdmin /t REG_DWORD /d 0 /f
```

**Rollback to default-ish behavior:**

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v ConsentPromptBehaviorAdmin /t REG_DWORD /d 5 /f
```

### 14.3. Disable secure desktop for UAC prompts

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v PromptOnSecureDesktop /t REG_DWORD /d 0 /f
```

**Rollback:**

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v PromptOnSecureDesktop /t REG_DWORD /d 1 /f
```

---

## 15. MPO / DWM fix

> One of the main useful tweaks here.  
> It helped with CS2 on a 200 Hz main monitor while Chromium/Electron/Yandex Music was active on a 100 Hz second monitor.

### 15.1. Disable MPO / add DWM overrides

Create `disable_mpo.reg`:

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Dwm]
"OverlayTestMode"=dword:00000005
"OverlayMinFPS"=dword:00000000
```

Reboot after applying.

### 15.2. Restore MPO defaults

Create `restore_mpo.reg`:

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Dwm]
"OverlayTestMode"=-
"OverlayMinFPS"=-
```

### 15.3. Check MPO keys

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows\Dwm" /v OverlayTestMode
reg query "HKLM\SOFTWARE\Microsoft\Windows\Dwm" /v OverlayMinFPS
```

---

## 16. MSI Utility v3 / mouse USB controller

> MSI Utility v3 is a GUI tool, but PowerShell was used to identify the correct USB controller.

### 16.1. Find mouse devices and their parents

```powershell
Get-PnpDevice -Class Mouse | ForEach-Object {
    $mouse = $_
    $parent = (Get-PnpDeviceProperty -InstanceId $mouse.InstanceId -KeyName DEVPKEY_Device_Parent).Data
    [PSCustomObject]@{ Mouse = $mouse.FriendlyName; Parent = $parent }
}
```

### 16.2. Get parent of a specific device

```powershell
Get-PnpDeviceProperty -InstanceId "<exact InstanceId>" -KeyName DEVPKEY_Device_Parent
```

Repeat the chain:

```text
HID device → USB Composite Device → USB Root Hub → PCI USB Controller
```

### 16.3. Find exact InstanceId by VID/PID

```powershell
Get-PnpDevice | Where-Object {$_.InstanceId -like "USB\VID_XXXX&PID_XXXX\*"} | Select InstanceId
```

Replace `VID_XXXX&PID_XXXX`.

### 16.4. Check MSI / Interrupt Priority in registry

Look under:

```text
HKLM\SYSTEM\CurrentControlSet\Enum\<PNP_ID>\Device Parameters\Interrupt Management\
```

Check:
- `MSISupported`;
- `MessageNumberLimit`;
- `DevicePriority`.

### 16.5. What was set in MSI Utility v3

Through MSI Utility v3 GUI:

```text
NVIDIA GeForce RTX 3070 Ti → MSI enabled / High
correct AMD USB Controller for ATTACK SHARK X3 receiver → MSI enabled / High
```

Do not set everything to High.

---

## 17. HAGS / Game Mode / Windows graphics

### 17.1. Check HAGS

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" /v HwSchMode
```

Usually:

```text
HwSchMode=2 → HAGS enabled
HwSchMode=1 → HAGS disabled
```

### 17.2. Enable HAGS

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" /v HwSchMode /t REG_DWORD /d 2 /f
```

Reboot.

### 17.3. Disable HAGS for testing

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" /v HwSchMode /t REG_DWORD /d 1 /f
```

**Result here:** HAGS Off + BIOS Data Link Feature Exchange made FPS/DPC worse, so HAGS was restored to On.

### 17.4. Check Game Mode registry value

```cmd
reg query "HKCU\Software\Microsoft\GameBar" /v AutoGameModeEnabled
```

If missing, check GUI:

```text
Settings → Gaming → Game Mode
```

---

## 18. BIOS / UEFI — Data Link Feature Exchange

> Not PowerShell, but part of the investigation.

MSI B550 Gaming Plus path:

```text
BIOS → F7 Advanced Mode → SETTINGS → Advanced → PCI Subsystem Settings → Data Link Feature Exchange → Enabled
```

Tested combination:

```text
HAGS Off + Data Link Feature Exchange Enabled
```

**Result on this system:**
- FPS dropped;
- DPC latency got worse.

HAGS was restored to enabled. DLFE/DLF was not isolated in a clean standalone test, so it should be listed only as an experimental BIOS option, not a required tweak.

---

## 19. Realtek LAN power saving

GUI path:

```text
Device Manager → Network adapters → Realtek PCIe GbE Family Controller → Properties → Advanced
```

Set:

```text
Power Saving Mode: Off
Energy Efficient Ethernet / EEE: Off
Green Ethernet: Off, if present
Interrupt Moderation: keep On
```

**What it does:** disables NIC power saving that can add micro-jitter.

DNS is not touched.

---

## 20. NVIDIA / ShadowPlay / Instant Replay

Done through NVIDIA App GUI.

```text
Instant Replay: enabled
Replay length: 5 min → 3 min
Bitrate: 33 Mbps → 15 Mbps
Resolution: 1080p
FPS: 60
Recording location: HDD
```

**What it does:** reduces background recording load without killing Instant Replay.

---

## 21. CRU / monitor 180 Hz → 200 Hz

### 21.1. What was done

```text
CRU → select Redmi G24 → Detailed resolutions → add/change 1920x1080 @ 200 Hz → OK → restart64.exe
```

### 21.2. Important CRU files

```text
restart64.exe — restarts graphics driver
reset-all.exe — emergency reset of all custom modes
```

Keep `reset-all.exe` ready before monitor overclocking experiments.

### 21.3. What still should be verified

```text
TestUFO Frame Skipping
```

Purpose: confirm the monitor truly displays 200 Hz without skipping frames.

---

## 22. Steam icon fix / blank white shortcuts

### 22.1. Check `.url` association

```cmd
assoc .url
```

Normal:

```text
.url=InternetShortcut
```

### 22.2. Check InternetShortcut file type

```cmd
ftype InternetShortcut
```

### 22.3. Refresh Windows icons

```cmd
ie4uinit.exe -show
```

### 22.4. Check Steam icon folder

```cmd
dir "C:\Program Files (x86)\Steam\steam\games"
```

### 22.5. Inspect suspicious `.ico` as text

```powershell
Get-Content "C:\Program Files (x86)\Steam\steam\games\ad22805b0d2a5bebc8aabe2007dc9b36f2b50ee1.ico"
```

### 22.6. Inspect file bytes

```powershell
Format-Hex "C:\Program Files (x86)\Steam\steam\games\ad22805b0d2a5bebc8aabe2007dc9b36f2b50ee1.ico"
```

**Cause found:** the script downloaded an HTML `301 Moved Permanently` page instead of an `.ico`.

**Fix:** use `curl -L` in the BAT script to follow redirects.

---

## 23. Commands not recommended as required

### 23.1. DoSvc disable

```powershell
Set-Service DoSvc -StartupType Disabled
```

Result here: access denied.

### 23.2. WindowsAI scheduled tasks

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\WindowsAI\ClickToDo\" -TaskName "ModelCachingIdle"
```

Result here: access denied.

### 23.3. CDPUserSvc disable

```powershell
Get-Service CDPUserSvc* | Set-Service -StartupType Disabled
```

Result here: invalid parameter.

### 23.4. NarratorQuickStart removal

```powershell
Get-AppxPackage Microsoft.Windows.NarratorQuickStart | Remove-AppxPackage
```

Result here: system app removal error.

---

## 24. Minimal safe set for most users

If you do not want to break Windows, start here:

```powershell
# Install AMD chipset and NVIDIA drivers manually from official sources.

# Defender exclusions
Add-MpPreference -ExclusionPath "C:\Program Files (x86)\Steam"
Add-MpPreference -ExclusionPath "D:\SteamLibrary"

# Power checks
powercfg /L
powercfg /Q SCHEME_CURRENT SUB_PROCESSOR

# VBS / HAGS checks
Get-CimInstance -Namespace root\Microsoft\Windows\DeviceGuard -ClassName Win32_DeviceGuard
reg query "HKLM\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" /v HwSchMode

# MPO fix only if you have dual-monitor stutter.
# MouseTester / receiver placement only if mouse movement feels inconsistent.
```

---

## 25. Main practical takeaways

The most useful changes were:

1. **MPO fix** for mixed-refresh dual monitors.
2. **MSI Utility v3 High priority** for GPU and the correct USB controller.
3. Moving the **ATTACK SHARK X3 receiver** to the desk.
4. **ShadowPlay / Instant Replay tuning**, not full disabling.
5. **Defender exclusions**, not a Defender war.
6. **Realtek LAN power saving off**.
7. **CapFrameX + LatencyMon + MouseTester**, not “it feels smoother”.

Average FPS barely changed. Smoothness improved because 1%/0.2% lows, frametime stability and DPC behavior improved.
