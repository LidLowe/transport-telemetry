# Master ICD

**APPROVED (P1 R9):** ранние дешёвые проверки UDP (28 Б, 0x11, допустимый ключевой контекст), лимиты ресурсов до AES-CCM, максимум две проверки ключей при rekey. Открытый trackerId не удостоверяет устройство, количественные ограничения OPEN. [Контракт](POSITION_V1.md#r9--предварительные-проверки-и-ограничение-ресурса-udp-приёмника).

**APPROVED (P1 R7):** архив TCP/TLS 1.3 повторяет неподтверждённые неизменные блоки со стабильной идентичностью; atomic durable commit и дедупликация `(trackerId, sequence)` предшествуют прикладному ACK. Сетевой ACK не является подтверждением сохранения; wire-format и лимиты OPEN. [Контракт](ARCHIVE_TRANSFER.md#подтверждение-и-возобновление).

**APPROVED (P1 R6):** новая live-позиция публикуется только при `sequence > lastPublishedSequence` этого trackerId (независимо от key epoch); при отсутствии новых сообщений UI показывает состояние устаревших данных, порог OPEN. Нет нового поля assignmentVersion в UDP. [Контракт](POSITION_V1.md#live-доставка).

**APPROVED (P1 R4):** для live `POSITION_V1` одно неизменяемое измерение на `sequence`, один AES-128-CCM-8 encrypt и одна прикладная попытка отправки UDP; никаких application-level retries. История восстанавливается через защищённый архив, сервер обрабатывает сетевые дубликаты через anti-replay. [Live-контракт](POSITION_V1.md#live-доставка).

**APPROVED (P1 R8):** любая выдача UDP-ключей, rekey, архивная выгрузка и защищённое управление требуют серверной проверки удостоверенной идентичности физического устройства на право использования конкретного `trackerId`. Блокировка отзывает все текущие и pending UDP-ключевые эпохи и доступ к TLS-операциям. [Канонические требования](../security/KEY_MANAGEMENT.md#авторизация-trackerid-и-отзыв-доступа).

| ID | Интерфейс | Назначение | Размер и статус | Зависимости |
|---|---|---|---|---|
| ICD-001 | [POSITION_V1](POSITION_V1.md) | Свежая позиция трекера | 20 Б, **UDP**, live No-ACK **APPROVED**. AES-128-CCM-8 (ключ 16 Б, TLS 1.3 Exporter с привязкой к device/purpose/direction/epoch) и wire layout **APPROVED** (7 Б AAD + 13 Б ciphertext + 8 Б tag = 28 Б UDP payload), аппаратная защита ключей и TLS 1.3 для rekey **APPROVED**, конструкция nonce **APPROVED**, выбор key epoch через `K_new`/`K_old` **APPROVED**; при утрате серверного anti-replay **fail-closed и новый TLS/rekey APPROVED**, сроки/окно/формат проверки **OPEN**, а **периодическая TLS-сверка + ограниченный срок ключа APPROVED**. | Идентичность события и серверный приём |
| ICD-002 | [TX_SUMMARY](TX_RX_SUMMARY.md#tx_summary) | Минутная статистика отправителя | Логика **APPROVED**, 9 Б **PROPOSED**. | POSITION и учётное окно |
| ICD-003 | [RX_SUMMARY](TX_RX_SUMMARY.md#rx_summary) | Независимая минутная статистика приёма | Логика **APPROVED**, 9 Б **PROPOSED**. | POSITION и учётное окно |
| ICD-004 | [POSITION_LOG_RECORD_V1](POSITION_LOG_RECORD_V1.md) | Локальная запись измерения и отправки | Единая запись и рабочая цель 35 Б **APPROVED**. Раскладка **PROPOSED**. | Идентичность POSITION, SEND_STATUS, CRC16 |
| ICD-005 | [ARCHIVE_TRANSFER](ARCHIVE_TRANSFER.md) | Фоновая передача журнала | TCP/TLS 1.3 **APPROVED**; атомарный идемпотентный commit + прикладной ACK после durable-save **R7 APPROVED**, формат блока/ACK и лимиты **OPEN**. | Локальный журнал, архивный приём |
| ICD-006 | Управление, конфигурация и OTA | Защищённые команды, подтверждение применения; `GET_DEVICE_INFO` и сервисное обновление ключей | `GET_DEVICE_INFO` и **сервисный rekey APPROVED**. Логический обмен `REKEY_REQUEST → REKEY_ACK`, логические `requestId/currentEpoch/newEpoch`, проверка повторов и сверка эпох после TLS reconnect **APPROVED**; бинарные коды, размеры полей, тайм-ауты и полный контракт **OPEN**. Требования заданы в [SRS v1.1](../../legacy/originals/Transport_Telemetry_SRS_v1.1.md). | Аутентификация, полномочия, жизненный цикл устройства |
| ICD-007 | [DEVICE_STATUS](DEVICE_STATUS.md) *(рабочее название)* | Отдельные сведения о состоянии трекера, включая доступность GNSS-фиксации | Добавление отдельного пакета в план **APPROVED**. Формат, семантика, периодичность и транспорт **OPEN**. | Мониторинг связи и GNSS; `POSITION_V1` |

[Управление ключами](../security/KEY_MANAGEMENT.md) · [Журнал решений](../DECISIONS.md) · [Карта переноса и история](../MIGRATION_PLAN.md) · [Открытое расхождение с SRS](POSITION_V1.md#live-доставка)
