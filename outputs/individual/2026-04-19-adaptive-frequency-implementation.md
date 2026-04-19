---
thread: 2026-04-19-adaptive-frequency-implementation
topic: Always-on implementation with autonomous frequency calibration
started: 2026-04-19
exchange_count: 1
---

> **Note**: This output was produced through AI-assisted research using Claude Code. It is a planning artifact, not an operational tool.

# Ideal Implementation — Always-On With Adaptive Frequency

## Exchange 1

**Summary**: Design an always-running situational awareness pipeline that auto-tunes its own cadence up during heightened threat environments and back down when conditions normalize — with guardrails against runaway self-escalation.

### Key Findings

#### 1. Design premise: the pattern recurs, so the system should be standing, not ad-hoc

Recurring regional conflict means the operator repeatedly spins up ad-hoc monitoring during each crisis and lets it decay between. That pattern is wasteful and also dangerous: in the first hours of a new escalation the operator is least prepared and most distracted, exactly when the monitoring system should already be running and already tuned.

**A standing pipeline, always on, always running at least once per 24 hours in peacetime**, solves two problems:

1. It removes the cold-start tax at the moment of a new crisis.
2. It produces a **baseline** — the operator learns what "normal" looks like across dozens of quiet days, which is what makes deviation detection possible.

#### 2. Architecture overview

Four components, deliberately decoupled:

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐     ┌─────────────┐
│  Scheduler  │────▶│  Collectors  │────▶│ Synthesizer  │────▶│  Notifier   │
└─────────────┘     └──────────────┘     └──────────────┘     └─────────────┘
       ▲                    │                    │                    │
       │                    ▼                    ▼                    ▼
       │              ┌──────────┐         ┌──────────┐         ┌──────────┐
       └──────────────│  State   │◀────────│ Event DB │         │ Delivery │
    cadence feedback  │  store   │         │          │         │ channels │
                      └──────────┘         └──────────┘         └──────────┘
