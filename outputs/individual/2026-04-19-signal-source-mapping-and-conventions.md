---
thread: 2026-04-19-signal-source-mapping-and-conventions
topic: Signal-to-source mapping, delivery/orchestration architecture, and SITREP conventions
started: 2026-04-19
exchange_count: 1
---

> **Note**: This output was produced through AI-assisted research using Claude Code. It is a planning artifact, not an operational tool.

# Signal → Source Mapping, Delivery Architecture, and SITREP Conventions

## Exchange 1

**Summary**: Translate earlier planning into concrete artifacts: a signal→keyword→source matrix, a delivery/orchestration architecture (serverless vs. host, transport roles), and baked-in conventions (UTC-with-IDT timestamps, 24h filenames, a proper SITREP template with source-bias and credibility ratings).

### Key Findings

---

## Part A — Signal → Keyword → Source Mapping

The earlier source-selection exchange defined four **cue classes** (A: kinetic-in-progress, B: escalation pre-signals, C: adversary posture, D: diplomatic). Each cue becomes operational only when bound to concrete keywords and concrete sources. The matrix below is that binding.

Notation:
- **HE** = Hebrew, **AR** = Arabic, **EN** = English, **FA** = Farsi.
- Keyword lists are *illustrative, not exhaustive*. The implementer tunes these against live corpora.
- Sources reference the registry from the source-selection exchange (§5).

### Cue Class A — Kinetic in progress

#### A1. Home Front Command alert firing (ground truth)

| | |
|---|---|
| **What it indicates** | A shelter/siren zone is active *right now* for a specific polygon. This is the only signal that produces an immediate action for the operator. |
| **Keywords / patterns** | HE: `פיקוד העורף`, `צבע אדום`, `התרעה`, `יש להיכנס למרחב מוגן`, `שוהים במרחב המוגן`. EN: `Home Front Command`, `Red Alert`, `shelter`, `rocket alert`. Also: any change to the HFC alert-zone JSON. |
| **Primary sources** | Home Front Command app/API (structured feed); `@AlertsIL` / equivalent alert-scraper bots; `@pikud_haoref` Telegram. |
| **Corroboration sources** | IDF Spokesperson channels; Kan News live feed; `@AbuAliEnglish` / `@AbuAliExpress`. |
| **Threshold band** | `critical` (authoritative single-source — HFC can alert alone per §4 of source-selection). |
| **False-positive filters** | Scheduled drills (`תרגיל`, `drill`, published schedule); anniversary commemorations; test broadcasts. |

#### A2. IDF Spokesperson confirmed event

| | |
|---|---|
| **What it indicates** | Official Israeli confirmation of an operation, interception, or incident. Sets domestic reality line. |
| **Keywords / patterns** | HE: `דובר צה"ל`, `יירוט מוצלח`, `שיגור`, `אירוע ביטחוני`, `תקיפה`, `פגיעה`. EN: `IDF confirms`, `intercepted`, `struck`, `security incident`. AR: `الجيش الإسرائيلي`, `اعتراض`, `استهداف`. |
| **Primary sources** | `@IDF` (EN), `@idfonline` (HE), `@AvichayAdraee` (AR) on X; IDF Telegram channels; `idf.il` press releases. |
| **Corroboration sources** | Reuters flash wire; AP wire; Al Jazeera English/Arabic; `@AbuAliExpress`. |
| **Threshold band** | `alert` (Tier-1 authoritative; can alert alone with high confidence). |

#### A3. Foreign-wire first-mention of kinetic event in Israel/region

| | |
|---|---|
| **What it indicates** | Event has happened; censor has not yet cleared domestic publication. Foreign wires will often be 5–30 minutes ahead. |
| **Keywords / patterns** | EN flash patterns: `BREAKING`, `explosions reported in`, `Israeli strike on`, `projectiles fired from`, `missiles toward`, `drone intercepted over`. AR: `انفجار`, `غارة`, `قصف`. |
| **Primary sources** | Reuters wire (`reuters.com/world/middle-east`, Reuters Twitter flash); AP Middle East wire; AFP Middle East; Al Jazeera Breaking. |
| **Corroboration sources** | IDF Spokesperson (for official confirmation); Abu Ali Express (for Arabic-sphere confirmation); local eyewitness social (Tier-3, only as claim). |
| **Threshold band** | `watch` on single-wire, `alert` on two-wire co-occurrence OR wire + IDF confirmation within 10 min. |
| **False-positive filters** | Retrospective / historical reporting ("on this day in…"); analysis pieces with "could" / "might" / "analysts warn"; exclude if publication date > 2h old. |

