# PowerShell / CMD / Registry Commands — Windows 11 + CS2 Optimization

**Назначение файла:** отдельный список команд из расследования Windows 11 25H2 + CS2: что команда делает, когда применять, что является опциональным/опасным и как откатить часть изменений.

**Система, под которую делалось:** Ryzen 7 5700X · RTX 3070 Ti · MSI MPG B550 Gaming Plus · Windows 11 Pro 25H2 build 26200 · CS2 · два монитора 200 Hz + 100 Hz · мышь ATTACK SHARK X3.

> Это не “запусти всё подряд”. Команды разделены по смыслу.  
> Для обычного пользователя безопаснее брать только разделы **Recommended / Диагностика / MPO / Defender exclusions**.  
> Разделы **UAC**, **полное отключение Defender**, массовое удаление UWP и агрессивный debloat — **по желанию и на свой риск**.

---

## 0. Как запускать

Большинство команд требуют терминал от администратора:

```powershell
Win + X → Терминал Windows (Администратор)
```

Для CMD-команд можно использовать тот же терминал, просто команды `reg`, `schtasks`, `DISM`, `vssadmin`, `bcdedit` работают и оттуда.

После изменений в реестре, службах, MPO, HAGS, Defender, планировщике и драйверах желательно делать перезагрузку.

---

## 1. Базовая диагностика перед твиками

### 1.1. Список схем питания

```powershell
powercfg /L
```

**Что делает:** показывает доступные схемы питания и активную схему со звёздочкой `*`.

**Зачем:** убедиться, что активна “Максимальная производительность” или другая нужная схема.

---

### 1.2. Проверить параметры процессора в активной схеме

```powershell
powercfg /Q SCHEME_CURRENT SUB_PROCESSOR
```

**Что делает:** выводит настройки управления питанием процессора.

**Что смотрели:**  
- Minimum processor state / Минимальное состояние процессора;
- Maximum processor state / Максимальное состояние процессора.

**У нас:** от сети было 100% / 100%.

---

### 1.3. Проверить Core Parking параметры

```powershell
powercfg /Q SCHEME_CURRENT SUB_PROCESSOR CPMINCORES
powercfg /Q SCHEME_CURRENT SUB_PROCESSOR CPMAXCORES
powercfg /Q | findstr /i "CPMINCORES CPMAXCORES"
```

**Что делает:** пытается найти параметры парковки ядер.

**Вывод:** явных параметров `CPMINCORES/CPMAXCORES` у нас не нашлось. QuickCPU/ParkControl сразу не ставили.

---

### 1.4. Проверить VBS / Device Guard

```powershell
Get-CimInstance -Namespace root\Microsoft\Windows\DeviceGuard -ClassName Win32_DeviceGuard
```

Расширенный вывод:

```powershell
Get-CimInstance -Namespace root\Microsoft\Windows\DeviceGuard -ClassName Win32_DeviceGuard | Format-List *
```

**Что делает:** показывает состояние VBS, Credential Guard, HVCI/Memory Integrity и связанных фич безопасности.

**Что смотрели:**  
- `VirtualizationBasedSecurityStatus`;
- `SecurityServicesRunning`;
- `SecurityServicesConfigured`.

**У нас:** VBS status показывал `2`, но `SecurityServicesRunning` был `{0}`.

---

### 1.5. Проверить BCD / загрузочные параметры Windows

```powershell
bcdedit /enum
```

**Что делает:** показывает параметры загрузчика Windows.

**Зачем:** проверить, есть ли явные параметры Hyper-V/hypervisorlaunchtype, isolatedcontext и т.п.

---

### 1.6. Проверить Hyper-V / виртуализацию в компонентах Windows

```powershell
Get-WindowsOptionalFeature -Online | Where-Object {$_.FeatureName -match "Hyper|Virtual|Sandbox|Containers"}
```

**Что делает:** показывает состояние Hyper-V, VirtualMachinePlatform, HypervisorPlatform, Sandbox, Containers.

**У нас:** Hyper-V/VirtualMachinePlatform были Disabled.

---

### 1.7. Проверить драйверный магазин Windows

```powershell
pnputil /enum-drivers
```

**Что делает:** выводит установленные OEM INF-драйверы.

**Зачем:** искали старые дубликаты AMD/NVIDIA/Realtek драйверов.

**Важно:** старые OEM-драйверы мы не удаляли, потому что FPS это почти не даёт, а риск сломать резервный драйвер есть.

---

### 1.8. Проверить TRIM для SSD

```cmd
fsutil behavior query DisableDeleteNotify
```

**Что делает:** проверяет, включён ли TRIM.

**Нормальный результат:**

```text
NTFS DisableDeleteNotify = 0
ReFS DisableDeleteNotify = 0
```

`0` = TRIM включён.

---

## 2. Питание и сон

### 2.1. Создать схему “Максимальная производительность”

```powershell
powercfg -duplicatescheme e9a42b02-d5df-448d-aa00-03f14749eb61
```

**Что делает:** создаёт схему Ultimate Performance / “Максимальная производительность”.

**После этого:** активировать созданную схему через GUI или командой с её GUID.

---

### 2.2. Активировать схему по GUID

Пример с нашим GUID:

```powershell
powercfg /setactive 4635cd9f-a6b8-45cf-bc4d-e5f34042c528
```

**Что делает:** делает выбранную схему активной.

**Альтернатива:** если есть стандартная схема High Performance:

```powershell
powercfg /setactive SCHEME_MIN
```

---

### 2.3. Отключить гибернацию и быстрый запуск