```

- **Scheduler** — decides when the next sweep runs. Reads current mode from the state store. Owns the adaptive-frequency logic (§4).
- **Collectors** — one per source (or source family). Stateless, cheap, independently retryable. Output normalized items to the event DB.
- **Synthesizer** — reads recent items, deduplicates, classifies into cue classes, decides whether thresholds are crossed, and produces digest + alert content. This is the only component that uses LLM reasoning.
- **Notifier** — reads pending notifications, dispatches to the delivery channels defined in the delivery-mechanics exchange, handles fallbacks and cool-downs.
- **State store** — current mode (`quiet` / `watch` / `heightened` / `active`), mode-entry timestamp, recent cue history, alert fatigue stats.
- **Event DB** — append-only log of everything collected, keyed by event-identity hash. Cheap SQLite or Postgres is enough.

Deliberate properties of the architecture:

- **Each component can be killed and restarted without losing state** — state is in the store, not in process memory.
- **Collectors are independent** — an IDF Spokesperson fetch failing does not block Reuters.
- **The synthesizer is idempotent** — running it twice on the same window produces the same output (important for recovery).
- **The scheduler is the only thing that knows "what time is it"** — collectors and synthesizer don't run clocks themselves.

#### 3. Where it runs

This is a single-operator system, so keep the substrate boring:

- A small always-on host: home server, Raspberry Pi, or a $5–10/month cloud VM. The operator's existing Ubuntu workstation is acceptable if uptime is good; a dedicated host is better.
- LLM calls go to a hosted API (Anthropic, OpenAI) — synthesizer work is modest per sweep and bursty during escalation.
- Event DB is SQLite on local disk. Daily backups to object storage.
- External uptime pinger (UptimeRobot, Healthchecks.io free tier) watches the scheduler heartbeat and SMSes the operator if it stops.

Explicitly avoided:

- Kubernetes, queues, distributed workers — overkill for one operator.
- Self-hosted LLMs — achievable but a distraction from the actual problem.
- A web UI — Telegram digest + file-system logs are the UI.

#### 4. Adaptive frequency — the heart of the system

The scheduler picks the next sweep interval based on a single scalar, the **threat level**, maintained in the state store. Threat level is a number in [0, 1] with discrete mode bands:

| Threat level | Mode | Sweep interval | Notifier posture |
|---|---|---|---|
| 0.0 – 0.2 | `quiet` | 6 hours (or once/24h floor) | Digest only |
| 0.2 – 0.4 | `watch` | 60 minutes | Digest + `watch` pushes enabled |
| 0.4 – 0.7 | `heightened` | 15 minutes | `alert` band enabled |
| 0.7 – 1.0 | `active` | 3–5 minutes | `critical` band enabled |

##### Inputs that raise threat level

- Cue Class B signal observed → small additive bump (+0.05 each, capped).
- Cue Class C signal observed → larger bump (+0.10).
- A confirmed Cue Class A (kinetic) event → jump to at least `heightened`.
- Home Front Command firing → jump to `active`.
- Baseline-deviation detector: count of region-related Tier-1 wire items per hour, compared to 30-day baseline. +2σ adds; +3σ jumps a mode.
- Manual operator override: operator can force a mode up (not down) via a command.

##### Inputs that lower threat level

- **Time decay only** — threat level decays toward baseline at a fixed rate per hour (e.g. –0.02/hr in `watch`, –0.05/hr once signals go absent). This is the single most important guardrail: the system cannot talk itself out of heightened mode, only time can.
- Explicit de-escalation signals (ceasefire announcement confirmed by multiple Tier-1 sources) accelerate decay but cannot fully collapse the level immediately.
- A manual operator "stand down" command drops the level to `quiet` floor — logged loudly in the digest.

##### Guardrails against runaway escalation

The failure mode to fear: the system's own alerts become inputs to its own threat assessment, producing a feedback loop where it convinces itself the world is ending from yesterday's alerts.

- **Synthesizer never reads its own output.** The threat-level function reads only collected source items, never prior alerts or prior digests.
- **Bounded rate of change.** Threat level cannot rise by more than 0.3 in any 15-minute window unless a Tier-1 kinetic confirmation fires. This prevents cascade from correlated soft signals.
- **Source independence check.** Multiple signals from the same upstream source (e.g. three X accounts all re-sharing one Reuters item) count as one signal. The synthesizer resolves to origin before scoring.
- **Sanity ceiling on duration at `active`.** If the system remains at `active` for > 24h without a fresh Tier-1 kinetic confirmation, it forcibly de-escalates one mode and logs why. The operator can re-escalate manually if the real world justifies it.
- **Hysteresis.** Mode transitions require the threshold to be crossed for a sustained period (e.g. 10 minutes for upward transitions into `heightened`; 60 minutes for downward transitions out of it). This prevents flapping.

##### Cadence floor and ceiling

- **Floor**: at least one sweep per 24 hours, even in `quiet`. The pipeline's job includes proving it is still alive.
- **Ceiling**: no faster than one sweep per 3 minutes, regardless of mode. Faster than that wastes source quotas and produces noise not signal.

#### 5. State model in concrete terms

The state store holds, at minimum:

```yaml
mode: heightened
threat_level: 0.52
mode_entered_at: 2026-04-19T14:38:00+03:00
last_sweep_at: 2026-04-19T21:35:00+03:00
next_sweep_at: 2026-04-19T21:50:00+03:00
recent_cues:
  - {class: B, source: flightaware, weight: 0.05, at: 2026-04-19T17:42:00+03:00}
  - {class: C, source: abu_ali_express, weight: 0.10, at: 2026-04-19T19:14:00+03:00}
  - {class: A, source: idf_spokesperson, weight: 0.40, at: 2026-04-19T21:07:00+03:00}
decay_rate_per_hour: 0.05
manual_overrides:
  active: false
fatigue:
  alerts_last_7d: 6
  false_positives_last_30d: 1