### Cue Class B — Escalation pre-signals (hours to 48h out)

#### B1. Airspace / NOTAM changes

| | |
|---|---|
| **What it indicates** | State-level decision that airspace is about to be used or contested. Frequently leads kinetic events by hours. |
| **Keywords / patterns** | EN: `NOTAM`, `airspace closed`, `restricted airspace`, `prohibited area`, `ZLLL`, `LLLL` (ICAO FIR codes for the region — `OLBB` Beirut, `OSTT` Damascus, `LLLL` Tel Aviv, `OIIX` Tehran, `OYSC` Sanaa). |
| **Primary sources** | FAA NOTAM feed; EASA conflict-zone bulletins; Eurocontrol EAD; OPSGROUP bulletins; `notams.aim.faa.gov`. |
| **Corroboration sources** | The War Zone reporting; airline route advisories; FlightRadar24 / ADS-B Exchange route deviation patterns. |
| **Threshold band** | `watch` on a single regional NOTAM change; elevates to `alert` if multiple adjacent FIRs change simultaneously. |

#### B2. Airline route changes / suspensions

| | |
|---|---|
| **What it indicates** | Carriers have received briefings or made their own threat assessments; often a public-facing leading indicator. |
| **Keywords / patterns** | EN: `suspends flights to`, `cancels Tel Aviv`, `reroutes around`, `avoiding Iranian airspace`, `crew positioning`. Airline names: `Lufthansa`, `United`, `Delta`, `Air France`, `BA`, `Emirates`, `ElAl`, `Arkia`. |
| **Primary sources** | Airline press releases / IR pages; Reuters aviation desk; OPSGROUP; FlightAware / FlightRadar24 anomaly detection. |
| **Corroboration sources** | Calcalist / Globes (domestic aviation reporting); Ynet economy desk. |
| **Threshold band** | `watch` on single-carrier; elevates to `alert` on ≥3 carriers within 24h. |

#### B3. US naval / air asset movements

| | |
|---|---|
| **What it indicates** | Repositioning of US combat power — signals US threat assessment and deters/enables specific actors. |
| **Keywords / patterns** | EN: `carrier strike group`, `CSG`, `CENTCOM repositions`, `B-2`, `B-52`, `F-22`, `F-35A`, `Patriot battery`, `THAAD`, `Gerald R. Ford`, `Eisenhower`, `Roosevelt`, `EastMed`, `Red Sea`. |
| **Primary sources** | `@CENTCOM` X / press releases; DoD press briefings; The War Zone; USNI News; Planet Labs / commercial-satellite-derived reporting. |
| **Corroboration sources** | ISW daily updates; Al Jazeera English; Reuters defense. |
| **Threshold band** | `watch`. Elevates when CENTCOM issues out-of-cadence statement. |

#### B4. Embassy / travel advisory changes

| | |
|---|---|
| **What it indicates** | Allied governments have private threat intelligence and are acting on it. Typically lags slightly behind private briefings. |
| **Keywords / patterns** | EN: `travel advisory`, `do not travel`, `reconsider travel`, `authorized departure`, `ordered departure`, `shelter in place`. HE: `הנחיות נסיעה`. |
| **Primary sources** | US State Dept travel.state.gov; UK FCDO gov.uk; French diplomatie.gouv.fr; German auswaertiges-amt.de; Canadian, Australian equivalents. |
| **Corroboration sources** | Reuters; AP; Axios (US foreign-policy angle). |
| **Threshold band** | `watch` on single-country upgrade; `alert` on ≥2 Five Eyes countries moving in step within 48h. |

#### B5. Cabinet / Security Cabinet convenings

