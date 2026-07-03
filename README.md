# Windows 11 + CS2 Optimization Research

**EN:** Practical research notes from troubleshooting Counter-Strike 2 microstutters on a fresh Windows 11 installation.
**RU:** Практические заметки по расследованию микрофризов Counter-Strike 2 на свежей установке Windows 11.

This repository is not a generic "FPS boost" pack. It documents what was tested, what helped, what did not help, and which changes should be treated as risky or optional.

Это не очередной набор "FPS BOOST +500". Здесь собраны замеры, выводы, полезные изменения, откаты и предупреждения для рискованных твиков.

## Language / Язык

- [English overview](#english-overview)
- [Русское описание](#русское-описание)
- [Quick commands / Быстрые команды](#quick-commands--быстрые-команды)
- [Full command reference / Полный справочник команд](#full-command-reference--полный-справочник-команд)

## English overview

### What this is about

The system already had high average FPS, but CS2 still felt uneven because of:

- frametime spikes;
- poor 1% and 0.2% lows;
- stutters when Chromium/Electron apps were visible on a second monitor;
- DPC latency peaks in LatencyMon;
- unstable wireless mouse polling.

The useful work came from measuring the problem, changing one area at a time, and rolling back tweaks that made the system worse.

### Test system

| Component | Details |
| --- | --- |
| CPU | AMD Ryzen 7 5700X, PBO +200 MHz, Curve Optimizer -30 all-core |
| GPU | NVIDIA GeForce RTX 3070 Ti |
| RAM | 32 GB DDR4 @ 3733 MHz CL17 |
| Motherboard | MSI MPG B550 Gaming Plus |
| OS | Windows 11 Pro 25H2 / build 26200, fresh install |
| Main monitor | Redmi G24, 180 Hz -> 200 Hz through CRU |
| Second monitor | Acer 100 Hz |
| Mouse | ATTACK SHARK X3 wireless |
| Headset | Logitech G435 wireless |
| Recording | NVIDIA Instant Replay / ShadowPlay, recording to HDD |

### Tools used

| Tool | Used for | Notes |
| --- | --- | --- |
| [LatencyMon](https://www.resplendence.com/latencymon) | DPC latency checks | Helped identify driver latency peaks such as `dxgkrnl.sys` and `nvlddmkm.sys`. |
| [CapFrameX](https://github.com/CXWorld/CapFrameX/releases) | FPS, frametime, 1% low and 0.2% low measurements | Requires [.NET Desktop Runtime 9.0](https://dotnet.microsoft.com/en-us/download/dotnet/9.0). |
| [MSI Utility v3](https://www.mediafire.com/file/ewpy1p0rr132thk/MSI_util_v3.zip) | MSI mode / interrupt priority tuning | Used only for the GPU and the correct USB controller. Do not set everything to High. |
| [CRU / Custom Resolution Utility](https://github.com/kreier/cru/releases/) | Monitor refresh-rate testing | Used for the Redmi G24 180 Hz -> 200 Hz test. Keep `reset-all.exe` nearby. |
| [NVIDIA Profile Inspector](https://github.com/Orbmu2k/nvidiaProfileInspector/releases) | NVIDIA profile inspection and tuning | Used for driver/profile-level checks. |
| [MouseTester](https://github.com/dobragab/MouseTester) | Mouse polling stability checks | Helped compare wired vs wireless polling and receiver placement. |
| [NVIDIA App / ShadowPlay](https://www.nvidia.com/en-us/software/nvidia-app/) | Instant Replay load tuning | Replay length and bitrate were reduced instead of disabling recording completely. |

MSI Utility v3 checksum used in this research:

```text
c08d7ae2fff3052fd801f6bf33831d08
```

DDU, TestMem5, and Process Lasso were not part of this run, so they are not presented as required steps.

### Results

| Metric | Before | After |
| --- | ---: | ---: |
| Average FPS | ~344.7 | ~345 |
| 1% Low Average | ~77.4 | ~137.8 |
| 0.2% Low / P0.2 | ~42.1 | ~143.8 |
| LatencyMon DPC peak | ~2249 us, dxgkrnl.sys | ~1136 us, nvlddmkm.sys |
| Wireless mouse polling | frequent 5-10 ms spikes | much cleaner after moving the receiver |
| Second-monitor stutter | hard freezes | small FPS drop after MPO fix |

Average FPS barely changed. Smoothness improved because frametime stability, lows, DPC latency, and mouse polling became better.

### What helped most

1. MPO fix for a mixed-refresh dual-monitor setup.
2. MSI Utility v3 high interrupt priority for the GPU and the correct USB controller only.
3. Moving the ATTACK SHARK X3 receiver to the desk with cleaner line of sight.
4. Lowering ShadowPlay / Instant Replay load instead of disabling it completely.
5. Defender exclusions for Steam and game libraries instead of fully disabling Defender.
6. Disabling Realtek LAN power saving features while keeping Interrupt Moderation enabled.
7. Measuring with CapFrameX, LatencyMon, and MouseTester instead of relying on feel alone.

### What did not help

- HAGS Off + BIOS Data Link Feature Exchange made this system worse.
- Random Windows service disabling cleaned the system but did not directly fix CS2.
- The idea that a different Steam account was smoother was not supported by inspected CS2 cloud/config files.

## Русское описание

### О чём репозиторий

Средний FPS уже был высоким, но CS2 ощущалась неровно из-за:

- игл frametime;
- плохих 1% и 0.2% lows;
- фризов при окнах Chromium/Electron на втором мониторе;
- пиков DPC latency в LatencyMon;
- нестабильного polling у беспроводной мыши.

Главный подход: сначала измерить проблему, потом менять по одному блоку и откатывать то, что делает хуже.

### Тестовая система

| Компонент | Детали |
| --- | --- |
| CPU | AMD Ryzen 7 5700X, PBO +200 MHz, Curve Optimizer -30 all-core |
| GPU | NVIDIA GeForce RTX 3070 Ti |
| RAM | 32 ГБ DDR4 @ 3733 MHz CL17 |
| Motherboard | MSI MPG B550 Gaming Plus |
| OS | Windows 11 Pro 25H2 / build 26200, свежая установка |
| Основной монитор | Redmi G24, 180 Гц -> 200 Гц через CRU |
| Второй монитор | Acer 100 Гц |
| Мышь | ATTACK SHARK X3 wireless |
| Гарнитура | Logitech G435 wireless |
| Запись | NVIDIA Instant Replay / ShadowPlay, запись на HDD |

### Использованные инструменты

| Инструмент | Для чего использовался | Пометка |
| --- | --- | --- |
| [LatencyMon](https://www.resplendence.com/latencymon) | Проверка DPC latency | Помог увидеть пики драйверов вроде `dxgkrnl.sys` и `nvlddmkm.sys`. |
| [CapFrameX](https://github.com/CXWorld/CapFrameX/releases) | Замеры FPS, frametime, 1% low и 0.2% low | Нужен [.NET Desktop Runtime 9.0](https://dotnet.microsoft.com/en-us/download/dotnet/9.0). |
| [MSI Utility v3](https://www.mediafire.com/file/ewpy1p0rr132thk/MSI_util_v3.zip) | MSI mode / interrupt priority | Использовался только для GPU и правильного USB-контроллера. Не ставить High на всё подряд. |
| [CRU / Custom Resolution Utility](https://github.com/kreier/cru/releases/) | Тест герцовки монитора | Использовался для Redmi G24 180 Гц -> 200 Гц. Перед экспериментами держать `reset-all.exe` рядом. |
| [NVIDIA Profile Inspector](https://github.com/Orbmu2k/nvidiaProfileInspector/releases) | Проверка и настройка NVIDIA-профилей | Использовался для driver/profile-level проверок. |
| [MouseTester](https://github.com/dobragab/MouseTester) | Проверка стабильности polling мыши | Помог сравнить проводной/беспроводной режим и понять проблему расположения ресивера. |
| [NVIDIA App / ShadowPlay](https://www.nvidia.com/en-us/software/nvidia-app/) | Настройка нагрузки Instant Replay | Длина повтора и bitrate были снижены вместо полного отключения записи. |

MD5 MSI Utility v3, использованный в расследовании:

```text
c08d7ae2fff3052fd801f6bf33831d08
```

DDU, TestMem5 и Process Lasso в этом прогоне не использовались, поэтому они не выдаются за обязательные шаги.

### Результаты

| Метрика | До | После |
| --- | ---: | ---: |
| Average FPS | ~344.7 | ~345 |
| 1% Low Average | ~77.4 | ~137.8 |
| 0.2% Low / P0.2 | ~42.1 | ~143.8 |
| LatencyMon DPC peak | ~2249 us, dxgkrnl.sys | ~1136 us, nvlddmkm.sys |
| Wireless mouse polling | частые пики 5-10 ms | заметно чище после выноса ресивера |
| Фризы от второго монитора | жёсткие фризы | небольшая просадка FPS после MPO fix |

Средний FPS почти не изменился. Плавность выросла за счёт frametime, 1%/0.2% lows, DPC latency и polling мыши.

### Что помогло больше всего

1. MPO fix для двух мониторов с разной герцовкой.
2. MSI Utility v3 High interrupt priority только для GPU и правильного USB-контроллера.
3. Вынос ресивера ATTACK SHARK X3 на стол, дальше от задней панели ПК и 2.4 ГГц помех.
4. Снижение нагрузки ShadowPlay / Instant Replay вместо полного отключения.
5. Defender exclusions для Steam и игровых библиотек вместо полного отключения Defender.
6. Отключение энергосбережения Realtek LAN при сохранении Interrupt Moderation.
7. Замеры через CapFrameX, LatencyMon и MouseTester, а не оценка "на глаз".

### Что не помогло

- HAGS Off + BIOS Data Link Feature Exchange на этой системе сделали хуже.
- Рандомное отключение служб почистило Windows, но не решило CS2 напрямую.
- Идея, что другой Steam-аккаунт работает плавнее, не подтвердилась по проверенным CS2 cloud/config файлам.

## Quick commands / Быстрые команды

> Do not run everything blindly. Most commands require Windows Terminal as Administrator.
> Не запускайте всё подряд. Большинство команд требуют Терминал Windows от администратора.

### Diagnostics / Диагностика

```powershell
powercfg /L
powercfg /Q SCHEME_CURRENT SUB_PROCESSOR
Get-CimInstance -Namespace root\Microsoft\Windows\DeviceGuard -ClassName Win32_DeviceGuard
bcdedit /enum
Get-WindowsOptionalFeature -Online | Where-Object {$_.FeatureName -match "Hyper|Virtual|Sandbox|Containers"}
fsutil behavior query DisableDeleteNotify
```

### Defender exclusions / Исключения Defender

```powershell
Add-MpPreference -ExclusionPath "C:\Program Files (x86)\Steam"
Add-MpPreference -ExclusionPath "D:\SteamLibrary"
Add-MpPreference -ExclusionPath "$env:USERPROFILE\Desktop\projects"
Get-MpPreference | Select-Object -ExpandProperty ExclusionPath
```

### MPO fix / MPO-твик

Use [registry/disable_mpo.reg](registry/disable_mpo.reg), then reboot.

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Dwm]
"OverlayTestMode"=dword:00000005
"OverlayMinFPS"=dword:00000000
```

Rollback / откат: [registry/restore_mpo.reg](registry/restore_mpo.reg), then reboot.

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Dwm]
"OverlayTestMode"=-
"OverlayMinFPS"=-
```

### HAGS check / Проверка HAGS

```cmd
reg query "HKLM\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" /v HwSchMode
```

Usually / обычно:

```text
HwSchMode=2 -> HAGS enabled / включён
HwSchMode=1 -> HAGS disabled / выключен
```

### Mouse USB chain / Цепочка USB для мыши

```powershell
Get-PnpDevice -Class Mouse | ForEach-Object {
    $mouse = $_
    $parent = (Get-PnpDeviceProperty -InstanceId $mouse.InstanceId -KeyName DEVPKEY_Device_Parent).Data
    [PSCustomObject]@{ Mouse = $mouse.FriendlyName; Parent = $parent }
}
```

### Minimal safer set / Минимальный безопасный набор

```powershell
# Defender exclusions
Add-MpPreference -ExclusionPath "C:\Program Files (x86)\Steam"
Add-MpPreference -ExclusionPath "D:\SteamLibrary"

# Power checks
powercfg /L
powercfg /Q SCHEME_CURRENT SUB_PROCESSOR

# VBS / HAGS checks
Get-CimInstance -Namespace root\Microsoft\Windows\DeviceGuard -ClassName Win32_DeviceGuard
reg query "HKLM\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" /v HwSchMode
```

## Repository files / Файлы репозитория

- [README_RU.md](README_RU.md) - отдельная русская версия.
- [docs/commands_en.md](docs/commands_en.md) - полный английский справочник команд.
- [docs/commands_ru.md](docs/commands_ru.md) - полный русский справочник команд.
- [registry/disable_mpo.reg](registry/disable_mpo.reg) - MPO/DWM registry tweak.
- [registry/restore_mpo.reg](registry/restore_mpo.reg) - rollback for the MPO/DWM tweak.

## Warnings / Предупреждения

- Do not run every command blindly. / Не запускайте все команды подряд.
- Do not set every MSI Utility device to High. / Не ставьте High priority всем устройствам в MSI Utility.
- Do not disable Windows Hello, VBS, Defender, UAC, or restore points unless you understand the tradeoff. / Не отключайте Windows Hello, VBS, Defender, UAC или точки восстановления без понимания последствий.
- Keep CRU `reset-all.exe` available before monitor overclocking. / Перед разгоном монитора через CRU держите рядом `reset-all.exe`.
- Reboot after registry, driver, MPO, HAGS, service, scheduled task, BIOS, or power-plan changes before judging results. / После изменений сначала перезагрузитесь, потом оценивайте результат.

## Full command reference / Полный справочник команд

<details>
<summary><strong>Русский: полный список PowerShell / CMD / Registry команд</strong></summary>

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

</details>

<details>
<summary><strong>English: full PowerShell / CMD / Registry command reference</strong></summary>

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

</details>
