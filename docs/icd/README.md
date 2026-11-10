# Master ICD

| ID | Интерфейс | Назначение | Размер и статус | Зависимости |
|---|---|---|---|---|
| ICD-001 | [POSITION_V1](POSITION_V1.md) | Свежая позиция трекера | 20 Б, **UDP**, live No-ACK **APPROVED**. AES-CCM-8 **APPROVED** (цель 28 Б UDP payload), ключи/nonce/упаковка **OPEN**. | Идентичность события и серверный приём |
| ICD-002 | [TX_SUMMARY](TX_RX_SUMMARY.md#tx_summary) | Минутная статистика отправителя | Логика **APPROVED**, 9 Б **PROPOSED**. | POSITION и учётное окно |
| ICD-003 | [RX_SUMMARY](TX_RX_SUMMARY.md#rx_summary) | Независимая минутная статистика приёма | Логика **APPROVED**, 9 Б **PROPOSED**. | POSITION и учётное окно |
| ICD-004 | [POSITION_LOG_RECORD_V1](POSITION_LOG_RECORD_V1.md) | Локальная запись измерения и отправки | Единая запись и рабочая цель 35 Б **APPROVED**. Раскладка **PROPOSED**. | Идентичность POSITION, SEND_STATUS, CRC16 |
| ICD-005 | [ARCHIVE_TRANSFER](ARCHIVE_TRANSFER.md) | Фоновая передача журнала | Приоритет, прикладной ACK и **TCP APPROVED**. TLS **PROPOSED**, формат блока и защита **OPEN**. | Локальный журнал, архивный приём |
| ICD-006 | Управление, конфигурация и OTA | Защищённые команды, подтверждение применения; запрос сведений о трекере `GET_DEVICE_INFO` | `GET_DEVICE_INFO` как функциональная потребность и защищённый обмен **APPROVED**. Поля ответа, формат, механизм защиты и полный контракт **OPEN**. Требования заданы в [SRS v1.1](../../legacy/originals/Transport_Telemetry_SRS_v1.1.md). | Аутентификация, полномочия, жизненный цикл устройства |
| ICD-007 | [DEVICE_STATUS](DEVICE_STATUS.md) *(рабочее название)* | Отдельные сведения о состоянии трекера, включая доступность GNSS-фиксации | Добавление отдельного пакета в план **APPROVED**. Формат, семантика, периодичность и транспорт **OPEN**. | Мониторинг связи и GNSS; `POSITION_V1` |

[Журнал решений](../DECISIONS.md) · [Карта переноса и история](../MIGRATION_PLAN.md) · [Открытое расхождение с SRS](POSITION_V1.md#live-доставка)