| | |
|---|---|
| **What it indicates** | Israeli political leadership has determined a decision is imminent. Out-of-hours convenings especially informative. |
| **Keywords / patterns** | HE: `קבינט`, `קבינט מדיני-ביטחוני`, `ישיבת חירום`, `התכנסות דחופה`. EN: `Security Cabinet`, `emergency cabinet`, `cabinet convenes`. |
| **Primary sources** | PMO press releases (gov.il/pmo); Kan News; Haaretz politics desk; Ynet. |
| **Corroboration sources** | Times of Israel; Axios Tel Aviv reporting. |
| **Threshold band** | `watch`; elevates if convening is out-of-hours or unscheduled. |

#### B6. Reservist call-up indicators

| | |
|---|---|
| **What it indicates** | IDF is expanding operational tempo. Historically a strong correlate of escalation. |
| **Keywords / patterns** | HE: `צו 8`, `צו שמונה`, `מילואים`, `גיוס מילואים`, `קריאה למילואים`. EN: `Tzav 8`, `reservist call-up`, `IDF mobilization`. |
| **Primary sources** | IDF Spokesperson; Kan; Ynet; Haaretz military correspondent (Amos Harel etc.); `@AbuAliExpress` translations. |
| **Corroboration sources** | Reuters Jerusalem; Times of Israel. |
| **Threshold band** | `watch`; elevates to `alert` on confirmed brigade-level+ call-up. |

#### B7. Abnormal market moves

| | |
|---|---|
| **What it indicates** | Financial markets sometimes lead on quiet policy shifts or non-public information leaks. |
| **Keywords / patterns** | Numeric thresholds: shekel ≥1.5% intraday move; TA-35 ≥2% intraday; CDS spread widening ≥20bp; oil Brent ≥3% intraday with regional attribution. |
| **Primary sources** | Bank of Israel rates; TASE feed; Bloomberg / Reuters markets wire; Calcalist; Globes. |
| **Corroboration sources** | Airline / insurer statements (see B2, B4). |
| **Threshold band** | `watch` only; market-derived cues never alert alone (too many non-security drivers). |

### Cue Class C — Adversary posture

#### C1. Hezbollah / Houthi / Iraqi militia / Iranian-proxy claims

| | |
|---|---|
| **What it indicates** | Adversary-side announcement of operation, ultimatum, or "attack video". Shapes expected response. |
| **Keywords / patterns** | AR: `حزب الله`, `المقاومة الإسلامية`, `أنصار الله`, `كتائب حزب الله`, `بيان عسكري`, `عملية`. EN translations: `Hezbollah statement`, `Houthi claim`, `Kataib Hezbollah`, `Islamic Resistance in Iraq`. |
| **Primary sources** | `al-Manar` (Hezbollah outlet); `al-Masirah` (Houthi); Telegram militia channels (via Abu Ali Express synthesis, not raw); `@AbuAliExpress`; SITE Intelligence Group summaries. |
| **Corroboration sources** | Al Jazeera Arabic; ISW Iran Update; Alma Research Center (northern front). |
| **Threshold band** | `watch` on single-source claim; elevates to `alert` on corroboration by Tier-1 Israeli/US source or physical effects reporting. |

#### C2. Iranian state-media tone shift

| | |
|---|---|
| **What it indicates** | Shift from generic denunciation to specific threat language suggests decision-making escalation in Tehran. |
| **Keywords / patterns** | FA: `انتقام`, `رد کوبنده`, `پاسخ قاطع`. AR translations: `انتقام`, `رد حاسم`. EN: `decisive response`, `punishing retaliation`, `axis of resistance will respond`, named-official threats from Khamenei / IRGC commanders. |
| **Primary sources** | IRNA; Tasnim; Fars; Press TV (English); Khamenei.ir. |
| **Corroboration sources** | ISW Iran Update; Al-Monitor Iran desk; Reuters Tehran. |
| **Threshold band** | `watch`. Elevates if *named senior figure* (Supreme Leader, IRGC commander, Foreign Minister) issues specific threat. |

#### C3. Imagery / movement at adversary sites

