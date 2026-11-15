# Master ICD

Каталог интерфейсов между трекером, сервером и локальным журналом. В таблице приведены логические имена сообщений. [Соглашение об именах и прежние обозначения](../DECISIONS.md#именование-сообщений)

## Сообщения

| Контракт | Имя | Расшифровка | Назначение |
| --- | --- | --- | --- |
| ICD-001 | [`POSITION`](POSITION.md) | Position | Трекер передаёт свежую GNSS-позицию серверу. |
| ICD-002 | [`TX_STATS`](TX_RX_STATS.md#tx_stats) | Transmission Statistics | Трекер сообщает статистику первичных попыток отправки. |
| ICD-003 | [`RX_STATS`](TX_RX_STATS.md#rx_stats) | Reception Statistics | Сервер сообщает трекеру статистику приёма позиций. |
| ICD-007 | [`DEVICE_STATUS`](DEVICE_STATUS.md) | Device Status | Рабочее имя сообщения об оперативном состоянии трекера. |
| R5 | [`RANGE_REQ`](POSITION.md#sequence) | Sequence Range Request | Трекер запрашивает диапазон номеров. |
| R5 | [`RANGE_RESP`](POSITION.md#sequence) | Sequence Range Response | Сервер возвращает границы выданного диапазона. |
| R5 | [`RANGE_ERR`](POSITION.md#range_err--ошибка-выдачи-диапазона) | Sequence Range Error | Сервер сообщает об отказе в выдаче. |
| Смена ключа | [`REKEY_REQ`](../security/KEY_MANAGEMENT.md#установление-и-обновление-ключей) | Rekey Request | Трекер запрашивает смену рабочего ключа. |
| Смена ключа | [`REKEY_RESP`](../security/KEY_MANAGEMENT.md#установление-и-обновление-ключей) | Rekey Response | Сервер возвращает результат смены ключа. |
| Сверка ключей | [`KEY_STATE_REQ`](../security/KEY_MANAGEMENT.md#контроль-срока-действия-и-периодическая-tls-сверка-ключевых-эпох) | Key State Request | **PROPOSED** для запроса, если у сверки будет отдельный обмен. |
| Сверка ключей | [`KEY_STATE_RESP`](../security/KEY_MANAGEMENT.md#контроль-срока-действия-и-периодическая-tls-сверка-ключевых-эпох) | Key State Response | **PROPOSED** для ответа в том же обмене. |
| ICD-005 | [`ARCHIVE_BLOCK`](ARCHIVE_TRANSFER.md#кандидат-идентификации-блока) | Archive Block | Трекер отправляет блок записей журнала. |
| ICD-005 | [`ARCHIVE_ACK`](ARCHIVE_TRANSFER.md#подтверждение-и-возобновление) | Archive Acknowledgement | Успешный ответ сервера подтверждает устойчивое сохранение блока. |
| ICD-006 | [`DEVICE_INFO_REQ`](../DECISIONS.md#device_info_req-и-device_info_resp--сведения-о-трекере) | Device Information Request | Сервер запрашивает сведения об устройстве. |
| ICD-006 | [`DEVICE_INFO_RESP`](../DECISIONS.md#device_info_req-и-device_info_resp--сведения-о-трекере) | Device Information Response | Трекер возвращает сведения об устройстве. |

Сообщения ICD-005 входят в [ARCHIVE_TRANSFER](ARCHIVE_TRANSFER.md). Полный контракт ICD-006, включая [конфигурацию и OTA](../SRS.md#46-device-management--security), остаётся **OPEN**.

## Локальный журнал

- ICD-004 · [`POSITION_RECORD_V1`](POSITION_RECORD_V1.md) — Position Record, Version 1, формат локальной записи измерения и результата отправки.

[Карта проекта](../PROJECT_MAP.md)