```powershell
powercfg -h off
```

**Что делает:** отключает гибернацию и удаляет `hiberfil.sys`.

**Плюсы:**  
- освобождает место на диске;
- отключает Fast Startup;
- Windows стартует чище, драйверы не подтягиваются из гибридного состояния.

**Минусы:**  
- пропадает гибернация;
- быстрый запуск Windows отключается.

**Откат:**

```powershell
powercfg -h on
```

---

### 2.4. Проверить доступные режимы сна

```powershell
powercfg /a
```

**Что делает:** показывает, какие режимы сна/гибернации доступны.

---

### 2.5. Отключить пробуждение ПК сетевой картой Realtek

```powershell
powercfg /devicedisablewake "Realtek PCIe GbE Family Controller"
```

**Что делает:** запрещает сетевой карте будить компьютер.

**Зачем:** чтобы Wake-on-LAN/сетевые события не трогали систему.

---

### 2.6. Проверить устройства, которым разрешено будить ПК

```powershell
powercfg /devicequery wake_armed
```

**Что делает:** показывает устройства, которые могут будить ПК.

---

### 2.7. Проверить устройства, которые вообще умеют wake

```powershell
powercfg /devicequery wake_programmable
```

**Что делает:** показывает устройства, для которых можно настраивать wake.

---

### 2.8. Проверить активные блокировки сна

```powershell
powercfg /requests
```

**Что делает:** показывает процессы/драйверы, которые мешают сну/выключению дисплея.

---

### 2.9. USB Selective Suspend — отключение

У нас через команду получилось не с первого раза, поэтому надёжный путь — GUI:

```text
Панель управления → Электропитание → Настройка схемы → Изменить дополнительные параметры питания → Параметры USB → Параметр временного отключения USB-порта → Запрещено
```

Проверка:

```powershell
powercfg /Q SCHEME_CURRENT SUB_USB
```

**Что делает:** проверяет настройки USB в текущей схеме питания.

---

## 3. Очистка диска и системного хранилища

### 3.1. Проверить Reserved Storage

```powershell
DISM /Online /Get-ReservedStorageState
```

**Что делает:** показывает, включено ли зарезервированное хранилище Windows.

---

### 3.2. Отключить Reserved Storage

```powershell
DISM /Online /Set-ReservedStorageState /State:Disabled
```

**Что делает:** отключает зарезервированное хранилище Windows.

**У нас:** освободило около 8 ГБ.

**Откат:**

```powershell
DISM /Online /Set-ReservedStorageState /State:Enabled
```

---

### 3.3. Удалить все теневые копии / Shadow Copies

```cmd
vssadmin delete shadows /all
```

**Что делает:** удаляет все точки теневого копирования/старые shadow copies.

**У нас:** освободило около 1 ГБ.

**Важно:** после этого старые точки восстановления будут потеряны.

---

### 3.4. Отключить System Restore для диска C:

```powershell
Disable-ComputerRestore -Drive "C:\"
```

**Что делает:** отключает восстановление системы на диске C:.

**Минус:** пропадают будущие точки восстановления для C:.

**Откат:**

```powershell
Enable-ComputerRestore -Drive "C:\"
```

---

### 3.5. Проверить точки восстановления

```powershell
Get-ComputerRestorePoint
```

**Что делает:** показывает доступные restore points.

---

### 3.6. Анализ WinSxS / Component Store

```cmd
Dism /Online /Cleanup-Image /AnalyzeComponentStore
```

**Что делает:** анализирует хранилище компонентов Windows.

**Что смотреть:**  
- Component Store size;
- Backup and Disabled Features;
- Recommended cleanup.

---

### 3.7. Очистить WinSxS / Component Store

```cmd
DISM /Online /Cleanup-Image /StartComponentCleanup
```

**Что делает:** удаляет старые версии компонентов Windows.

**У нас:** сначала была ошибка `0x800f0806` из-за отложенных операций, после перезагрузки прошло нормально.

---

### 3.8. Проверить CompactOS

```cmd
compact /compactos:query
```

**Что делает:** проверяет, сжата ли Windows через CompactOS.

**У нас:** Windows сказала, что сжатие не полезно. Не включали.

---

## 4. OneDrive

### 4.1. Проверить наличие OneDrive

```powershell
Get-Command OneDrive*
where.exe OneDrive.exe
```

**Что делает:** ищет OneDrive в системе.

---