| | |
|---|---|
| **What it indicates** | Physical preparation at known facilities (missile sites, drone launch pads, naval assets). |
| **Keywords / patterns** | EN: `satellite imagery shows`, `new construction at`, `activity at`, `missile fueling`, `TEL deployment`. Site names: specific facility names tracked by Alma / ISW / FDD. |
| **Primary sources** | Alma Research Center reports; ISW map updates; The War Zone imagery; FDD; Planet Labs / Maxar releases. |
| **Corroboration sources** | Reuters defence; Israeli security correspondents. |
| **Threshold band** | `watch`; rarely alerts alone (imagery interpretation has lag and false positives). |

### Cue Class D — Diplomatic escalation / de-escalation

#### D1. White House / US posture

| | |
|---|---|
| **What it indicates** | US political signaling; presence/absence of Israel/Iran mentions in readouts is itself informative. |
| **Keywords / patterns** | EN: `White House readout`, `POTUS spoke with`, `Israel`, `Iran`, `Gaza`, `Lebanon`, `ceasefire`, `restraint`, `right to defend`. |
| **Primary sources** | whitehouse.gov press briefings / readouts; `@WhiteHouse`, `@PressSec` on X; State Department press briefings. |
| **Corroboration sources** | Axios (Barak Ravid); Reuters Washington. |
| **Threshold band** | `watch`; elevates on unscheduled POTUS statement or presidential-level phone call readout. |

#### D2. Mediator statements

| | |
|---|---|
| **What it indicates** | Qatar/Egypt/Oman channels opening or closing signals backchannel movement. |
| **Keywords / patterns** | EN: `Qatar mediator`, `Egyptian intelligence`, `Omani channel`, `back-channel`, `hostage talks`, `ceasefire framework`. AR: `وساطة قطرية`, `الوساطة المصرية`. |
| **Primary sources** | Qatari MoFA; Egyptian State Information Service; Al Jazeera Arabic; Axios. |
| **Corroboration sources** | Reuters; Times of Israel diplomatic correspondent. |
| **Threshold band** | `watch`. |

#### D3. UN Security Council actions

| | |
|---|---|
| **What it indicates** | Emergency sessions / draft resolutions signal diplomatic-track escalation. |
| **Keywords / patterns** | EN: `UNSC emergency session`, `draft resolution`, `veto`, `S/RES/`. |
| **Primary sources** | UN press releases; `@UN` feeds. |
| **Corroboration sources** | Reuters UN; AFP UN. |
| **Threshold band** | `watch`. |

### Consolidated matrix (for collector implementation)

| Cue | Class | Band (solo) | Primary source type | Corroboration required? |
|---|---|---|---|---|
| A1 HFC alert | A | `critical` | Structured API | No |
| A2 IDF Spox | A | `alert` | Official social + site | No |
| A3 Foreign wire kinetic | A | `watch` → `alert` | News API / RSS | Yes (2 wires or wire+IDF) |
| B1 NOTAM | B | `watch` | Structured feed | No (for `watch`) |
| B2 Airline route | B | `watch` | Press releases + flight data | Yes for `alert` |
| B3 US naval/air | B | `watch` | Official + imagery | No |
| B4 Travel advisory | B | `watch` | Government sites | Yes for `alert` |
| B5 Cabinet convene | B | `watch` | PMO + press | No |
| B6 Reservist call-up | B | `watch` → `alert` | IDF + press | Yes for `alert` |
| B7 Market move | B | `watch` | Market data | Never alerts alone |
| C1 Militia claim | C | `watch` | Adversary outlets + synthesis | Yes for `alert` |
| C2 Iran tone | C | `watch` | Iranian state media | No |
| C3 Imagery | C | `watch` | OSINT specialists | Rarely alerts |
| D1 White House | D | `watch` | whitehouse.gov + wires | No |
| D2 Mediators | D | `watch` | MoFAs + Axios | No |
| D3 UNSC | D | `watch` | UN press | No |

---

## Part B — Delivery and Orchestration Architecture

The delivery-mechanics exchange fixed channels, formats, and thresholds. This part pins down *where code runs*, *how transports combine*, and *how state moves*.

### B.1 Architectural shape: hybrid (serverless for collection, always-on for state)

Neither pure-serverless nor pure-always-on is right. The components have different runtime profiles:

| Component | Runtime profile | Best home |
|---|---|---|
| Scheduled collectors (poll RSS, wire APIs, NOTAM, gov sites) | Stateless, run every N minutes, idempotent | **Serverless** (Cloudflare Workers / AWS Lambda / Google Cloud Functions) with cron triggers |
| Webhook receivers (HFC push if available, Telegram webhooks) | Event-driven, stateless | **Serverless** |
| Event store / dedup state / cadence state | Stateful, shared across invocations | **Managed DB** (Cloudflare D1 / DynamoDB / Supabase) |
| Synthesizer (LLM calls for cue extraction, triangulation, SITREP drafting) | Stateless but heavy, rate-limit aware | **Serverless** with queue in front |
| Transport dispatchers (Pushover, Telegram, SMTP, SMS) | Stateless, fan-out | **Serverless** |
| Uptime pinger / dead-man's switch | External, independent of the pipeline itself | **Third-party** (Healthchecks.io, Better Stack, or separate serverless in different provider) |
| Operator CLI / "mute for N hours" control plane | Interactive, low-frequency | **Telegram bot** commands, processed by serverless webhook |

**Rationale for not running the whole thing on a home VM**: a home VM is a single point of failure on residential internet, subject to power/ISP outages at exactly the wrong moment. The pipeline should survive the operator's house losing power — because that's correlated with the events the pipeline exists to detect.

**Rationale for not going fully serverless**: cadence state and dedup state need a durable store that outlives invocations; and the synthesizer's LLM budget must be governed centrally.

### B.2 Component map

```
┌─────────────────────────────────────────────────────────────────────┐
│                       SCHEDULED TRIGGERS                            │
│  (cron-like: collectors fire at signal-appropriate cadences)        │
└───────────────────┬─────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    COLLECTORS (serverless)                          │
│  One per source family: IDF, HFC, wires, NOTAM, gov advisories,     │
│  market data, adversary media, UN, White House, etc.                │
│  Output: raw_item records → queue                                   │
└───────────────────┬─────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     RAW-ITEM QUEUE                                  │
└───────────────────┬─────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│              MATCHER + CUE EXTRACTOR (serverless)                   │
│  Rule-based keyword/pattern matching on raw_item → cue_detection   │
│  Falls back to LLM for ambiguous items (flagged "reduced            │
│  confidence" if synthesizer degraded).                              │
└───────────────────┬─────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│              EVENT AGGREGATOR (serverless + DB)                     │
│  Dedup, event-identity hashing, corroboration tracking.             │
│  Emits: nothing, watch, alert, or critical.                         │
│  Updates: cadence state, event store.                               │
└───────────────────┬─────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│              DISPATCHERS (serverless fan-out)                       │
│  Pushover (primary alerts) · Telegram (digest + audit trail         │
│  + failover) · SMTP (SITREP email) · SMS (last-resort).             │
│  Each dispatcher writes delivery receipts to event store.           │
└───────────────────┬─────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│              EXTERNAL UPTIME PINGER                                 │
│  Independent service. If pipeline goes silent without liveness,    │
│  pages operator via SMS.                                            │
└─────────────────────────────────────────────────────────────────────┘
```

### B.3 Transport roles — which surface is authoritative for what

Re-affirming and extending the delivery-mechanics exchange:

| Transport | Authoritative for | Never carries |
|---|---|---|
| **Pushover** | Real-time bands: `watch`, `alert`, `critical`. Sound-differentiated. | Long-form content; digests; SITREPs. |
| **Telegram (private bot channel)** | Daily digest; full audit trail of every push; interactive control ("/mute 2h"); failover alert path when Pushover degraded. | First-line alerts (too easy to mute Telegram accidentally). |
| **Email (SMTP)** | Long-form **SITREP reports** — end-of-day, weekly, or on-demand. Archive of record. | Time-sensitive alerts. Email is *not* an alert surface. |
| **SMS** | `critical` band failover when both Pushover and Telegram have failed; pipeline-liveness notifications from external pinger. | Anything routine. SMS budget is scarce and attention-expensive. |

