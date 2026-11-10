# PROJECT MAP — Transport Telemetry

**Срез 2026-11-09.** [Master ICD](icd/README.md) ведёт к текущим контрактам. [PROJECT_CONTEXT](PROJECT_CONTEXT.md) хранит историю и основания решений.

## Цель

Собственная система GNSS-мониторинга общественного транспорта. MVP — один трекер, сервер, карта и симулятор. Целевые масштабы из SRS — около 5 000 трекеров и 10 000 клиентов. В номинальном режиме поступает одна координата в секунду.

## Потоки

```text
GNSS → tracker → POSITION_V1 (20 Б, UDP) → live backend → текущее состояние → карта
           └──→ POSITION_LOG_RECORD_V1 (~35 Б) → NVM → ARCHIVE_TRANSFER (TCP) → архив
tracker → TX_SUMMARY (60 с) → сервер
tracker ← RX_SUMMARY (60 с) ← сервер, независимо от TX
управление и OTA ↔ защищённый подтверждаемый обмен (полный контракт OPEN)
```

Свежая позиция имеет приоритет. `(trackerId, sequence)` связывает представления одного события, а назначение трекера автобусу ведёт сервер. Время измерения, локальной отправки и серверного приёма различаются. История может иметь пропуски, её выгрузка не меняет текущее состояние и live RX.

## Интерфейсы

| Интерфейс | Текущее состояние |
|---|---|
| [POSITION_V1](icd/POSITION_V1.md) | 20 Б, UDP и best-effort live без прикладного ACK **APPROVED**. AES-CCM-8 и wire layout **APPROVED** (7 Б AAD + 13 Б ciphertext + 8 Б tag = 28 Б); хранение ключей и TLS 1.3 rekey **APPROVED**, nonce **APPROVED**, выбор key epoch без `keyId` через два допустимых ключа **APPROVED**; после потери серверного anti-replay **fail-closed с новым TLS/rekey APPROVED**, сроки/окно/триггер восстановления **OPEN**. |
| [TX_SUMMARY и RX_SUMMARY](icd/TX_RX_SUMMARY.md) | Независимые окна 60 с **APPROVED**. По 9 Б **PROPOSED**. |
| [POSITION_LOG_RECORD_V1](icd/POSITION_LOG_RECORD_V1.md) | Одна запись и рабочая цель 35 Б **APPROVED**. Раскладка **PROPOSED**. |
| [ARCHIVE_TRANSFER](icd/ARCHIVE_TRANSFER.md) | Фоновая передача блоками, архивный ACK и **TCP APPROVED**. TLS 1.3 **APPROVED**, формат блоков/авторизация **OPEN**. |
| [ICD-006](icd/README.md) | Управление, конфигурация, OTA и сервисный rekey **APPROVED как потребности**. Формат команд и полного контракта **OPEN**. |
| [DEVICE_STATUS (ICD-007)](icd/DEVICE_STATUS.md) | Отдельный пакет статуса включён в план **APPROVED**. Формат и правила отправки **OPEN**. |

## Ключи и защищённая среда

**APPROVED (архитектура):** [KEY_MANAGEMENT](security/KEY_MANAGEMENT.md) — индивидуальный долгосрочный приватный ключ в защищённой области MCU либо Secure Element; временный AES-CCM-8-ключ только в RAM, TLS 1.3/ECDHE для установления и недельной ротации по отдельному сервисному запросу трекера, новый ключ после reboot, без открытого fallback. Временная поддержка двух ключевых эпох и логический обмен `REKEY_REQUEST → REKEY_ACK` при rekey **APPROVED**; первый успешно проверенный UDP новой эпохи неявно подтверждает переход. Логические `requestId/currentEpoch/newEpoch` и сверка после TLS reconnect **APPROVED**; бинарные размеры/коды и timeout/overlap **OPEN**. Противодействие физическому доступу — требование, реализация и threat model OPEN.

## Будущие области проектирования

| Область | Статус |
|---|---|
| Мониторинг доступности | Наблюдение, журнал и оповещение **APPROVED** как потребность. Модель доступности **PROPOSED**, пороги и сообщения канала **OPEN**. [Общий ICD §9](../baseline-old-schema/Transport_Telemetry_ICD_v0.9_SNAPSHOT.md#9-мониторинг-трекеров-и-журнал-событий) |
| Длительные стоянки | Обнаружение **APPROVED** как потребность. Молчание трекера и последний фикс сами по себе не доказывают стоянку. Пороги и реакции **OPEN**. [Общий ICD §15](../baseline-old-schema/Transport_Telemetry_ICD_v0.9_SNAPSHOT.md#15-контроль-длительных-стоянок-транспорта-функциональная-потребность) |
| Питание и RTC | Питание независимо от зажигания и резервный источник RTC для сохранения UTC **APPROVED**. GNSS — основной источник сверки, сеть — резервный. Выбор схемы аккумуляторов, энергорежимы, точность RTC и доступность OTA вне рейса **OPEN**. [Решение](DECISIONS.md#rtc-питание-и-синхронизация-времени-position_v1), [Исходный ICD §18.2](../baseline-old-schema/Transport_Telemetry_ICD_v0.9_SNAPSHOT.md#182-питание-и-доступность-управления) |
| Режимы работы и ночные рейсы | Ночные рейсы **APPROVED** как ограничение. Выбор режима по состоянию рейса **PROPOSED**, критерии и переходы **OPEN**. [Общий ICD §18.1](../baseline-old-schema/Transport_Telemetry_ICD_v0.9_SNAPSHOT.md#181-не-привязывать-режим-к-времени-суток) |

Подробности и исходные статусы приведены в [DECISIONS](DECISIONS.md#будущие-области-проектирования).

## Точка входа

[CHECKPOINT](CHECKPOINT.md) → [открытые параметры локальной записи](icd/POSITION_LOG_RECORD_V1.md#открытые-параметры). Первый вопрос — необходимость `flags`.

[SRS v1.1](../legacy/originals/Transport_Telemetry_SRS_v1.1.md) остаётся **BASELINE** с требованиями к устойчивому приёму и replay. Их согласование с live-профилем **OPEN**, [SRS v1.2](../baseline-old-schema/Transport_Telemetry_SRS_v1.2_DRAFT_r4.md) остаётся **DRAFT**. [Карта переноса](MIGRATION_PLAN.md) связывает новые контракты с замороженными источниками.
