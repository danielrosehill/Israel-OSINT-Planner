---
thread: 2026-04-19-delivery-mechanics
topic: Delivery mechanics for an Israel situational awareness pipeline
started: 2026-04-19
exchange_count: 1
---

> **Note**: This output was produced through AI-assisted research using Claude Code. It is a planning artifact, not an operational tool.

# Delivery Mechanics

## Exchange 1

**Summary**: Specify how a situational-awareness pipeline should actually reach the operator — channels, message formats, thresholds, de-duplication, and failure handling.

### Key Findings

#### 1. Two delivery surfaces, not one

Combining "silent daily digest" and "wake up, something is happening" into a single channel is the primary design failure mode. They have opposite requirements:

| Surface | Purpose | Latency target | Volume | Failure of this surface looks like |
|---|---|---|---|---|
| **Digest surface** | Baseline awareness — "here's what the region looks like" | Hours acceptable | 1–4 messages/day | Operator loses general context |
| **Alert surface** | Action-relevant signal — "something changed, possibly act" | Seconds to low-minutes | Zero on a calm day; never more than a handful on a bad day | Operator misses something material |

These must use **different transport channels, different sounds, different formats, and different thresholds**. If the digest and the alert both arrive via the same Pushover sound, the alert channel is already devalued.

#### 2. Channel selection

##### Primary alert channel: Pushover

Pushover is the right primary alert transport because it:

- Delivers reliably on mobile without a Telegram-style "social" surface.
- Supports **per-message priority** (0 normal, 1 high, 2 emergency-with-acknowledgement).
- Supports **distinct sounds per message**, which lets the operator recognise severity pre-cognitively.
- Has no inline-reply loop, which prevents the operator from getting sucked into a conversation during an event.

Configuration:

- Dedicated Pushover application token for this pipeline (not shared with other automations).
- Three sounds mapped to three priority bands:
  - `watch` (priority 0, distinctive but quiet) — pre-escalation cues worth noting.
  - `alert` (priority 1, loud) — confirmed escalation event.
  - `critical` (priority 2, emergency-with-ack) — reserved for Home Front Command–level ground truth or confirmed major kinetic events in the operator's region.

##### Secondary channel: Telegram bot

A private Telegram channel (bot-posted, operator-only) is useful as:

- The **digest surface** — longer-form daily/periodic summaries the operator reads at their own pace.
- An **audit trail** — every push emitted to Pushover is also mirrored to Telegram, timestamped, so the operator can reconstruct "what did the system say and when".
- A **failover path** — if Pushover API is down, Telegram still delivers.

##### Tertiary: SMS / email

- **SMS** via Twilio / local gateway: fallback for the `critical` band only, if Pushover and Telegram both fail to acknowledge within N seconds. SMS is the last-line option because it is the one channel that works through cellular when data networks degrade.
- **Email**: only for the daily digest archive. Not for alerts. Email inboxes are not alert surfaces.

##### Explicitly rejected channels

- **X / Twitter DM, Discord, Slack** — social attention sinks; rejected for the same reason the pipeline exists.
- **Phone call robocall** — high false-positive cost, and the pipeline should not pretend to be an emergency service.
- **Smartwatch-only** — acceptable as a mirror of Pushover, never as the sole surface.

#### 3. Message formats

Message formats are tight and fixed. A template is not decoration; it is what makes the operator able to trust the pipeline at 4am.

##### `watch` message (pre-escalation cue)

```
[WATCH] {short signal}
{source(s), 1 line}
{timestamp}
```

Example:
```
[WATCH] Multiple airlines rerouting off IR airspace
Reuters + FlightAware observed 17:42
2026-04-19 17:44 IDT
```

No link. No analysis. If the operator wants context, the digest is where context lives.

##### `alert` message (confirmed event)

```
[ALERT] {what happened, one line}
WHERE: {location if known, or 'not yet disclosed'}
SOURCES: {Tier-1 source names}
TIME: {ISO timestamp}
```

Example:
```
[ALERT] Projectile fire toward northern Israel, intercepted
WHERE: Western Galilee
SOURCES: IDF Spokesperson, Reuters
TIME: 2026-04-19 21:07 IDT
```

##### `critical` message (Home Front Command–level ground truth)

```
[CRITICAL] HFC alert firing — {region}
ACTION: {shelter instruction if known}
TIME: {ISO}
```

Reserved. This band fires only on an authoritative ground-truth source.

##### Digest (Telegram + email)

A structured, skimmable document, not a wall of text:

- **Headline state** (one line): what mode the pipeline is in today, why.
- **Signals since last digest**: bullet list, timestamped, grouped by cue class.
- **Open questions / unresolved claims**: things the pipeline saw but could not confirm.
- **Sources consulted this cycle**: Tier-1/2/3 counts, any sources that failed to fetch.
- **System status**: cadence currently in force, time since last mode change.

#### 4. Thresholds — silence is the default

The pipeline's hardest job is **staying quiet**. The threshold rules:

1. **Default state is silent.** The pipeline emits nothing unless a threshold is crossed.
2. **Watch threshold**: at least two Cue Class B signals (see source-selection exchange) co-occur within a rolling window, OR one Cue Class C signal from a Tier-1 source. One `watch` message emitted; cadence tightens internally.
3. **Alert threshold**: a Tier-1 kinetic or authoritative source confirms a specific event, OR Watch + wire-service confirmation within 10 minutes. One `alert` message emitted, possibly updated once if the event materially changes.
4. **Critical threshold**: Home Front Command fires, or a confirmed major regional kinetic event affecting the operator's area. One `critical` message emitted, ack required.
5. **Digest threshold**: fixed-schedule (e.g. 07:00 IDT) plus optional end-of-day wrap if the day had alerts.

Thresholds are **conjunctive by default** — multiple independent signals before firing — to suppress the single-source false positive that is the dominant failure mode of amateur OSINT pipelines.

#### 5. De-duplication and update semantics

A single real-world event produces many separate detections across sources. Naïve pipelines emit N pushes for one event. The rules:

- **Event identity**: each detection is hashed into a candidate event (location + event-type + time-bucket). New detections that match an existing event enrich it rather than fire a new push.
- **Update cap**: per event, at most one `alert` push and at most one follow-up `update` push (for material changes: casualty count moves from 0 to non-zero, scope changes from local to regional, official framing released).
- **Cool-down**: after an `alert`, the same event cannot re-fire for N minutes unless it escalates band (`alert` → `critical`).
- **Silencing stale events**: events older than 6 hours stop accruing follow-ups; they move to the digest.
- **Dedup is never cross-event.** Two separate events firing close in time both get their own alerts.

#### 6. Failure modes and degradation

A delivery pipeline that fails silently is worse than one that never existed. Explicit failure handling:

| Failure | What the system does | What the operator sees |
|---|---|---|
| A Tier-1 source API fails | Log, continue with remaining sources, note in next digest | Digest entry: "IDF Spokesperson feed not reachable 14:00–14:20" |
| Pushover API fails | Retry with backoff; on sustained failure, fail over to Telegram for alerts | Telegram message: "Pushover delivery failed — alerts routing via Telegram" |
| Telegram also fails | Fail over to SMS for `alert` and `critical` only | SMS: "[SYSTEM] Alert channels degraded; SMS fallback active" |
| Synthesizer (LLM) unreachable or rate-limited | Fall back to raw-match rules (keyword + source-tier logic) with a "reduced confidence" prefix on outputs | Next digest flags the degradation window |
| Network partition on the host | Local queue retains outbound messages; replays on reconnection with original timestamps and a `[DELAYED]` prefix | Operator sees catch-up burst flagged as delayed |
| Host down entirely | External uptime pinger notices and pages via SMS after N minutes | SMS: "[SYSTEM] OSINT pipeline host unreachable Xm" |

**Dead-man's-switch**: if the pipeline has been silent for longer than the digest cadence, the external pinger checks liveness. Silence without liveness → SMS notification. Silence with liveness → no action (this is the good case).

#### 7. Anti-fatigue discipline

Alert fatigue is the dominant long-term failure mode. Hard rules:

- **Monthly review**: the operator reviews every `alert` and `critical` fired in the prior month and retroactively labels it (useful / borderline / false positive). The ratio drives threshold tuning.
- **Target false-positive rate**: < 10% on `alert`, < 2% on `critical`. If breached, thresholds tighten before anything else.
- **No "growth" metric.** The system never optimises for more pushes. Fewer pushes, higher precision is always the right direction.
- **Escape hatch**: a one-command "mute for N hours" that the operator can invoke — with the mute fact itself logged to the digest for later review.

#### 8. What the operator experiences on a typical day vs. a bad day

**Typical quiet day:**

- 07:00 — one Telegram digest: "Region baseline. No significant kinetic activity in prior 24h. 3 watch signals logged; none confirmed."
- No Pushover notifications.
- 22:00 — optional end-of-day wrap if anything happened.

**Active day:**

- 07:00 — digest notes elevated cadence (pipeline in heightened mode — see implementation exchange).
- 14:22 — Pushover `watch`: "Multiple airlines rerouting off IR airspace."
- 14:38 — Pushover `alert`: "CENTCOM confirms US air assets repositioning to EastMed."
- 21:07 — Pushover `critical`: "HFC alert firing — Western Galilee. Shelter."
- 21:40 — Pushover `alert` update: event scope/confirmation.
- 23:00 — Telegram wrap digest.

Total Pushover pushes on a materially-bad day: under ten. Any design that would produce more has the wrong thresholds.

### Sources

Methodological synthesis. References:

- Prior exchange: `outputs/individual/2026-04-19-situational-awareness-scope.md`.
- Prior exchange (cue taxonomy): `outputs/individual/2026-04-19-source-selection-monitoring.md`.
- Pushover API documentation (priority levels, sounds, emergency ack) — [pushover.net/api](https://pushover.net/api).
- General alert-fatigue literature from incident-management / SRE practice (Google SRE Book, ch. 6 — "Monitoring Distributed Systems"; Rob Ewaschuk's "My Philosophy on Alerting").