### 4.2. Отключить синхронизацию OneDrive политикой

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\OneDrive" /v DisableFileSyncNGSC /t REG_DWORD /d 1 /f
```

**Что делает:** запрещает OneDrive File Sync через policy.

**Откат:**

```cmd
reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows\OneDrive" /v DisableFileSyncNGSC /f
```

---

### 4.3. Убрать OneDriveSetup из автозагрузки текущего пользователя

```cmd
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v OneDriveSetup /f
```

**Что делает:** удаляет OneDriveSetup из автозапуска пользователя.

---

## 5. Edge / EdgeUpdate / автозапуск

### 5.1. Остановить Edge Update службы

```powershell
Stop-Service edgeupdate -ErrorAction SilentlyContinue
Stop-Service edgeupdatem -ErrorAction SilentlyContinue
```

**Что делает:** останавливает службы обновления Microsoft Edge, если они есть.

---

### 5.2. Отключить Edge Update службы

```powershell
Set-Service edgeupdate -StartupType Disabled -ErrorAction SilentlyContinue
Set-Service edgeupdatem -StartupType Disabled -ErrorAction SilentlyContinue
```

**Что делает:** запрещает автозапуск служб Edge Update.

**Откат:**

```powershell
Set-Service edgeupdate -StartupType Manual -ErrorAction SilentlyContinue
Set-Service edgeupdatem -StartupType Manual -ErrorAction SilentlyContinue
```

---

### 5.3. Удалить ярлыки Microsoft Edge с рабочего стола

```powershell
Remove-Item "$env:PUBLIC\Desktop\Microsoft Edge.lnk" -Force -ErrorAction SilentlyContinue
Remove-Item "$env:USERPROFILE\Desktop\Microsoft Edge.lnk" -Force -ErrorAction SilentlyContinue
```

**Что делает:** удаляет ярлыки Edge с общего и пользовательского рабочего стола.

---

### 5.4. Проверить Run-автозагрузку

```cmd
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Run"
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Run"
reg query "HKLM\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Run"
```

**Что делает:** показывает автозапуск пользователя, системы и WOW6432Node.

---

### 5.5. Удалить Edge AutoLaunch из автозагрузки

```cmd
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v MicrosoftEdgeAutoLaunch_D7AF3B185A21C1505ABE7863144C14FF /f
```

**Что делает:** удаляет автозапуск Edge `--no-startup-window --win-session-start`.

---

### 5.6. Найти Edge scheduled tasks

```powershell
Get-ScheduledTask | Where-Object {$_.TaskName -like "*Edge*" -or $_.TaskPath -like "*Edge*"} | Select TaskName,TaskPath,State
```

**Что делает:** ищет задачи планировщика, связанные с Edge.

---

### 5.7. Отключить EdgeUpdate задачи

```powershell
Disable-ScheduledTask -TaskName "MicrosoftEdgeUpdateTaskMachineCore{C3347F68-E4A6-4C80-B2C5-7E5E9803EFBE}"
Disable-ScheduledTask -TaskName "MicrosoftEdgeUpdateTaskMachineUA{450E9791-858D-4398-95DB-8CFA62EB5365}"
```

**Что делает:** отключает задачи обновления Edge.

---

### 5.8. Убить процесс MicrosoftEdgeUpdate.exe

```cmd
taskkill /F /IM MicrosoftEdgeUpdate.exe
```

**Что делает:** принудительно завершает Edge updater, если задача зависла в Running.

---

### 5.9. Проверить папку EdgeUpdate

```powershell
dir "C:\Program Files (x86)\Microsoft\EdgeUpdate"
```

**Что делает:** показывает содержимое папки EdgeUpdate.

---

## 6. Автозагрузка

### 6.1. Посмотреть все startup items

```powershell
Get-CimInstance Win32_StartupCommand | Select-Object Name, Command
```

**Что делает:** показывает элементы автозагрузки из разных мест.

---

### 6.2. Убрать SecurityHealth из автозагрузки

```cmd
reg delete "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v SecurityHealth /f
```

**Что делает:** убирает иконку/процесс SecurityHealth из Run.

**Важно:** это не равно полному отключению Defender.

---

### 6.3. Убрать OneDriveSetup из автозагрузки

```cmd
reg delete "HKCU\Software\Microsoft\Windows\CurrentVersion\Run" /v OneDriveSetup /f
```

**Что делает:** убирает OneDriveSetup из автозапуска текущего пользователя.

---

## 7. Телеметрия / приватность / consumer features

### 7.1. Отключить Advertising ID

```cmd
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\AdvertisingInfo" /v Enabled /t REG_DWORD /d 0 /f
```

**Что делает:** отключает рекламный идентификатор Windows для текущего пользователя.

---

### 7.2. Отключить Activity History / Timeline upload

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\System" /v PublishUserActivities /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\System" /v UploadUserActivities /t REG_DWORD /d 0 /f
```

**Что делает:** запрещает публикацию/загрузку активности пользователя.

---

### 7.3. Отключить Windows Consumer Features

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\CloudContent" /v DisableWindowsConsumerFeatures /t REG_DWORD /d 1 /f
```

**Что делает:** отключает потребительские рекомендации/автоустановку части мусорных компонентов Windows.

---

### 7.4. Отключить Tailored Experiences

```cmd
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\Privacy" /v TailoredExperiencesWithDiagnosticDataEnabled /t REG_DWORD /d 0 /f
```

**Что делает:** отключает персонализированные советы на основе диагностических данных.

---

### 7.5. Ограничить Data Collection

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v AllowTelemetry /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v DisableTelemetryOptInSettingsUx /t REG_DWORD /d 1 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v DoNotShowFeedbackNotifications /t REG_DWORD /d 1 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v DisableOneSettingsDownloads /t REG_DWORD /d 1 /f
```

**Что делает:** урезает телеметрию, feedback-уведомления и OneSettings downloads.

---

### 7.6. Отключить DiagTrack и dmwappushservice

```powershell
Stop-Service DiagTrack -ErrorAction SilentlyContinue
Set-Service DiagTrack -StartupType Disabled -ErrorAction SilentlyContinue

Stop-Service dmwappushservice -ErrorAction SilentlyContinue
Set-Service dmwappushservice -StartupType Disabled -ErrorAction SilentlyContinue
```

**Что делает:** останавливает и отключает службы Connected User Experiences and Telemetry / dmwappushservice.

---

## 8. Службы Windows, которые отключались

> Не надо слепо отключать всё подряд на чужом ПК. Ниже — что делалось на конкретной системе.

### 8.1. Windows Search / индексатор

