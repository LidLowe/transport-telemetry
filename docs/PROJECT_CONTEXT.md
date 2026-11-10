# PROJECT_CONTEXT — Transport Telemetry

**Handoff snapshot:** 2026-11-09  
**Status:** context transfer / not an approved new technical specification  
**Audience:** project owner, Codex, future contributors  
**After phase 3:** use the [Master ICD](icd/README.md) for current contracts and the [migration map](MIGRATION_PLAN.md) for traceability. The tables and workflow below describe the frozen handoff.

**Reading priority for this historical handoff:** `baseline-old-schema/README.md` → current frozen old-schema specifications in that folder → `legacy/originals/` (immutable history). The owner-approved architectural decisions are reconstructed below. `SRS v1.1` remains the formal BASELINE; `SRS v1.2` remains DRAFT.

> This document exports the *available project-specific context* reconstructed from the discussion and saved documents. It is not a literal export of private model state, hidden reasoning, or a complete verbatim transcript. It can be incomplete. If this summary conflicts with a referenced detailed source, report the conflict; do not silently choose the preferred answer. New conversation decisions after the last written ICD are listed separately.

## 0. Agreed migration sequence (2026-11-09)

The owner explicitly decided to avoid jumping immediately from long legacy documents into a new hierarchy. **First freeze complete current specifications using the old document scheme; then generate this detailed context file and an AGENTS.md describing the desired new hierarchy; only then use Codex to refactor files after review.** The complete freeze lives in `baseline-old-schema/`. Do not treat the freeze as approval of every byte-level field; see its status prefaces and the Decision Register.

This is a project-context handoff, **not the model's complete private context** or a replacement for source documents. Keep it comprehensive enough to resume after 2–3 days, but maintain PROJECT_MAP and CHECKPOINT as the entrypoints rather than forcing full rereads.

## A. What the owner wants from Codex — collaboration protocol

The owner designs an independent GNSS telemetry system for public transport and wants to **retain intellectual control over every part**. The failure mode to avoid is an AI generating specifications, interfaces, acronyms and options faster than a human can assimilate them. In prior sessions an innocent question about a single `flags` byte led to a whole flag table and a new design issue, causing loss of orientation.

The owner wants to keep AI's full engineering ability, but work under a controlled **divide-and-conquer, progressive-disclosure** method:

1. First explain **why a thing exists**, in plain engineering terms, then describe how it works, then propose design choices. Answer the actual question before adding alternatives.
2. Discuss **one meaningful design question at a time**. If a new issue appears, put it in OPEN/backlog rather than expanding it into an unrequested multi-section design.
3. Keep at most 2–3 genuinely different alternatives in ordinary decision discussions; articulate a practical trade-off.
4. Mark statuses consistently: `APPROVED` only after explicit owner confirmation; `PROPOSED` for AI suggestions; `OPEN` for unresolved issues; `REJECTED` for rejected proposals; `BASELINE` for an existing formally adopted document even if a change is being prepared.
5. Distinguish **engineering proposal** from **documentation migration**. Never silently elevate historical proposals to requirements.
6. Show concise, delta-oriented changes: what changed, what did not, why, what it affects. Do not make the owner reread an 80-page combined ICD to approve a one-byte change.
7. Maintain an accessible one-page PROJECT_MAP and a succinct CHECKPOINT, because the owner may be away for 2–3 days or longer. On return, summarize overall project, last decision, and exact next question before designing further.
8. Hierarchical documentation is the owner's chosen organizational direction: `PROJECT_MAP -> Master ICD (catalog) -> separate complete component ICD`. Maintain a decision register and SRS separately. The general ICD should be short, not another accumulation of entire specifications.
9. Codex is preferred because a real repository, files, diffs, navigation and version control are more comfortable than a sprawling chat. First **inventory and migration plan**, then obtain approval before destructive or semantic consolidation.
10. Do not require memorizing every field. The owner must know system responsibilities, constraints and consequences; specifications hold the bit offsets and implementation detail.

Related workflow: preserve previous files as untouched originals; use sensible Git commits and show diff/statistics after modifications; update `CHECKPOINT.md` at the end of substantial sessions. Treat this file as a snapshot, not an oracle.

## B. System purpose and MVP

**Transport Telemetry** is an independent tracking and observability infrastructure for public buses, covering GNSS tracker hardware/firmware, ingestion backend, current-position state, operator/client map, device management and eventual history analytics.