**Email's specific role**: SITREPs (see Part C) are structured long-form documents meant to be read carefully, archived, and re-read. Email is the correct surface — it threads, archives, and is searchable. A daily end-of-day SITREP and an on-demand "escalation SITREP" are the two email products.

### B.4 Scheduling, state, and retries

#### Scheduling
- Each collector has its own cadence, stored in DB and mutable by the cadence controller (adaptive-frequency exchange covers this).
- Baseline cadences (quiet mode):
  - HFC feed: 30s
  - IDF Spokesperson: 60s
  - Foreign wires: 2 min
  - NOTAM: 10 min
  - Travel advisories: 30 min
  - Cabinet/political: 5 min
  - Market data: 5 min during market hours
  - Adversary media: 5 min
  - UN / White House: 10 min
- Heightened-mode cadences compress by 3–10x per the adaptive-frequency plan.

#### State
- **Event store** (DB table): event_id, first_seen_utc, last_updated_utc, cue_class, sources_corroborating[], confidence, band_emitted, delivery_receipts{}.
- **Cadence state**: current mode, last_mode_change_utc, justification, guardrail counters.
- **Dedup index**: rolling hash window of recent cue signatures.
- **Mute state**: active/inactive, expiry_utc, reason.

#### Retries
- Collectors: exponential backoff (1s, 2s, 4s, 8s; max 5 attempts). Failure logged to digest.
- Dispatchers: Pushover retry 3x, then failover to Telegram. Telegram retry 3x, then failover to SMS *for `critical` only*. Email retry with standard SMTP backoff.
- **Idempotency**: every outbound push carries a deterministic client-side idempotency key (event_id + band + update_seq) so duplicate dispatch from retry does not produce duplicate notifications.

### B.5 Security and least privilege

- Each serverless function has a scoped secret: only the Pushover dispatcher holds the Pushover token; only the email dispatcher holds SMTP credentials; collectors hold only read-only API keys.
- Operator control-plane commands (`/mute`) authenticated by a shared secret inside the Telegram bot's verified chat_id — no public commands.
- Private channel IDs, phone numbers, and the operator's location (for HFC zone matching) live in a single `operator_profile` record, encrypted at rest.

---

## Part C — Baked-in Conventions

These conventions apply everywhere the pipeline produces output. They are cheap to enforce from day one and expensive to retrofit.

### C.1 Timestamps

**Rule**: every timestamp is rendered as **UTC primary, Israel local time in brackets**.

Format: `YYYY-MM-DDTHH:MMZ (HH:MM IDT)` or `(HH:MM IST)` depending on season.

Examples:
- `2026-04-19T18:07Z (21:07 IDT)`
- `2026-01-12T06:30Z (08:30 IST)`

Rationale:
- UTC is the unambiguous reference for correlation across sources (wires, CENTCOM, UN all use UTC or easy UTC derivatives).
- Israel local is the operator's cognitive frame; always shown in brackets so the operator doesn't do arithmetic at 3am.
- The `Z` suffix makes the UTC assertion explicit and machine-parseable.
- `IDT` / `IST` distinguishes daylight time — never use the ambiguous bare "Israel time".

**Storage**: always UTC in DB and JSON. **Rendering**: dual-format at presentation time.

### C.2 Filenames

**Rule**: ISO date, 24-hour time, no spaces, hyphen-separated.

Format: `YYYY-MM-DD-HHMM-{slug}.{ext}` or `YYYY-MM-DDTHHMMZ-{slug}.{ext}` when sub-day precision matters.

Examples:
- `2026-04-19-2107-sitrep-western-galilee.md`
- `2026-04-19T1807Z-alert-event-e4f9a2.json`
- `2026-04-19-daily-digest.md`

Rationale:
- Sorts correctly lexicographically, without special tooling.
- Copy-pasteable into shell commands.
- Year-first prevents collisions across years in a flat archive.

### C.3 Identifiers

- **Event IDs**: short hex (first 8 chars of SHA-256 over `cue_class + location_bucket + time_bucket`). Stable across corroborating detections of the same event.
- **Message IDs**: `{event_id}-{band}-{seq}`. Used as Pushover/Telegram idempotency key.
- **Source IDs**: matches the registry — lowercase, underscore-separated (`idf_spokesperson_en`, `reuters_me_wire`, `hfc_alert_feed`).