```powershell
Stop-Service WSearch
Set-Service WSearch -StartupType Disabled
```

**Что делает:** отключает индексатор поиска.

**Минус:** поиск по файлам может стать медленнее.

---

### 8.2. Print Spooler

```powershell
Stop-Service Spooler
Set-Service Spooler -StartupType Disabled
```

**Что делает:** отключает диспетчер печати.

**Минус:** принтеры/PDF-принтеры могут не работать.

**Не отключать**, если нужен принтер.

---

### 8.3. PrintDeviceConfigurationService

```powershell
Stop-Service PrintDeviceConfigurationService
Set-Service PrintDeviceConfigurationService -StartupType Disabled
```

**Что делает:** отключает службу настройки принтеров.

---

### 8.4. Remote Registry

```powershell
Stop-Service RemoteRegistry
Set-Service RemoteRegistry -StartupType Disabled
```

**Что делает:** отключает удалённый доступ к реестру.

---

### 8.5. RetailDemo

```powershell
Stop-Service RetailDemo
Set-Service RetailDemo -StartupType Disabled
```

**Что делает:** отключает демо-режим магазина.

---

### 8.6. SysMain

```powershell
Stop-Service SysMain
Set-Service SysMain -StartupType Disabled
```

**Что делает:** отключает SysMain/Superfetch.

**Комментарий:** на SSD часто можно отключить, но прямого FPS-буста ждать не надо.

---

### 8.7. Geolocation Service

```powershell
Stop-Service lfsvc
Set-Service lfsvc -StartupType Disabled
```

**Что делает:** отключает геолокацию Windows.

---

### 8.8. Data Usage Service

```powershell
Stop-Service DusmSvc
Set-Service DusmSvc -StartupType Disabled
```

**Что делает:** отключает учёт использования данных.

---

### 8.9. Radio Management Service

```powershell
Stop-Service RmSvc
Set-Service RmSvc -StartupType Disabled
```

**Что делает:** отключает службу управления радио-модулями.

**У нас:** остановка дала ошибку, но `StartupType` выставился Disabled.

---

### 8.10. Hyper-V Host Compute Service

```powershell
Stop-Service HvHost
Set-Service HvHost -StartupType Disabled
```

**Что делает:** отключает Hyper-V host service.

**Не отключать**, если нужны Hyper-V/WSL2/виртуалки.

---

### 8.11. InventorySvc

```powershell
Stop-Service InventorySvc
Set-Service InventorySvc -StartupType Disabled
```

**Что делает:** отключает службу инвентаризации и оценки совместимости.

---

### 8.12. MapsBroker

```powershell
Stop-Service MapsBroker
Set-Service MapsBroker -StartupType Disabled
```

**Что делает:** отключает офлайн-карты Windows.

---

### 8.13. CDPSvc

```powershell
Stop-Service CDPSvc
Set-Service CDPSvc -StartupType Disabled
```

**Что делает:** отключает Connected Devices Platform: связка устройств, Nearby Share, часть функций телефона/ПК.

---

### 8.14. CDPUserSvc

```powershell
Get-Service CDPUserSvc* | Stop-Service
```

**Что делает:** останавливает user-service Connected Devices Platform.

Попытка отключить полностью:

```powershell
Get-Service CDPUserSvc* | Set-Service -StartupType Disabled
```

**У нас:** дала ошибку `Параметр задан неверно`, поэтому как обязательную команду не использовать.

---

### 8.15. WSAIFabricSvc

```powershell
Stop-Service WSAIFabricSvc
Set-Service WSAIFabricSvc -StartupType Disabled
```

**Что делает:** отключает службу Windows AI Fabric.

---

### 8.16. Delivery Optimization / DoSvc

Пытались:

```powershell
Stop-Service DoSvc
Set-Service DoSvc -StartupType Disabled
```

**У нас:** `Set-Service` дал `Отказано в доступе`.

**Итог:** не включать в обязательный список как рабочую команду.

---

### 8.17. HappService — если Happ установлен, но VPN уже на роутере

```powershell
Stop-Service HappService -ErrorAction SilentlyContinue
Set-Service HappService -StartupType Manual -ErrorAction SilentlyContinue
```

**Что делает:** переводит Happ Proxy Client Service в ручной запуск.

**Зачем:** если VPN уже на роутере/OpenWrt, Happ не обязан висеть службой в фоне.

**Не применять**, если Happ нужен постоянно.

---

## 9. Планировщик задач

### 9.1. CEIP / Feedback

```cmd
schtasks /Change /TN "\Microsoft\Windows\Customer Experience Improvement Program\Consolidator" /Disable
schtasks /Change /TN "\Microsoft\Windows\Customer Experience Improvement Program\UsbCeip" /Disable
schtasks /Change /TN "\Microsoft\Windows\Feedback\Siuf\DmClient" /Disable
schtasks /Change /TN "\Microsoft\Windows\Feedback\Siuf\DmClientOnScenarioDownload" /Disable
```

**Что делает:** отключает задачи Customer Experience Improvement Program и Feedback.

---

### 9.2. StartupAppTask

```cmd
schtasks /Change /TN "\Microsoft\Windows\Application Experience\StartupAppTask" /Disable
```

**Что делает:** отключает задачу анализа startup-приложений.

---