MVP philosophy: validate one physical tracker, one backend, one live web map, and a simulator before scaling. The owner is investigating the architecture systematically before choosing all hardware, connectivity and implementation details. Do not choose a GNSS chipset, modem, MCU, hosting vendor or programming language without asking unless explicitly tasked.

Original SRS scope and targets from v1.1/v1.2 draft:
- Nominal position acquisition/stream: **1 Hz** = **60 position events per minute**, under valid GNSS fix and normal connectivity.
- Potential scale planning: around **5,000 trackers** and **10,000 simultaneous viewers** (targets, not current load).
- Near-live targets in initial SRS: end-to-end p95 around **800 ms**, displayed position freshness p95 **<= 2 s** in nominal conditions; `STALE` after approximately **5 s** nominally. These are aspirational acceptance targets, not measured guarantees.
- Keep distinction between measured GNSS time, local attempt time, server receive time, server publish time and client render time. Units, clock synchronization and measurement boundaries matter when checking subsecond SLO.
- Avoid showing the last known coordinate as evidence that a bus is still physically there. Distinguish offline, delayed delivery, missing GNSS fix and actual stationary vehicle.
- Real-world routes include night service; never equate nighttime with parked mode.

Actors/identity: `trackerId` is a device ID (not the bus/vehicle ID); tracker-to-vehicle assignments are maintained on backend, including relevant history. A position is logically identified by `(trackerId, sequence)`. Clients must get explicit freshness/monotonic state, not silently overwrite newer fixes with delayed older events. The exact authentication/network framing is not finalized.

## C. Conceptual data flow

```text
    GNSS sample at tracker
             |
             +-- live POSITION_V1 (20 B) -> live intake -> current state -> map
             |                            no application POSITION ACK
             |                            new fixes always highest priority
             |
             +-- local POSITION_LOG_RECORD_V1 (~35 B) -> nonvolatile journal
                                                      -> background archive upload
                                                      -> separate archive intake

    tracker TX_SUMMARY (minute window) ------------> diagnostics intake
    server  RX_SUMMARY (minute window) <------------ independently generated

    device commands/configuration/OTA <-----------> separately authenticated,
                                                     confirmed management path
```

The live map must never wait for archival checks, journal flushes, minute summaries or management tasks. Archival playback must **not** update current live state nor inflate live `rxCount`. Live reception and historical completeness are distinct metrics.

## D. `POSITION_V1` — approved compact live event

**Size: 20 bytes fixed.** Detailed original: `legacy/originals/Transport_Telemetry_ICD_POSITION_V1_v0.1.md`; consolidated historical narrative: `...ICD_v0.8.md`.

| Byte offset | Field | Type | Bytes | Meaning |
|---:|---|---|---:|---|
| 0 | `header` | `uint8` | 1 | High nibble = version, low nibble = message type; candidate `0x11` for v1/POSITION |
| 1–2 | `trackerId` | `uint16` | 2 | Device ID independent of bus identity |
| 3–6 | `sequence` | `uint32` | 4 | Globally monotone across reboots |
| 7–10 | `measuredAt` | `uint32` | 4 | Unix seconds UTC |
| 11–14 | `latitudeE7` | `int32` | 4 | WGS84 latitude × 10^7 |
| 15–18 | `longitudeE7` | `int32` | 4 | WGS84 longitude × 10^7 |
| 19 | `horizontalAccuracy` | `uint8` | 1 | Compact two-range accuracy encoding |

Accuracy byte proposal from initial discussion: high bit 0 represents units of 0.1m; high bit 1 represents units of 1m; `0xFF` unknown. Confirm exact rounding, sentinel/invalid-fix handling and validity constraints before implementation; centimeter *resolution* is not GNSS measurement accuracy.

No speed, heading, altitude, send timestamp, send delta, application CRC or application authentication is embedded in this 20-byte record. `header` currently carries version/type, **not** a generic `flags` field. On-wire byte order / message codes / integrity and security still require finalization where not approved.

`sequence` must never reset on reboot; possible persistence schemes include FRAM/MRAM per event or reserved blocks in Flash/EEPROM. With sequence range reservation, missing sequence numbers do not necessarily prove lost packets. `sourceEpoch` was consciously removed; sequence persistence design OPEN.

## E. Live delivery: explicit new architecture vs older SRS baseline

