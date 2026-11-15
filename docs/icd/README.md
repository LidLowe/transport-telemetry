# Master ICD

| ID | Интерфейс | Назначение | Статус | Связи |
| --- | --- | --- | --- | --- |
| ICD-001 | [POSITION_V1](POSITION_V1.md) | Позиция трекера в live-потоке | 20 Б логической позиции, 28 Б защищённой UDP-датаграммы. Best-effort без прикладного ACK **APPROVED**. | [Выдача номеров R5](POSITION_V1.md#sequence), [ключи](../security/KEY_MANAGEMENT.md) |
| ICD-002 | [TX_SUMMARY](TX_RX_SUMMARY.md#tx_summary) | Минутная статистика отправителя | Семантика **APPROVED**, формат на 9 Б **PROPOSED**. | `POSITION_V1`, учётное окно |
| ICD-003 | [RX_SUMMARY](TX_RX_SUMMARY.md#rx_summary) | Независимая минутная статистика приёма | Семантика **APPROVED**, формат на 9 Б **PROPOSED**. | `POSITION_V1`, учётное окно |
| ICD-004 | [POSITION_LOG_RECORD_V1](POSITION_LOG_RECORD_V1.md) | Локальная запись измерения и отправки | Единая запись и рабочий ориентир 35 Б **APPROVED**. Раскладка **PROPOSED**. | `POSITION_V1`, `SEND_STATUS_V1`, CRC16 |
| ICD-005 | [ARCHIVE_TRANSFER](ARCHIVE_TRANSFER.md) | Фоновая выгрузка журнала | TCP/TLS 1.3, идемпотентное сохранение и ACK после устойчивой записи **APPROVED**. Формат блока и ACK **OPEN**. | Локальный журнал, разрешённый `trackerId` |
| ICD-006 | Управление, конфигурация и OTA | Защищённые команды, сведения об устройстве и обновление ключей | [GET_DEVICE_INFO](../DECISIONS.md#get_device_info--запрос-информации-о-трекере) и принципы [rekey](../security/KEY_MANAGEMENT.md#установление-и-обновление-ключей) **APPROVED**. Полный контракт **OPEN**. | Авторизация, [SRS §4.6](../SRS.md#46-device-management--security) |
| ICD-007 | [DEVICE_STATUS](DEVICE_STATUS.md) | Отдельное сообщение о состоянии трекера | Добавление в план **APPROVED**. Формат, семантика и способ отправки **OPEN**. | `POSITION_V1`, мониторинг GNSS и связи |

[Управление ключами](../security/KEY_MANAGEMENT.md) · [Журнал решений](../DECISIONS.md) · [Системные требования](../SRS.md)