### 9.3. Microsoft Compatibility Appraiser Exp

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Application Experience\" -TaskName "Microsoft Compatibility Appraiser Exp"
```

**Что делает:** отключает новую задачу оценки совместимости Windows.

---

### 9.4. WinSAT

```cmd
schtasks /Change /TN "\Microsoft\Windows\Maintenance\WinSAT" /Disable
```

**Что делает:** отключает периодический Windows System Assessment Tool.

---

### 9.5. Family Safety

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Shell\" -TaskName "FamilySafetyMonitor"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Shell\" -TaskName "FamilySafetyRefreshTask"
```

**Что делает:** отключает Family Safety задачи.

---

### 9.6. MapsToastTask

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Maps\" -TaskName "MapsToastTask"
```

**Что делает:** отключает toast-задачу карт.

---

### 9.7. RetailDemo cleanup

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\RetailDemo\" -TaskName "CleanupOfflineContent"
```

**Что делает:** отключает очистку офлайн-контента RetailDemo.

---

### 9.8. SpeechModelDownloadTask

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Speech\" -TaskName "SpeechModelDownloadTask"
```

**Что делает:** отключает загрузку speech-моделей.

---

### 9.9. Flighting / A-B tests Microsoft

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

**Что делает:** отключает задачи Flighting/feature experiments/usage reporting.

---

### 9.10. Sustainability

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Sustainability\" -TaskName "PowerGridForecastTask"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Sustainability\" -TaskName "SustainabilityTelemetry"
```

**Что делает:** отключает задачи Sustainability/энерго-телеметрии.

---

### 9.11. AccountHealth

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\AccountHealth\" -TaskName "RecoverabilityToastTask"
```

**Что делает:** отключает уведомления Account Health / recovery toast.

---

### 9.12. WindowsAI — попытка отключения, но отказано

Пытались:

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\WindowsAI\ClickToDo\" -TaskName "ModelCachingIdle"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\WindowsAI\ClickToDo\" -TaskName "ModelCachingLimit"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\WindowsAI\ClickToDo\" -TaskName "ModelCachingUpdate"
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\WindowsAI\Recall\" -TaskName "PolicyConfiguration"
```

**У нас:** `Отказано в доступе`.

**Итог:** не добавлять как обязательный рабочий твик.

---

## 10. UWP / Appx удаление

> Это уже debloat. Не всем нужно. Microsoft Store мы удаляли, но обычному пользователю лучше подумать дважды.

### 10.1. Удалить Microsoft Store

```powershell
Get-AppxPackage *WindowsStore* | Remove-AppxPackage
```

**Что делает:** удаляет Microsoft Store для текущего пользователя.

**Минус:** сложнее ставить/обновлять Store-приложения.

---

### 10.2. Удалить Web Experience

```powershell
Get-AppxPackage *WebExperience* | Remove-AppxPackage
```

**Что делает:** удаляет Windows Web Experience Pack / виджетную часть.

---

### 10.3. Удалить Phone Link

```powershell
Get-AppxPackage Microsoft.YourPhone | Remove-AppxPackage
```

**Что делает:** удаляет Phone Link.

---

### 10.4. Удалить Outlook for Windows

```powershell
Get-AppxPackage Microsoft.OutlookForWindows | Remove-AppxPackage
```

**Что делает:** удаляет новый Outlook.

---

### 10.5. Удалить Clipchamp

```powershell
Get-AppxPackage Clipchamp.Clipchamp | Remove-AppxPackage
```

**Что делает:** удаляет Clipchamp.

---

### 10.6. Удалить Dev Home

```powershell
Get-AppxPackage Microsoft.Windows.DevHome | Remove-AppxPackage
```

**Что делает:** удаляет Dev Home.

---

### 10.7. Удалить Get Help

```powershell
Get-AppxPackage Microsoft.GetHelp | Remove-AppxPackage
```

**Что делает:** удаляет приложение “Получить помощь”.

---

### 10.8. Удалить Bing Search

```powershell
Get-AppxPackage Microsoft.BingSearch | Remove-AppxPackage
```

**Что делает:** удаляет Bing Search app.

---

### 10.9. Удалить Power Automate Desktop

```powershell
Get-AppxPackage Microsoft.PowerAutomateDesktop | Remove-AppxPackage
```

**Что делает:** удаляет Power Automate Desktop.

---

### 10.10. Удалить CrossDevice

```powershell
Get-AppxPackage MicrosoftWindows.CrossDevice | Remove-AppxPackage
```

**Что делает:** удаляет CrossDevice, связку мобильных устройств/кросс-девайс функций.

**У нас:** после удаления стало меньше RuntimeBroker/фонового UWP-мусора.

---

### 10.11. Xbox appx — частично не удалилось

```powershell
Get-AppxPackage *Xbox* | Remove-AppxPackage
```

**Что делает:** пытается удалить Xbox-компоненты.

**У нас:** `Microsoft.XboxGameCallableUI` не удалился, потому что системный компонент.

---

### 10.12. NarratorQuickStart — не удалять как рабочую команду

```powershell
Get-AppxPackage Microsoft.Windows.NarratorQuickStart | Remove-AppxPackage
```

**У нас:** ошибка `0x80073CFA`, системное приложение.

---

## 11. Windows Optional Features / Capabilities

### 11.1. Отключить Work Folders

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName WorkFolders-Client -NoRestart
```

**Что делает:** отключает Work Folders Client.

---

### 11.2. Отключить Internet Printing Client

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName Printing-Foundation-InternetPrinting-Client -NoRestart
```

**Что делает:** отключает клиент интернет-печати.

---

