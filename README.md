# Transport Telemetry — подготовленный пакет для Codex

**Дата среза:** 2026-11-09. Переносимый набор содержит все найденные документы проекта, завершённый **рабочий срез старой схемы** и новую процессную документацию. Актуальные интерфейсы собраны в [Master ICD](docs/icd/README.md) и четырёх тематических спецификациях.

## Быстрый вход (2 минуты)

1. [CHECKPOINT](docs/CHECKPOINT.md) — на чём остановились.
2. [PROJECT_MAP](docs/PROJECT_MAP.md) — вся система на одной странице.
3. [Master ICD](docs/icd/README.md) — каталог и текущие технические контракты.
4. [AGENTS.md](AGENTS.md) — правила совместной работы с Codex.
5. [PROJECT_CONTEXT](docs/PROJECT_CONTEXT.md) — **подробный** контекст, все доступные причины и решения.
6. [MIGRATION_PLAN](docs/MIGRATION_PLAN.md) — куда перенесён каждый раздел старого ICD.

## Дерево

```text
transport-telemetry/
├── AGENTS.md                   # правила, Codex читает при работе в репозитории
├── README.md
├── baseline-old-schema/        # замороженный рабочий срез до миграции
│   ├── README.md
│   ├── Transport_Telemetry_SRS_v1.2_DRAFT_r4.md
│   ├── Transport_Telemetry_ICD_v0.9_SNAPSHOT.md
│   ├── Transport_Telemetry_ICD_POSITION_V1_v0.2_SNAPSHOT.md
│   ├── Transport_Telemetry_Diagnostics_ICD_v0.3_SNAPSHOT.md
│   ├── Transport_Telemetry_Local_Journal_ICD_v0.3_SNAPSHOT.md
│   └── Transport_Telemetry_DECISIONS_v0.6_SNAPSHOT.md
├── docs/
│   ├── PROJECT_MAP.md
│   ├── CHECKPOINT.md
│   ├── PROJECT_CONTEXT.md
│   ├── DECISIONS.md            # решения и будущие области проектирования
│   ├── MIGRATION_PLAN.md
│   └── icd/
│       ├── README.md           # Master ICD
│       ├── POSITION_V1.md
│       ├── TX_RX_SUMMARY.md
│       ├── POSITION_LOG_RECORD_V1.md
│       └── ARCHIVE_TRANSFER.md
└── legacy/
    ├── MANIFEST.tsv            # SHA-256 и размер каждого оригинала
    └── originals/             # 22 исходных Markdown без изменений
```

## Ключевое ограничение

`SRS v1.1` остаётся прежним BASELINE. `SRS v1.2_DRAFT_r4` — только рабочий проект изменений. Даже если в ICD согласована концепция, неподтверждённые детали `flags`, профиля CRC и архивного транспорта остаются PROPOSED/OPEN.

## История

[Срез старой схемы](baseline-old-schema/README.md) и [оригиналы](legacy/MANIFEST.tsv) сохраняют состояние до этапа 3. [CHECKSUMS.sha256](CHECKSUMS.sha256) относится к исходному пакету.