```

This structure is small, human-readable, and diffable. The operator can open it at any time and see why the system believes what it believes.

#### 6. What runs once per day no matter what

Even in `quiet`, the daily sweep does real work:

1. Pull a standard list of Tier-1 feeds (IDF Spokesperson, Home Front Command status, CENTCOM, ISW latest update, Al-Monitor top items, Reuters Israel topic).
2. Compute baseline statistics for the deviation detector (rolling 30-day counts per source).
3. Run the synthesizer to produce a terse "nothing significant" digest.
4. Emit a heartbeat to the external pinger.
5. Rotate and back up the event DB.

The daily sweep exists partly for awareness, partly to keep the operator's trust — a system that reports in when there's nothing to report is a system the operator keeps believing when it suddenly has something to say.

#### 7. Buildability — what is real now vs. aspirational

##### Real now (can be built in a weekend, iterated over weeks)

- Scheduler + state store + SQLite event DB — trivial.
- Collectors for: RSS feeds, X via API or RSS-bridge, Home Front Command public feeds, wire-service RSS, Telegram public channels via Telethon. Each collector is 50–150 lines.
- Synthesizer v1: rule-based cue detection + LLM call for digest/alert wording only. The *decision* to fire should not be LLM-gated in v1; it should be deterministic and inspectable.
- Notifier via Pushover + Telegram bot. Both have simple HTTP APIs.
- Adaptive frequency via the scalar threat level, decay, and guardrails described above.

##### Aspirational / later

- LLM-gated cue classification (replacing rule-based cue detection). Only worth doing after the rule-based version has been live long enough to produce a labelled corpus.
- Audio ingestion of Reshet Bet hourly bulletins (transcript → cue extraction).
- Satellite / imagery fusion (ISW / Alma already do this externally; no reason to rebuild).
- Reliability scoring per source that auto-updates from monthly operator review outcomes.
- Multi-operator deployment (share digests, not alerts).

#### 8. Maintenance posture

Single-operator systems die from neglect, not from bugs. Countermeasures:

- **Monthly review (already specified in delivery exchange).** Review alert outcomes, tune thresholds.
- **Quarterly source audit.** Any source with zero contributions to fired alerts in 90 days is demoted or dropped. Any source responsible for > 20% of false positives is moved to Tier 3 or dropped.
- **Versioned config.** Thresholds, weights, and decay rates live in a config file checked into this repo's `private/`. Every change is a commit with a reason.
- **Explicit retirement path.** If the operator stops using the system for > 30 days, it daily-digests a "still running, still consuming X/mo, do you still want this?" reminder.

#### 9. What makes this good vs. another amateur OSINT bot

Most self-built OSINT monitors fail in one of three ways:

1. **They optimise for volume.** More sources, more pushes, more dashboards. This design optimises for silence.
2. **They have no decay model.** Threat level only goes up. This design makes decay the default and escalation the exception.
3. **They fuse "detection" and "delivery" into one LLM prompt.** Which means they can't be debugged. This design makes detection deterministic and inspectable; LLM work is confined to wording and summarization.

The test is not "does it catch everything". The test is: **over 12 months, does the operator spend fewer hours on news than before, while missing no event that they would have acted on?**

### Sources

Methodological synthesis drawing on the two prior exchanges and standard distributed-systems / SRE practice.

- Prior exchange: `outputs/individual/2026-04-19-situational-awareness-scope.md`.
- Prior exchange: `outputs/individual/2026-04-19-source-selection-monitoring.md`.
- Prior exchange: `outputs/individual/2026-04-19-delivery-mechanics.md`.
- Google SRE Book, chs. 6 (Monitoring) and 11 (On-Call) — on alert fatigue, hysteresis, dead-man's-switch design.
- Rob Ewaschuk, "My Philosophy on Alerting" — on conjunctive thresholds and false-positive discipline.
- Pushover, Telegram Bot, Healthchecks.io / UptimeRobot — standard transports and pingers referenced in the delivery exchange.