### 11.3. Отключить WCF TCP Port Sharing

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName WCF-TCP-PortSharing45 -NoRestart
```

**Что делает:** отключает WCF TCP Port Sharing.

---

### 11.4. Удалить Hello Face

```powershell
Remove-WindowsCapability -Online -Name Hello.Face.20134~~~~0.0.1.0
```

**Что делает:** удаляет компонент Windows Hello Face.

**Не делать**, если используете вход лицом/Windows Hello.

---

### 11.5. Удалить Handwriting RU

```powershell
Remove-WindowsCapability -Online -Name Language.Handwriting~~~ru-RU~0.0.1.0
```

**Что делает:** удаляет русский рукописный ввод.

---

### 11.6. Удалить Math Recognizer

```powershell
Remove-WindowsCapability -Online -Name MathRecognizer~~~~0.0.1.0
```

**Что делает:** удаляет распознавание математического ввода.

---

### 11.7. Удалить Steps Recorder

```powershell
Remove-WindowsCapability -Online -Name App.StepsRecorder~~~~0.0.1.0
```

**Что делает:** удаляет старое средство записи действий.

---

## 12. Defender — безопасный вариант: исключения

> Рекомендуемый вариант для большинства: **не убивать Defender**, а добавить исключения для Steam/игр/проектов.

### 12.1. Добавить Steam в исключения

```powershell
Add-MpPreference -ExclusionPath "C:\Program Files (x86)\Steam"
```

**Что делает:** Defender не сканирует Steam-папку в реальном времени.

---

### 12.2. Добавить SteamLibrary в исключения

Пример:

```powershell
Add-MpPreference -ExclusionPath "D:\SteamLibrary"
```

**Что делает:** Defender не сканирует библиотеку Steam.

**Путь заменить на свой.**

---

### 12.3. Добавить папку проектов в исключения

```powershell
Add-MpPreference -ExclusionPath "$env:USERPROFILE\Desktop\projects"
```

**Что делает:** Defender не сканирует папку проектов.

**Путь заменить на свой.**

---

### 12.4. Проверить список исключений

```powershell
Get-MpPreference | Select-Object -ExpandProperty ExclusionPath
```

**Что делает:** выводит текущие исключения Defender.

---

## 13. Defender — по желанию / рискованно

> Этот раздел не для всех. Полное отключение Defender снижает безопасность.  
> На Windows 11 25H2 часть настроек может откатываться Tamper Protection.

### 13.1. Вручную отключить Tamper Protection

```text
Windows Security → Virus & threat protection → Manage settings → Tamper Protection → Off
```

**Что делает:** разрешает менять часть параметров Defender.

**Без этого команды могут не сработать или откатиться.**

---

### 13.2. Отключить real-time monitoring

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
```

**Что делает:** отключает защиту в реальном времени.

---

### 13.3. Отключить behavior monitoring

```powershell
Set-MpPreference -DisableBehaviorMonitoring $true
```

**Что делает:** отключает поведенческий мониторинг.

---

### 13.4. Отключить IOAV protection

```powershell
Set-MpPreference -DisableIOAVProtection $true
```

**Что делает:** отключает проверку загружаемых/открываемых файлов через IOAV.

---

### 13.5. Отключить script scanning

```powershell
Set-MpPreference -DisableScriptScanning $true
```

**Что делает:** отключает сканирование скриптов.

---

### 13.6. Policy-ключ DisableAntiSpyware

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableAntiSpyware /t REG_DWORD /d 1 /f
```

**Что делает:** добавляет policy-ключ отключения Defender.

**Важно:** в новых Windows может игнорироваться/откатываться.

**Откат:**

```cmd
reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableAntiSpyware /f
```

---

### 13.7. Policy-ключ DisableRealtimeMonitoring

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection" /v DisableRealtimeMonitoring /t REG_DWORD /d 1 /f
```

**Что делает:** добавляет policy-ключ отключения real-time protection.

**Откат:**

```cmd
reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender\Real-Time Protection" /v DisableRealtimeMonitoring /f
```

---

## 14. UAC — по желанию / рискованно

> Не всем нужно. Отключение UAC снижает безопасность и может странно влиять на UWP/часть системных функций.

### 14.1. Отключить UAC через EnableLUA

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v EnableLUA /t REG_DWORD /d 0 /f
```

**Что делает:** отключает UAC на уровне EnableLUA.

**Требуется перезагрузка.**

**Откат:**

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v EnableLUA /t REG_DWORD /d 1 /f
```

---

### 14.2. Убрать prompt для администратора

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v ConsentPromptBehaviorAdmin /t REG_DWORD /d 0 /f
```

**Что делает:** меняет поведение запроса повышения прав для администратора.

**Откат к стандартному поведению:**

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v ConsentPromptBehaviorAdmin /t REG_DWORD /d 5 /f
```

---

### 14.3. Отключить secure desktop для UAC

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v PromptOnSecureDesktop /t REG_DWORD /d 0 /f
```

**Что делает:** отключает затемнённый secure desktop для UAC prompts.

**Откат:**

```cmd
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" /v PromptOnSecureDesktop /t REG_DWORD /d 1 /f
```

---

## 15. MPO / DWM fix

> Один из главных полезных твиков в нашем расследовании.  
> Помог при сценарии: CS2 на основном мониторе 200 Hz, Яндекс Музыка/Chromium/Electron на втором 100 Hz → фризы при разворачивании окна.

### 15.1. Отключить MPO / добавить DWM overrides

Создать файл `disable_mpo.reg`:

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Dwm]
"OverlayTestMode"=dword:00000005
"OverlayMinFPS"=dword:00000000
```

**Что делает:** добавляет DWM override для MPO/overlay behavior.