### C.4 Source bias and credibility ratings

Each source in the registry carries two ratings, reviewed quarterly:

- **Bias** (editorial lean): `left | center-left | center | center-right | right | state-aligned | adversary-aligned | n/a`.
- **Credibility** (track record on factual claims in this domain): `A` (consistently reliable), `B` (generally reliable with known caveats), `C` (mixed — verify), `D` (use as claim-feed only), `F` (excluded).

These ratings appear in every SITREP alongside every source citation. They are not hidden metadata.

Example:
- `IDF Spokesperson (state-aligned, A)`
- `Al Jazeera Arabic (regional editorial line, B)`
- `0404 Telegram (aggregator, D — claim-feed)`

Rationale: the operator and any future reader of the archive should see at a glance whether a claim rests on a reliable source, and whether the framing is colored.

### C.5 SITREP template

SITREP (Situation Report) is the right long-form template because:

1. **Facts-first, narrative-last.** Military and intelligence SITREPs lead with what is known, where, and when, before interpretation. This resists the temptation to spin.
2. **Source transparency built in.** Credibility grading is a standard SITREP field — a discipline borrowed directly from the **Admiralty Code** / NATO STANAG 2022 (source reliability A–F × information credibility 1–6). This is exactly what the operator needs in a disinformation-rich environment.
3. **Triangulation discipline.** SITREPs require corroboration across sources before upgrading a claim from "reported" to "confirmed". Without this structure, claim-feeds contaminate the record.
4. **Known genre.** Anyone with OSINT / journalism / military background can read a SITREP and immediately parse it. Shared genre is a tax on noise.

#### SITREP template (markdown, for email delivery)

```markdown
---
sitrep_id: {YYYY-MM-DD-HHMM-slug}
issued: 2026-04-19T18:07Z (21:07 IDT)
classification: OSINT — public sources only
scope: {e.g., Israel — northern front | Israel — regional | on-demand: {trigger}}
period_covered: 2026-04-19T12:00Z — 2026-04-19T18:00Z (15:00 — 21:00 IDT)
pipeline_mode: heightened
---

# SITREP — {short title}

## 1. Bottom Line Up Front (BLUF)

One to three sentences. What is the situation, what changed in the reporting period,
what does the operator need to know. No analysis beyond what the evidence supports.

## 2. Confirmed Facts

Each item: timestamp (UTC/IDT), fact, source(s) with (bias, credibility) tags,
Admiralty-code rating.

- 2026-04-19T17:42Z (20:42 IDT) — IDF Spokesperson confirms interception of
  projectile fire from Lebanon toward Western Galilee.
  Source: IDF Spokesperson EN (state-aligned, A). Admiralty: A-1.

- 2026-04-19T17:48Z (20:48 IDT) — Reuters wire confirms interception; no casualties
  reported.
  Source: Reuters ME wire (center, A). Admiralty: A-1.

## 3. Reported but Unconfirmed

Claims from lower-credibility or uncorroborated sources. Explicitly tagged as claim.

- 2026-04-19T17:50Z — Telegram aggregator reports second projectile salvo toward
  the same area. No Tier-1 confirmation at issue time.
  Source: 0404 Telegram (aggregator, D). Admiralty: E-3 (claim-feed).

## 4. Adversary Signaling

Statements, claims of responsibility, propaganda output relevant to the period.

- 2026-04-19T17:55Z — Hezbollah-affiliated channel issues statement claiming
  responsibility for projectile fire; specific framing: "response to" (prior
  Israeli action).
  Source: al-Manar / Abu Ali Express translation (adversary-aligned / synthesis, B).
  Admiralty: B-2.

## 5. Allied / Diplomatic Activity

White House, CENTCOM, State Dept, FCDO, mediator channels.

- (None in reporting period.)

## 6. Triangulation Notes

Explicit record of which claims were cross-checked against which sources. Where
Tier-1 corroboration is *missing* for a claim carried in Section 3, say so.

- Section 2 bullets corroborated across IDF Spokesperson + Reuters + Abu Ali Express.
- Section 3 second-salvo claim: NOT corroborated by any Tier-1 source as of issue
  time. Retained as watch-item only.

## 7. Assessment (Clearly Labeled as Interpretation)

Short, hedged, and explicitly separated from fact. Uses confidence language
(*low / moderate / high*), not certainty language.

- Moderate confidence: the incident fits the pattern of ongoing cross-border
  exchanges rather than signaling a qualitative escalation. Watch for follow-on
  IDF retaliatory reporting and further Hezbollah framing over the next 12–24h.

## 8. Open Questions

What the pipeline could not resolve this cycle.

- Whether projectile type was rocket, drone, or mortar — not yet publicly specified.
- Whether Home Front Command alert zones were expanded beyond initial footprint.

## 9. Sources Consulted This Cycle

Full source list with bias/credibility tags and fetch status.

- IDF Spokesperson EN — (state-aligned, A) — fetched OK.
- Reuters ME wire — (center, A) — fetched OK.
- Al Jazeera Arabic — (regional line, B) — fetched OK.
- Abu Ali Express — (synthesis, B) — fetched OK.
- 0404 Telegram — (aggregator, D) — fetched OK, treated as claim-feed.
- Home Front Command feed — (official, A) — fetched OK; no active alert at issue time.

## 10. Pipeline Self-Report

- Current mode: heightened (entered 2026-04-19T11:30Z)
- Collectors healthy: 14 / 15
- Degraded: `jpost_rss` (HTTP 503 at last fetch; retrying)
- Synthesizer: healthy; no fallback active in this cycle.

---

*Issued by the Israel OSINT situational-awareness pipeline. AI-assisted synthesis
over public sources. Not for operational decision-making in isolation.*
```

