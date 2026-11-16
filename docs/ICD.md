# Master ICD

Каталог интерфейсов между трекером, сервером и локальным журналом. В таблице приведены логические имена сообщений. [Соглашение об именах и прежние обозначения](DECISIONS.md#message-naming)

[Сценарии обмена](PROCESS_FLOW.md)

<a id="messages"></a>

## Сообщения

| Контракт | Имя | Расшифровка | Назначение |
| --- | --- | --- | --- |
| ICD-001 | [`POSITION`](POSITION.md) | Position | Трекер передаёт свежую GNSS-позицию серверу. |
| ICD-002 | [`TX_STATS`](TX_RX_STATS.md#tx_stats) | Transmission Statistics | Трекер сообщает статистику первичных попыток отправки. |
| ICD-003 | [`RX_STATS`](TX_RX_STATS.md#rx_stats) | Reception Statistics | Сервер сообщает трекеру статистику приёма позиций. |
| ICD-007 | [`DEVICE_STATUS`](DEVICE_STATUS.md) | Device Status | Рабочее имя сообщения об оперативном состоянии трекера. |
| Выдача диапазонов | [`RANGE_REQ`](POSITION.md#range-allocation) | Sequence Range Request | Трекер запрашивает диапазон номеров. |
| Выдача диапазонов | [`RANGE_RESP`](POSITION.md#range-allocation) | Sequence Range Response | Сервер возвращает границы выданного диапазона. |
| Выдача диапазонов | [`RANGE_ERR`](POSITION.md#open-questions) | Sequence Range Error | Сервер сообщает об отказе в выдаче. |
| Смена ключа | [`REKEY_REQ`](KEY_MANAGEMENT.md#rekey) | Rekey Request | Трекер запрашивает смену рабочего ключа. |
| Смена ключа | [`REKEY_RESP`](KEY_MANAGEMENT.md#rekey) | Rekey Response | Сервер возвращает результат смены ключа. |
| ICD-005 | [`ARCHIVE_BLOCK`](ARCHIVE_TRANSFER.md#open-questions) | Archive Block | Трекер отправляет блок записей журнала. |
| ICD-005 | [`ARCHIVE_ACK`](ARCHIVE_TRANSFER.md#ack-and-resume) | Archive Acknowledgement | Успешный ответ сервера подтверждает устойчивое сохранение блока. |
| ICD-006 | [`DEVICE_INFO_REQ`](DECISIONS.md#device-info) | Device Information Request | Сервер запрашивает сведения об устройстве. |
| ICD-006 | [`DEVICE_INFO_RESP`](DECISIONS.md#device-info) | Device Information Response | Трекер возвращает сведения об устройстве. |

Сообщения ICD-005 входят в [ARCHIVE_TRANSFER](ARCHIVE_TRANSFER.md).

## Локальный журнал

- ICD-004 · [`POSITION_RECORD`](POSITION_RECORD.md) — Position Record, формат локальной записи измерения и результата отправки.

## Что ещё определить

| Область | Что предстоит определить | Обсуждаемый вариант |
| --- | --- | --- |
| Сверка ключей | [Состав и формат обмена](KEY_MANAGEMENT.md#open-questions). | Если потребуется отдельная пара, `KEY_STATE_REQ` (Key State Request) для запроса и `KEY_STATE_RESP` (Key State Response) для ответа. |
| Управление ICD-006 | Детали запроса сведений об устройстве, [конфигурации и OTA](SRS.md#46-device-management--security). | [Открытые вопросы управления](DECISIONS.md#open-questions) |

[Карта проекта](PROJECT_MAP.md)
