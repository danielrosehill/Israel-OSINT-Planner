---
thread: 2026-04-19-situational-awareness-scope
topic: Exploration ground for a low-noise, fact-based Israel situational awareness pipeline
started: 2026-04-19
exchange_count: 1
---

> **Note**: This output was produced through AI-assisted research using Claude Code. It is a planning artifact, not an operational tool.

# Situational Awareness Pipeline — Exploration Ground

## Exchange 1

**Prompt**: prompts/run/initial/2026-04-19-situational-awareness-scope.md
**Summary**: Document the problem space, parameters, existing tools, and signal taxonomy for a low-noise situational awareness pipeline focused on Israel's active and latent conflict fronts.

### Key Findings

#### 1. The core problem is noise, not information scarcity

The operator already has access to more information than is useful: Telegram groups, Twitter/X, OSINT accounts, Israeli and international media, Liveuamap, official statements. The bottleneck is **filtering**, not collection. The desired system removes information, not adds it.

A second, separate problem is the **physical time budget** during active strikes: in an apartment without a nearby public shelter, the gap between a red alert and needing to be inside shelter is tight. Any pipeline that reliably surfaces a "something big is developing" signal a few minutes before the siren produces disproportionate real-world value (time to dress, prepare a go-bag, move toward shelter).

#### 2. Two distinct modes the pipeline must serve

| Mode | Trigger | Cadence | Output |
|---|---|---|---|
| **Quiet-period monitoring** | No active kinetic activity, ceasefire/grey-zone | Low (e.g., every 10–30 min sweep, most returning "nothing significant") | Periodic fact-based situational report; silent most of the time |
| **Escalation watch** | Pre-escalation signals crossing threshold, or active conflict | Higher; push notifications | Pushover alert with terse, factual summary — no speculation |

Both modes share the same ingestion and synthesis pipeline, but differ in notification threshold and cadence.

#### 3. Parameters (hard constraints)

- **Not real-time**. ~10-minute sweep cadence is acceptable and probably right — most sweeps will correctly yield nothing.
- **Cost is not the constraint**. Quality of filtering is.
- **Notification surface**: Pushover is primary. Telegram bots (keyword-triggered) are a reasonable secondary channel.
- **Tone**: fact-forward. No speculation, no "analysts say", no mood music. If a claim is unconfirmed, it is labelled as a claim, not as an event.
- **Operator goal**: disconnect from the speculative news cycle. The pipeline's success metric is *fewer hours spent doomscrolling*, not *more data surfaced*.

#### 4. Existing tools inventory

**What works (keep):**

- **Perplexity Sonar API** for grounded retrieval — factual, cited, current.
- **OpenRouter** (Sonnet-class model) for synthesis and report writing — cost-efficient, coherent output.
- **Liveuamap** — the closest existing product to what's wanted. Strong for kinetic events on a map; weaker for pre-escalation signals and official statements.
- **Red-alert apps** — continue to own the siren-level immediate-threat layer. The new system does not try to replicate these.

**What doesn't work (replace or reduce):**

- Raw Telegram group monitoring — high volume, high noise, emotionally costly.
- Twitter/X scrolling — same, worse on the speculation axis.
- Most enterprise OSINT platforms — functionally close, but priced for enterprise buyers.

**What's missing:**

- A single surface that combines (a) confirmed events, (b) official statements, and (c) posture / diplomatic-cadence shifts — de-duplicated, time-ordered, and delivered on a cadence the operator sets.

#### 5. Signal taxonomy

In rough descending order of reliability / priority:

1. **Official statements from named leaders / governments** — US (especially the President), Israeli cabinet and IDF spokesperson, Iranian leadership (Supreme Leader, IRGC, Foreign Ministry), regional heads of state, spokespeople for Hezbollah/Houthis/Hamas when speaking on-record.
2. **Confirmed material events** — strikes, interceptions, airspace closures, port closures, mobilisations, evacuations, infrastructure hits. Requires multi-source confirmation or an official claim.
3. **Force-posture indicators** — visible in open sources: ship movements (AIS), aircraft movements (ADS-B), base activity, call-ups.
4. **Diplomatic-cadence shifts** — cancelled meetings, recalled envoys, unusual leader travel, emergency UN sessions, sudden summit calls.
5. **Analyst commentary / think-tank output** — treated as context, not as trigger.
6. **Anonymous social-media reporting** — treated as signal-of-claim only; never alone sufficient to trigger a notification.

The pipeline's filter rules should weight these tiers explicitly. A Tier 1 statement above a threshold of materiality is a push-worthy event; a Tier 6 rumour is not, regardless of volume.

#### 6. Fronts / scope

Current focal theatre: **Iran**. But the pipeline must natively handle the concurrent fronts:

- Iran (direct exchanges, proxy activity, nuclear-file signals)
- Lebanon (Hezbollah posture, cross-border fire)
- Gaza (ceasefire compliance, hostage-deal status, tactical activity)
- Yemen / Houthis (Red Sea shipping, long-range launches)
- Syria (Iranian logistics, Israeli strikes)
- West Bank (settler violence, IDF operations, PA stability)
- Diplomatic / US posture (administration statements, force deployments to region)

The operator's request is **front-agnostic architecture**: today the focus is Iran; the same plumbing should serve whichever front flares next.

#### 7. Pre-escalation signal hypothesis

The operator's lived observation is that each recent round of escalation was preceded by a **cluster of specific, observable signals** — not a single smoking gun, but a shift in the mix. This is the central design bet: an LLM-assisted sweep that reads a fixed panel of sources every N minutes and scores the *change in signal composition* (not just the presence of any one item) may produce useful pre-escalation cues that no single-feed monitor can.

This hypothesis needs structured testing — retrospective analysis of the 24–72 hours preceding the last several escalation events would help validate which signal categories actually carry predictive weight.

### Open design questions (for subsequent prompts)

1. **Source map**: concrete list of primary sources per tier, per front. Which Telegram channels, which official spokespeople, which wire services, which OSINT accounts are worth ingesting, and in what language.
2. **Report schema**: fixed structure for the periodic situational report — so the operator can skim it in 30 seconds and a machine can diff successive reports.
3. **Trigger rules**: what threshold promotes a sweep result from "silent" to "Pushover push"? Probably a combination of tier, multi-source confirmation, and change-from-previous-sweep.
4. **Pre-escalation scoring**: what does the signal-composition scoring actually look like? Retrospective validation against past escalations is the obvious first step.
5. **Pipeline architecture**: Perplexity Sonar for retrieval, OpenRouter for synthesis is the baseline. Open question: where does state live (what was already reported, what's new), and how are duplicates suppressed across sweeps?
6. **Language coverage**: Hebrew and Arabic primary-source material is high-value and under-represented in English-only pipelines. How is this handled — machine translation in the ingest step, or language-native sub-pipelines?
7. **Failure mode**: what does the system do when it's *uncertain*? The default bias should be silence over false alarm, since every false alarm erodes the operator's trust and rebuilds the doomscroll habit.

### Sources

This exchange is a scoping / framing document drawn from the operator's stated requirements. No external sources were consulted; no factual claims about specific events are made. Subsequent exchanges will cite sources as they introduce them.
