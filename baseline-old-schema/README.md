# OLD-SCHEMA SNAPSHOT — index

**Дата среза:** 2026-11-09. Этот каталог — точка передачи документов в Codex **до** перехода к новой иерархии.

## Какие документы являются актуальным рабочим срезом

| Файл | Область | Статус |
|---|---|---|
| `Transport_Telemetry_SRS_v1.2_DRAFT_r4.md` | Требования и change requests | DRAFT, не принят вместо SRS v1.1 |
| `Transport_Telemetry_ICD_v0.9_SNAPSHOT.md` | Полный общий ICD старой схемы | WORKING SNAPSHOT; содержит историю вариантов |
| `Transport_Telemetry_ICD_POSITION_V1_v0.2_SNAPSHOT.md` | Подробная спецификация live POSITION | APPROVED состав 20 Б, wire details OPEN |
| `Transport_Telemetry_Diagnostics_ICD_v0.3_SNAPSHOT.md` | TX_SUMMARY и RX_SUMMARY | APPROVED логика, размер 9 Б PROPOSED |
| `Transport_Telemetry_Local_Journal_ICD_v0.3_SNAPSHOT.md` | 35-Б journal и архив | APPROVED принцип, многие GNSS fields PROPOSED |
| `Transport_Telemetry_DECISIONS_v0.6_SNAPSHOT.md` | Решения и статусы | Источник решения/процесса; не замена техническому контракту |

**Важно:** `legacy/originals/Transport_Telemetry_SRS_v1.1.md` — последний исходный SRS BASELINE. Полный текущий проектный контекст в `docs/PROJECT_CONTEXT.md`, рабочие правила в `AGENTS.md`.

## Приоритет в случае противоречия

1. Явное новейшее подтверждение владельца, записанное в `DECISIONS_v0.6_SNAPSHOT` и контексте.
2. Дополнение «Актуальные решения» в соответствующем срезе из этой папки.
3. Последние тематические главы полного исходника.
4. Ранние/исторические главы — только трассировка прежних предложений.

**Исключение:** даже новое APPROVED на уровне ICD не переписывает нормативный baseline SRS v1.1 автоматически. Противоречащие требования должны быть вынесены на формальный change review.

## Чего НЕ делали при заморозке

- Не утверждали CRC-16/IBM-3740, `flags`, TCP/TLS, точную раскладку всего 35-Б journal, тайминги RX/TX.
- Не переопределяли цели p95, 24-часовой буфер, loss policy как гарантии новой ратифицированной SRS.
- Не очищали длинные исторические документы от старых вариантов. Эта работа относится к будущей миграции в Master ICD + тематические ICD и требует отдельного review.