**После применения:** перезагрузка.

---

### 15.2. Откат MPO fix

Создать файл `restore_mpo.reg`:

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Dwm]
"OverlayTestMode"=-
"OverlayMinFPS"=-
```

**Что делает:** удаляет параметры `OverlayTestMode` и `OverlayMinFPS`.

**После применения:** перезагрузка.

---

### 15.3. Проверить MPO-ключи

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows\Dwm" /v OverlayTestMode
reg query "HKLM\SOFTWARE\Microsoft\Windows\Dwm" /v OverlayMinFPS
```

**Что делает:** показывает, применён ли MPO-твик.

---

## 16. MSI Utility v3 / USB-контроллер мыши

> MSI Utility v3 — GUI-утилита, но часть диагностики делалась через PowerShell.

### 16.1. Найти мыши и их родителей

```powershell
Get-PnpDevice -Class Mouse | ForEach-Object {
    $mouse = $_
    $parent = (Get-PnpDeviceProperty -InstanceId $mouse.InstanceId -KeyName DEVPKEY_Device_Parent).Data
    [PSCustomObject]@{ Mouse = $mouse.FriendlyName; Parent = $parent }
}
```

**Что делает:** показывает мыши и их parent device.

**Зачем:** найти цепочку от HID-мыши до USB-контроллера.

---

### 16.2. Узнать parent конкретного устройства

```powershell
Get-PnpDeviceProperty -InstanceId "<точный InstanceId>" -KeyName DEVPKEY_Device_Parent
```

**Что делает:** показывает родителя устройства.

**Как использовать:** повторять по цепочке:

```text
HID device → USB Composite Device → USB Root Hub → PCI USB Controller
```

---

### 16.3. Найти точный InstanceId USB-устройства

```powershell
Get-PnpDevice | Where-Object {$_.InstanceId -like "USB\VID_XXXX&PID_XXXX\*"} | Select InstanceId
```

**Что делает:** находит полный InstanceId USB-устройства по VID/PID.

**Важно:** `VID_XXXX&PID_XXXX` заменить на реальные значения устройства.

---

### 16.4. Проверить MSI / Interrupt Priority в реестре

Примерная проверка GPU/USB делалась аудитом, но вручную можно смотреть ветку:

```text
HKLM\SYSTEM\CurrentControlSet\Enum\<PNP_ID>\Device Parameters\Interrupt Management\
```

**Что смотреть:**  
- `MSISupported`;
- `MessageNumberLimit`;
- `DevicePriority`.

**У нас:** для GPU был `DevicePriority=3`, то есть High.

---

### 16.5. Что выставлялось в MSI Utility v3

Через MSI Utility v3 GUI:

```text
NVIDIA GeForce RTX 3070 Ti → MSI enabled / High
правильный AMD USB Controller для ATTACK SHARK X3 → MSI enabled / High
```

**Что делает:** повышает interrupt priority для GPU и USB-контроллера мыши.

**Важно:** не ставить High на всё подряд.

---

## 17. HAGS / Game Mode / графика Windows

### 17.1. Проверить HAGS

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" /v HwSchMode
```

**Что делает:** показывает состояние Hardware-Accelerated GPU Scheduling.

Обычно:

```text
HwSchMode=2 → HAGS включён
HwSchMode=1 → HAGS выключен
```

---

### 17.2. Включить HAGS

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" /v HwSchMode /t REG_DWORD /d 2 /f
```

**Что делает:** включает Hardware-Accelerated GPU Scheduling.

**После изменения:** перезагрузка.

**У нас:** HAGS в итоге оставлен включённым.

---

### 17.3. Выключить HAGS для теста

```cmd
reg add "HKLM\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" /v HwSchMode /t REG_DWORD /d 1 /f
```

**Что делает:** выключает HAGS.

**У нас:** связка HAGS Off + BIOS DLF дала хуже FPS/DPC, поэтому HAGS вернули On.

---

### 17.4. Проверить Game Mode registry value

```cmd
reg query "HKCU\Software\Microsoft\GameBar" /v AutoGameModeEnabled
```

**Что делает:** проверяет Game Mode.

Если ключа нет, лучше смотреть через GUI:

```text
Параметры → Игры → Игровой режим
```

**У нас:** Game Mode был включён через настройки Windows.

---

## 18. BIOS / UEFI — Data Link Feature Exchange

> Это не PowerShell, но важный пункт расследования.

### 18.1. Где находится на MSI B550 Gaming Plus

```text
BIOS → F7 Advanced Mode → SETTINGS → Advanced → PCI Subsystem Settings → Data Link Feature Exchange → Enabled
```

**Что делает:** включает PCIe Data Link Feature Exchange.

**Что пробовали:** связку:

```text
HAGS Off + Data Link Feature Exchange Enabled
```

**Результат на нашей системе:**  
- FPS просел;
- DPC latency вырос.

**Что откатили:** HAGS вернули во включённое состояние.

**DLF/DLFE:** отдельно от HAGS не был полноценно перепроверен, поэтому как “обязательный твик” его указывать нельзя. Максимум — как экспериментальный BIOS-пункт.

---

## 19. Realtek LAN power saving

Это делалось через GUI, но важно для гайда.

```text
Диспетчер устройств → Сетевые адаптеры → Realtek PCIe GbE Family Controller → Свойства → Дополнительно
```

Поставить:

```text
Power Saving Mode: Off
Energy Efficient Ethernet / EEE: Off
Green Ethernet: Off, если есть
Interrupt Moderation: оставить On
```