Owner approved **hybrid live-first, best-effort**: live POSITION messages are sent without per-POSITION application-level ACK or mandatory complete replay; occasional gaps are acceptable and newer coordinates must not queue behind old ones. Critical commands, OTA, configuration and background archive acceptance require appropriate protected reliable confirmation.

**Documentation conflict:** original `SRS v1.1` includes durable accept/replay/24h buffer commitments. These remain the *formal original baseline*, while later owner-approved architectural intent is reflected in ICD/decisions and change-review sections of `SRS v1.2 DRAFT r3`. Never claim v1.2 is formally approved or silently erase baseline conflicts. Prepare an explicit change request/approval before upgrading SRS baseline.

The owner illustrated tolerance of incomplete history (including historical discussion of ~20% missing history as acceptable), and later noted that even losing 200–300 of 10,000 historical records (2–3%) during power loss need not justify complicated storage. **These are context/example figures, not validated SLAs or an authorized precise loss budget.** Do not imply a 2–3% guarantee.

## F. Minute diagnostics — TX_SUMMARY and RX_SUMMARY

`RECEPTION_REPORT` is an **old name**; owner explicitly changed it to **`RX_SUMMARY`**. Always use `RX_SUMMARY` in new material. Historical specifications and old text still include the former name.

- `TX_SUMMARY`: tracker reports **actual count of unique live POSITION events attempted/submitted during a minute**, not merely `generated=60`. If only 48 were attempted while normally expecting 60, that is itself diagnostic even if server received all 48.
- `RX_SUMMARY`: server independently counts unique valid live positions received/attributed to the matching minute and sends its own periodic report.
- Both logical summaries are **independent and asynchronous**. RX is *not* triggered by TX arrival. Missing TX is anomalous and prevents an accurate TX/RX fraction for that window, but server may still know its own RX.
- The current candidate for each message is **9 bytes**: `header:uint8` (1) + `trackerId:uint16` (2) + `windowStart:uint32` (4) + `txCount/rxCount:uint16` (2). This is a **candidate wire layout**, not fully ratified including header values, ordering and ACK policies.
- 60-second windows; close by time, not by reaching 60 events. At start of new minute, prioritize new live POSITION, then send the previous interval's TX summary even if GNSS stops providing fixes.
- Missing TX, delayed reports, stale/offline and poor signal are observability issues, not exceptions that stop the live pipeline. Do not use naive sequence gaps for exact loss measurement.
- Unresolved: whether to count a locally failed `send()` as TX; attribution of send-time vs measurement-time minute; late/out-of-order positions; report revision/grace period, time sync and reboot; whether diagnostic report ACK is needed. The two counts must refer to **the same coherent definition of window** before computing quality ratios.
- Do not add `generated`, `attempted`, `rejected` all at once into the compact format by default. Owner explicitly rejected that verbosity; detailed failures belong in local/server diagnostics.

## G. Local journal — approved principles and detailed draft

Decision: **one logical GNSS event creates a compact live representation and one richer local record**. The initial proposal to have two records `GNSS_RECORD` (32 B) + `SEND_RECORD` (16 B) was superseded by a single merged `POSITION_LOG_RECORD_V1`.

**Working target: 35 B local record**. The conceptual combined record and some field semantics were approved; the *complete byte-by-byte GNSS layout and endianness are not yet approved*. Current source: `legacy/originals/Transport_Telemetry_Local_Journal_ICD_v0.2.md` §11.

| Offset | Field | Size | Current status |
|---:|---|---:|---|
| 0 | `recordHeader` | 1 | PROPOSED version/type encoding |
| 1 | `flags` | 1 | PROPOSED — meanings not agreed |
| 2–5 | `sequence` | 4 | APPROVED logical identity |
| 6–9 | `measuredAt:uint32` | 4 | APPROVED Unix seconds |
| 10–11 | `measuredMs:uint16` | 2 | APPROVED 0..999 ms |
| 12–15 | `latitudeE7:int32` | 4 | APPROVED position semantics |
| 16–19 | `longitudeE7:int32` | 4 | APPROVED position semantics |
| 20–21 | `horizontalAccuracyCm:uint16` | 2 | PROPOSED |
| 22–23 | `speedCms:uint16` | 2 | PROPOSED |
| 24–25 | `courseCdeg:uint16` | 2 | PROPOSED |
| 26–27 | `altitudeDm:int16` | 2 | PROPOSED |
| 28 | `fixType:uint8` | 1 | PROPOSED |
| 29 | `satellitesUsed:uint8` | 1 | PROPOSED |
| 30 | `sendStatus:uint8` | 1 | APPROVED enum |
| 31–32 | `sendDeltaMs:uint16` | 2 | APPROVED monotonic delta |
| 33–34 | `recordCrc16:uint16` | 2 | CRC16 presence APPROVED; parameters PROPOSED/OPEN |
| | **Total** | **35 B** | Working target, not fully final schema |

