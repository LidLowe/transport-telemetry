# Master ICD

| ID | Интерфейс | Назначение | Размер и статус | Зависимости |
|---|---|---|---|---|
| ICD-001 | [POSITION_V1](POSITION_V1.md) | Свежая позиция трекера | 20 Б, **UDP**, live No-ACK **APPROVED**. AES-CCM-8 **APPROVED** (цель 28 Б UDP payload), аппаратная защита ключей и TLS 1.3 для rekey **APPROVED**, конструкция nonce **APPROVED**, key epoch/упаковка **OPEN**. | Идентичность события и серверный приём |
| ICD-002 | [TX_SUMMARY](TX_RX_SUMMARY.md#tx_summary) | Минутная статистика отправителя | Логика **APPROVED**, 9 Б **PROPOSED**. | POSITION и учётное окно |
| ICD-003 | [RX_SUMMARY](TX_RX_SUMMARY.md#rx_summary) | Независимая минутная статистика приёма | Логика **APPROVED**, 9 Б **PROPOSED**. | POSITION и учётное окно |
| ICD-004 | [POSITION_LOG_RECORD_V1](POSITION_LOG_RECORD_V1.md) | Локальная запись измерения и отправки | Единая запись и рабочая цель 35 Б **APPROVED**. Раскладка **PROPOSED**. | Идентичность POSITION, SEND_STATUS, CRC16 |
| ICD-005 | [ARCHIVE_TRANSFER](ARCHIVE_TRANSFER.md) | Фоновая передача журнала | Приоритет, прикладной ACK и **TCP APPROVED**. TLS 1.3 **APPROVED**, формат блока и детали авторизации **OPEN**. | Локальный журнал, архивный приём |
| ICD-006 | Управление, конфигурация и OTA | Защищённые команды, подтверждение применения; `GET_DEVICE_INFO` и сервисное обновление ключей | `GET_DEVICE_INFO` и **сервисный rekey APPROVED**. Логический обмен `REKEY_REQUEST → REKEY_ACK` и новый TLS 1.3/ECDHE-контекст **APPROVED**; бинарные коды, поля, тайм-ауты и полный контракт **OPEN**. Требования заданы в [SRS v1.1](../../legacy/originals/Transport_Telemetry_SRS_v1.1.md). | Аутентификация, полномочия, жизненный цикл устройства |
| ICD-007 | [DEVICE_STATUS](DEVICE_STATUS.md) *(рабочее название)* | Отдельные сведения о состоянии трекера, включая доступность GNSS-фиксации | Добавление отдельного пакета в план **APPROVED**. Формат, семантика, периодичность и транспорт **OPEN**. | Мониторинг связи и GNSS; `POSITION_V1` |

[Управление ключами](../security/KEY_MANAGEMENT.md) · [Журнал решений](../DECISIONS.md) · [Карта переноса и история](../MIGRATION_PLAN.md) · [Открытое расхождение с SRS](POSITION_V1.md#live-доставка)
