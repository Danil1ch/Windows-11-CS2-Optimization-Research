# Windows 11 + CS2 Optimization Research

Практические заметки по расследованию микрофризов Counter-Strike 2 на свежей установке Windows 11.

Это не очередной набор "FPS BOOST +500". В репозитории собраны замеры, выводы, полезные изменения, откаты и отдельные предупреждения для рискованных твиков.

> English version: [README.md](README.md)

## О чём репозиторий

Средний FPS уже был высоким, но CS2 ощущалась неровно из-за:

- игл frametime;
- плохих 1% и 0.2% lows;
- фризов при окнах Chromium/Electron на втором мониторе;
- пиков DPC latency в LatencyMon;
- нестабильного polling у беспроводной мыши.

Главный подход: сначала измерить проблему, потом менять по одному блоку и откатывать то, что делает хуже.

## Тестовая система

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

## Структура

- [docs/commands_ru.md](docs/commands_ru.md) - полный справочник команд на русском с пояснениями и откатами.
- [docs/commands_en.md](docs/commands_en.md) - английская версия справочника команд.
- [registry/disable_mpo.reg](registry/disable_mpo.reg) - MPO/DWM-твик, который использовался в расследовании.
- [registry/restore_mpo.reg](registry/restore_mpo.reg) - откат MPO/DWM-твика.

## Использованные инструменты

- LatencyMon
- CapFrameX
- .NET Desktop Runtime 9.0 для CapFrameX
- CRU / Custom Resolution Utility
- NVIDIA Profile Inspector
- MSI Utility v3
- MouseTester

MD5 MSI Utility v3, использованный в расследовании:

```text
c08d7ae2fff3052fd801f6bf33831d08
```

DDU, TestMem5 и Process Lasso в этом прогоне не использовались, поэтому они не выдаются за обязательные шаги.

## Результаты

| Метрика | До | После |
| --- | ---: | ---: |
| Average FPS | ~344.7 | ~345 |
| 1% Low Average | ~77.4 | ~137.8 |
| 0.2% Low / P0.2 | ~42.1 | ~143.8 |
| LatencyMon DPC peak | ~2249 us, dxgkrnl.sys | ~1136 us, nvlddmkm.sys |
| Wireless mouse polling | частые пики 5-10 ms | заметно чище после выноса ресивера |
| Фризы от второго монитора | жёсткие фризы | небольшая просадка FPS после MPO fix |

Средний FPS почти не изменился. Плавность выросла за счёт frametime, 1%/0.2% lows, DPC latency и polling мыши.

## Что помогло больше всего

1. MPO fix для двух мониторов с разной герцовкой.
2. MSI Utility v3 High interrupt priority только для GPU и правильного USB-контроллера.
3. Вынос ресивера ATTACK SHARK X3 на стол, дальше от задней панели ПК и 2.4 ГГц помех.
4. Снижение нагрузки ShadowPlay / Instant Replay вместо полного отключения.
5. Defender exclusions для Steam и игровых библиотек вместо полного отключения Defender.
6. Отключение энергосбережения Realtek LAN при сохранении Interrupt Moderation.
7. Замеры через CapFrameX, LatencyMon и MouseTester, а не оценка "на глаз".

## Что не помогло

- HAGS Off + BIOS Data Link Feature Exchange на этой системе сделали хуже.
- Рандомное отключение служб почистило Windows, но не решило CS2 напрямую.
- Идея, что другой Steam-аккаунт работает плавнее, не подтвердилась по проверенным CS2 cloud/config файлам.

## Безопасный старт

Если не хочется рисковать системой, начинайте с диагностики и обратимых изменений:

1. Замерьте frametime через CapFrameX.
2. Проверьте DPC через LatencyMon.
3. Проверьте polling мыши через MouseTester.
4. Применяйте MPO fix только если есть два монитора с разной герцовкой и окна на втором мониторе вызывают фризы.
5. Добавьте Defender exclusions для Steam/игровых папок.
6. Вынесите ресивер беспроводной мыши из задней панели ПК.

Полный список команд лежит в [docs/commands_ru.md](docs/commands_ru.md).

## Предупреждения

- Не запускайте все команды подряд.
- Не ставьте High priority всем устройствам в MSI Utility.
- Не отключайте Windows Hello, VBS, Defender, UAC или точки восстановления без понимания последствий.
- Перед разгоном монитора через CRU держите рядом `reset-all.exe`.
- После изменений в реестре, драйверах, MPO, HAGS, службах, планировщике, BIOS или питании сначала перезагрузитесь, потом оценивайте результат.

## Настройки CS2

- 1280x1024, 4:3 stretched
- NVIDIA Reflex: Вкл + Ускорение
- V-Sync: Выкл
- FPS limit: 0
- MSAA: 2x
- Shadows: Medium
- Models/textures: Low
- HDR: Performance
- FSR: Off