Time semantics: `measuredAt` Unix UTC seconds; `measuredMs` fraction 0..999; `sendDeltaMs` elapsed milliseconds measured using a **monotonic MCU timer** to the local send attempt, not computed by subtracting potentially corrected UTC clocks. `0xFFFF` means unknown/out of range; values `0..65534` represent 0..65.534 seconds. Reconstruct time when meaningful as `sendAttemptAtMs = measuredAt*1000 + measuredMs + sendDeltaMs`, not proof of radio emission or server receipt. Since measuredAt is only seconds in live POSITION, adding these local fields does **not** expand live format.

`SEND_STATUS_V1` **explicitly approved**:

| Code | Meaning |
|---|---|
| `0x00 NOT_ATTEMPTED` | no attempt |
| `0x01 SUBMITTED` | network stack/modem locally accepted submission, **not delivered** |
| `0x02 SUBMIT_FAILED` | local API attempted; returned error |
| `0x03 NO_NETWORK` | no attempt because offline |
| `0x04 INVALID_FIX` | no live POSITION due to invalid GNSS fix |
| `0x05 QUEUE_DROPPED` | local queue discarded before send |
| `0x06..0xFE` | RESERVED |
| `0xFF UNKNOWN` | unknown outcome |

`sendDeltaMs` normally `0xFFFF` if no send attempt; on SUBMITTED use a known interval if available; on SUBMIT_FAILED delay to failure may be known. Precise interface point for SUBMITTED requires chosen modem API. The journal is *best effort*: in abrupt power loss, some last entries can be lost. **Never block freshest live POSITION for strict journal durability**. Still detect corrupted/incomplete records and avoid fabricated valid data.

### CRC16 discussion (after the saved v0.8 ICD)

Owner learned CRC is a **non-cryptographic integrity check**, computed after record contents are finalized and stored in last two bytes. For proposed 35 B layout: CRC over **first 33 bytes**, compare when reading. The **presence** of CRC16 is approved; the specific **CRC-16/IBM-3740 (often called CCITT-FALSE)** profile with `width=16`, `poly=0x1021`, `init=0xFFFF`, `refin=false`, `refout=false`, `xorout=0`, check for ASCII `123456789` = `0x29B1` and a 256-entry lookup table (512 B) was explicitly left **PROPOSED**, *not approved*. Byte order for stored CRC, performance benchmarks and framing require final decision. Live path does not compute this journal CRC.

### `flags` discussion (most recent design pause)

User asked what flags are, indicating earlier assistant explanation had expanded too rapidly. A bitflag is a single Boolean encoded in one bit. In live `POSITION_V1` header, upper/lower nibbles encode version/type; no dedicated flags byte exists. Candidate local `flags:uint8` at offset 1 would encode GNSS field validity (e.g., valid position, speed, course, altitude, time, accuracy). That **example bit allocation was only PROPOSED**, never approved; some bits may duplicate `fixType`. Before proposing a table, explain the purpose and ask whether field validity is needed separately from existing fix/sentinel representation. Current stopping point: `flags` understanding/necessity and possible simplification.

### Retrospective data and additional diagnostic sampling

- Speed/course/altitude/fix type/satellite count are proposals aimed at understanding movement and GNSS quality. Owner has not approved exact widths or all fields.
- A separate `MODEM_SAMPLE` record (periodic or on meaningful state changes) was proposed, not approved. Continuous 1Hz modem queries may be wasteful and values may update at slower rates.
- A 35 B record at 1 Hz for 24h corresponds to **3,024,000 B** = **3.024 MB** ≈ **2.884 MiB** per device/day, excluding metadata. Use this for rough planning only. Earlier 32+16 B model totals 4.1472 MB/day and is superseded.

## H. Archive/background upload