#### Admiralty Code quick reference (to cite in SITREPs)

| Reliability | Credibility |
|---|---|
| A — Completely reliable | 1 — Confirmed by other sources |
| B — Usually reliable | 2 — Probably true |
| C — Fairly reliable | 3 — Possibly true |
| D — Not usually reliable | 4 — Doubtful |
| E — Unreliable | 5 — Improbable |
| F — Reliability cannot be judged | 6 — Truth cannot be judged |

Sources tagged as e.g. `A-1` (IDF confirming an event that Reuters also confirms) or `D-3` (aggregator Telegram, possibly true) make the basis of every claim auditable.

### C.6 Consistency enforcement

- A single `render_timestamp()` helper is the only code path that produces human-readable timestamps. No ad-hoc formatting.
- A single `render_source_tag()` helper produces `Name (bias, credibility)` strings from the registry.
- SITREP generation uses a fixed template file; the synthesizer fills sections but does not alter structure.
- A linter on outbound email and digest content rejects timestamps without the `Z (HH:MM IDT/IST)` dual form.

---

### Open items for subsequent exchanges

- Concrete collector code skeletons per source family (serverless runtime choice, rate-limit handling, parser contracts).
- Cost model for the hybrid serverless + DB + LLM architecture under quiet vs. heightened cadence.
- Admiralty-code automation: can the synthesizer assign A–F / 1–6 ratings reliably, or does this need human-in-the-loop review for first N weeks?
- Backfill: ingesting the archive of prior SITREPs into a searchable store so the pipeline can reference "last week the Cabinet also convened out-of-hours" without operator memory.

### Sources

Methodological synthesis. References:

- Prior exchanges: `outputs/individual/2026-04-19-situational-awareness-scope.md`; `outputs/individual/2026-04-19-source-selection-monitoring.md`; `outputs/individual/2026-04-19-delivery-mechanics.md`; `outputs/individual/2026-04-19-adaptive-frequency-implementation.md`.
- `CONTEXT.md` — known sources of truth and domain framing.
- Pushover API documentation (priority levels, sounds) — [pushover.net/api](https://pushover.net/api).
- Admiralty Code / NATO STANAG 2022 source reliability and information credibility rating system — standard reference in intelligence reporting.
- US military SITREP conventions (e.g., FM 6-0 *Commander and Staff Organization and Operations*, Appendix D — situation reports) — format inspiration, not classification scheme.
- ICAO FIR codes and NOTAM formatting — public aeronautical reference.