**Что делает:** убирает энергосбережение сетевой карты, которое может добавлять микроджиттер.

**DNS не трогается.**

---

## 20. NVIDIA / ShadowPlay / Instant Replay

Это делалось через GUI NVIDIA App.

### 20.1. Что изменили

```text
Instant Replay: включён
Replay length: 5 min → 3 min
Bitrate: 33 Mbps → 15 Mbps
Resolution: 1080p
FPS: 60
Recording location: HDD
```

**Что делает:** снижает фоновую нагрузку от постоянной записи, но не убивает полезную функцию.

---

## 21. CRU / монитор 180 Hz → 200 Hz

### 21.1. Что делали

```text
CRU → выбрать Redmi G24 → Detailed resolutions → добавить/изменить 1920x1080 @ 200 Hz → OK → restart64.exe
```

**Что делает:** добавляет кастомную герцовку 200 Hz.

### 21.2. Важные файлы CRU

```text
restart64.exe — перезапускает видеодрайвер
reset-all.exe — аварийно сбрасывает все кастомные режимы
```

**Важно:** держать `reset-all.exe` под рукой до экспериментов с герцовкой.

### 21.3. Что ещё надо проверить

```text
TestUFO Frame Skipping
```

**Зачем:** убедиться, что монитор реально показывает 200 кадров, а не пропускает часть кадров.

---

## 22. Steam icon fix / белые ярлыки

### 22.1. Проверить ассоциацию .url

```cmd
assoc .url
```

Нормально:

```text
.url=InternetShortcut
```

---

### 22.2. Проверить ftype InternetShortcut

```cmd
ftype InternetShortcut
```

У нас было:

```text
InternetShortcut="C:\Windows\System32\rundll32.exe" "C:\Windows\System32\ieframe.dll",OpenURL %l
```

---

### 22.3. Обновить иконки Windows

```cmd
ie4uinit.exe -show
```

**Что делает:** пытается обновить/перерегистрировать кэш иконок.

---

### 22.4. Проверить папку Steam icons

```cmd
dir "C:\Program Files (x86)\Steam\steam\games"
```

**Что делает:** показывает `.ico` файлы Steam-ярлыков.

---

### 22.5. Проверить подозрительный .ico как текст

```powershell
Get-Content "C:\Program Files (x86)\Steam\steam\games\ad22805b0d2a5bebc8aabe2007dc9b36f2b50ee1.ico"
```

**Что делало:** выяснили, что вместо иконки скачался HTML `301 Moved Permanently`.

---

### 22.6. Проверить файл через hex

```powershell
Format-Hex "C:\Program Files (x86)\Steam\steam\games\ad22805b0d2a5bebc8aabe2007dc9b36f2b50ee1.ico"
```

**Что делает:** показывает байты файла. У нас подтвердилось, что это HTML, а не ICO.

---

### 22.7. Причина проблемы

Скрипт скачал редирект 301 вместо иконки.

**Фикс в BAT:** использовать `curl -L`, чтобы следовать редиректам.

---

## 23. Команды, которые НЕ стоит давать как обязательные

### 23.1. DoSvc disable

```powershell
Set-Service DoSvc -StartupType Disabled
```

**У нас:** отказано в доступе.

---

### 23.2. WindowsAI scheduled tasks

```powershell
Disable-ScheduledTask -TaskPath "\Microsoft\Windows\WindowsAI\ClickToDo\" -TaskName "ModelCachingIdle"
```

**У нас:** отказано в доступе.

---

### 23.3. CDPUserSvc Set-Service Disabled

```powershell
Get-Service CDPUserSvc* | Set-Service -StartupType Disabled
```

**У нас:** `Параметр задан неверно`.

---

### 23.4. NarratorQuickStart Remove-AppxPackage

```powershell
Get-AppxPackage Microsoft.Windows.NarratorQuickStart | Remove-AppxPackage
```

**У нас:** системное приложение, ошибка удаления.

---

## 24. Минимальный безопасный набор для обычного пользователя

Если не хочется ломать Windows, брать только это:

```powershell
# AMD Chipset / NVIDIA драйверы ставятся вручную с официальных сайтов

# Defender exclusions
Add-MpPreference -ExclusionPath "C:\Program Files (x86)\Steam"
Add-MpPreference -ExclusionPath "D:\SteamLibrary"

# Проверка питания
powercfg /L
powercfg /Q SCHEME_CURRENT SUB_PROCESSOR

# Проверка VBS / HAGS
Get-CimInstance -Namespace root\Microsoft\Windows\DeviceGuard -ClassName Win32_DeviceGuard
reg query "HKLM\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" /v HwSchMode

# MPO fix — если два монитора и есть фризы от окон на втором мониторе
# см. раздел 15

# MouseTester / ресивер мыши — если есть рывки мыши
# см. раздел 16 и MouseTester
```

---

## 25. Что реально считать главным

По результатам расследования, самые полезные изменения:

1. **MPO fix** для двух мониторов с разной герцовкой.
2. **MSI Utility v3 High priority** для GPU и правильного USB-контроллера мыши.
3. **Вынос ресивера ATTACK SHARK X3** на стол.
4. **ShadowPlay/Instant Replay tuning**, а не полное отключение.
5. **Defender exclusions**, а не война с Defender.
6. **Realtek LAN power saving off**.
7. **CapFrameX + LatencyMon + MouseTester**, а не “мне кажется стало плавнее”.

Средний FPS почти не изменился. Плавность выросла из-за 1%/0.2% low, frametime и DPC.