Approved direction: journal data should be uploaded **periodically, approximately daily** as low-priority background work, not necessarily at midnight. Night buses may be active. The upload is resumable in principle, uses bounded pieces/offsets and **application-level acknowledgement after durable server acceptance**, distinct from TCP ACK. A failed archive operation must not interrupt live delivery. History may be incomplete; server must not quietly represent missing historical data as complete.

**TCP/TLS stream was PROPOSED** as simpler than inventing reliable UDP block transport; it has not received explicit final owner approval. TCP provides ordered reliable byte stream but does not define application record boundaries, resume offsets, authorization or durable persistence. Past illustrative schedule comparison: 1, 2, 4, 8 uploads/day = once every 24, 12, 6, 3 hours. The '4 uploads/day' recommendation was **PROPOSED, not APPROVED**; scheduling still OPEN. 4 KiB blocks were one illustrative idea, **not a requirement**.

Potential data path: stored journal -> optional block framing -> TCP/TLS -> server archive ingestion -> integrity validation + durable storage -> application confirmation -> safely mark local range confirmed, subject to retention. No need to guarantee complete source history at any cost.

## I. Observability and fleet semantics

- Backend should distinguish `lastSeenAt` (communication), `lastValidFixAt` (good GNSS), stale/live/offline, last archived event, actual motion and historical gaps. A no-fix device can be connected; an offline device can have its last-known coordinate persisted; neither means 'bus parked'.
- Aggregated metrics and asynchronous event logs (e.g., POSITION_STALE, DEVICE_OFFLINE, RECOVERED), not expensive per-POSITION exceptional logs on critical path. A ratio `rxCount/txCount` is meaningless without aligned windows and count semantics.
- Long stationary intervals may be useful operational signals, but require fresh, reliable fixes and route context. Do not automatically accuse or penalize drivers. Driver feedback is proposed, not approved implementation.
- Freshness/ordering: the latest server-recorded arrival is not always the newest measured event; use measuredAt/sequence semantics and explicit state transitions.
- Client visibility and historical location are potentially sensitive. Access control, retention and authentication need a separate security design rather than unprotected public archives.

## J. Device operations and hardware questions

- Desired ability to manage device, update firmware (OTA), configure intervals and collect diagnostics, with authenticated and confirmed critical commands. Keep live POSITION independent.
- GNSS, MCU, cellular modem, NVM, antenna, vehicle power supply and possible backup supply not selected. Consider GNSS clock quality and monotonic MCU timer.
- Sequence must remain monotonic across power cycling; persistence strategy OPEN (range reservation may skip sequence values, which is not evidence of packet loss).
- Vehicle supply/ignition, backup battery, backup-time OTA, low-power mode, parking detection and 24h buffer implications are unresolved. Do not assume all buses are inactive at night.
- Write endurance and recovery strategy for local journal OPEN; acceptable loss of last events on sudden power failure was explicitly preferred over heavyweight transactions.

## K. Security/transport and source-of-truth limitations

Live POSITION security, identity/authentication, transport framing, on-wire byte order and CRC protection are not all decided; do not assume '20B application payload' is a complete secure transport. Configuration and OTA need protected management, signatures/authenticated content as appropriate. Archive TCP/TLS is proposed.

The repository is currently primarily **documentation**, not a verified running implementation. Do not invent code repositories, CI status, deployed services, chipset details or verified performance results.

There are multiple historical documents that include superseded choices. Start from **latest version per topic** while keeping older versions as evidence, and validate with the owner when contradictions remain. In particular:

1. `SRS v1.1` requires stronger durable replay than the live-first design later approved verbally. `SRS v1.2_DRAFT_r3` is *not* formally ratified.
2. Early ICD sections discuss Solo/Batch ACK and `RECEPTION_REPORT`; they are archival discussions, not current architecture.
3. Early Local Journal sections describe separate GNSS/SEND records; §11 of v0.2 supersedes them with merged 35B target.
4. `CRC-16/IBM-3740` parameters were agreed only as a proposal; latest saved ICD still says CRC parameters OPEN.
5. Detailed `flags` meaning was never approved.
6. Two kinds of success must not be mixed: `SUBMITTED` (local API) versus `DELIVERED` (server receive); archive block `ACK` differs from TCP ACK.
7. `TX_SUMMARY` count semantics for failed local submission and minute boundary assignment are OPEN.

## L. Chronology of major design evolution

- Initial SRS v1.1: end-to-end live latency and quality targets, 24h buffer, durable ingestion/ACK/replay ambition.
- Initial ICD: fixed 20 B POSITION, globally monotonic sequence, one-second Unix measurement time, compressed accuracy.
- Live design reevaluation: user questions ACK overhead, opts for unacknowledged best-effort POSITION with strict new-data priority; critical management remains reliable.
- Minute statistics: user rejects redundant generated/attempted/rejected fields; settles on one `txCount` and `rxCount` side-band report, candidate 9 B each; independent async TX and RX; missing TX flagged.
- History/diagnostics: two representations of event, rich local journal plus background upload; do not add `sendDelta` to live POSITION.
- First local proposal: 32 B GNSS record + 16 B SEND record, optional separate MODEM_SAMPLE. Both later superseded/kept as proposals, respectively.
- User suggests archive TCP and merging records; merged `POSITION_LOG_RECORD_V1` chosen, with best-effort persistence; 35 B target and efficient `measuredAt` seconds + `measuredMs` + monotonic `sendDeltaMs`.
- Approved SEND_STATUS enum; CRC16 needed, but specific algorithm CRC-16/IBM-3740 with table only PROPOSED.
- Last technical question: `flags` and difference from `recordHeader`; user highlighted cognitive overload and loss of project understanding.
- New collaboration process: PROJECT_MAP, CHECKPOINT, DECISIONS, compact MASTER ICD, narrow per-interface full specs, explicit statuses and small changes. Owner wants to migrate working docs to Codex.

## M. What to do first in Codex (do not silently execute redesign)

1. Open `README.md`, `docs/PROJECT_MAP.md`, `docs/CHECKPOINT.md`, this file and `AGENTS.md`.
2. Inventory `legacy/originals/` using `MANIFEST.tsv`, identify newest topic sources and conflicts.
3. Show the owner a **migration plan and exact proposed changes**. Retain originals unmodified; do not prematurely rewrite SRS or assign final byte offsets.
4. After owner approval, create concise canonical `docs/icd/README.md` and separate fully specified components (POSITION, TX_RX, POSITION_LOG, ARCHIVE_TRANSFER). Resolve contradictions only via confirmed decisions, not by arbitrary preference.
5. Current next engineering issue: whether `flags` actually add information beyond `fixType`/sentinels, followed by remaining GNSS field formats and CRC parameter approval.
6. End with a short updated CHECKPOINT noting last decision and open question. The owner can return after days with just "продолжаем".

## N. Source references

- `legacy/originals/Transport_Telemetry_SRS_v1.1.md` — formal original baseline.
- `legacy/originals/Transport_Telemetry_SRS_v1.2_DRAFT_r3.md` — latest saved draft/change requests.
- `legacy/originals/Transport_Telemetry_ICD_POSITION_V1_v0.1.md` — original compact POSITION specification.
- `legacy/originals/Transport_Telemetry_ICD_v0.8.md` — latest saved combined ICD (includes older historical sections).
- `legacy/originals/Transport_Telemetry_Diagnostics_ICD_v0.2.md` — latest saved detailed minute diagnostic ICD, still uses historical name RECEPTION_REPORT in title.
- `legacy/originals/Transport_Telemetry_Local_Journal_ICD_v0.2.md` — latest saved Local Journal ICD, merged record §11.
- `legacy/originals/Transport_Telemetry_DECISIONS_v0.5.md` — latest saved decision register; additional discussions after this version are recorded in this handoff.
- `legacy/originals/` also contains earlier revisions for traceability. These are source files, not necessarily current active specifications.

**Maintenance rule:** after owner-approved semantic changes, update this handoff if they materially affect the project; update PROJECT_MAP and CHECKPOINT first, then precise component spec and decisions. Do not keep adding historical chapters forever to one master document.

## O. Pre-Codex freeze produced 2026-11-09

- `baseline-old-schema/README.md` lists **six full old-schema working snapshots** and their status. This is the transition point agreed by the owner.
- `legacy/MANIFEST.tsv` verifies SHA-256 for all 22 source files recovered from ChatGPT Library. The original SRS v1.1 was not silently revised.
- `docs/MIGRATION_PLAN.md` explicitly separates completed packaging from future semantic restructuring requiring review.
- `AGENTS.md` describes the new interaction/documentation process, to be used after transferring the files into Codex.
