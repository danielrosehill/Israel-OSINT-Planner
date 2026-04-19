---
date: 2026-04-19
slug: situational-awareness-scope
type: initial
source: chat (voice-typed), raw in prompts/drafting/2026-04-19-situational-awareness-raw.md
---

# Situational awareness pipeline — scope & exploration ground

## Context

Israel has well-developed red-alert apps for incoming missile warnings, but where there is no nearby public shelter, the real-world time budget between alert and needing to be inside shelter is only a few minutes — which at night, from an apartment, is very tight.

Separate from the siren-level alert, each recent round of escalation has been preceded by what feel like **specific pre-escalation signals**: observable shifts in official statements, force posture, diplomatic cadence, etc. These feel tractable to an AI-enhanced news-filtering / notification pipeline.

We are currently in a ceasefire that does not look likely to hold. Iran is the current focal theatre, but the problem generalises across Israel's concurrent fronts (Houthis, Lebanon, Gaza, Syria, West Bank).

## Objective

Build a **low-noise, fact-based situational awareness layer** that:

- Lets the operator disconnect from the 24/7 speculative news cycle.
- Surfaces only meaningful signals — especially pre-escalation cues and confirmed material events.
- In active conflict, provides the few-minutes-early "something is happening" cue that is genuinely invaluable (time to get dressed, ready a go-bag, move toward shelter).
- Produces periodic **situational reports** in a fixed, fact-forward format rather than a continuous feed.

Explicit non-goals: replicating the red-alert app, real-time per-second tracking, speculation/punditry.

## Constraints & preferences

- **Cadence**: not real-time. Something like every ~10 minutes is fine; most sweeps will correctly return "nothing significant."
- **Cost**: not the bottleneck.
- **Notification surface**: Pushover is the preferred delivery channel. Telegram bots with keyword triggers are a secondary option.
- **Existing workflow that has worked**: Perplexity Sonar API for grounded retrieval → OpenRouter (Sonnet-class model) for synthesis and report writing. Cost-efficient; worth preserving as the baseline pipeline.
- **Closest existing tool that is actually useful**: Liveuamap. Everything else tends to be enterprise-priced or too noisy.
- **Current manual surface being replaced**: Telegram groups, Twitter/X, ad-hoc OSINT feeds — valuable occasionally, mostly noise and speculation.

## Signal priorities

The highest-value signals, in rough order:

1. Official statements from governments and named leaders (e.g., US administration / Trump, Israeli cabinet, Iranian leadership, regional heads of state).
2. Confirmed material events (strikes, mobilisations, airspace closures, evacuations, infrastructure hits).
3. Force-posture indicators visible in open sources.
4. Diplomatic-cadence shifts (cancelled meetings, recalled envoys, unusual travel).

Lower priority: analyst commentary, social-media rumour, anonymous-sourced reporting — treated as signal-of-claim, not evidence.

## Scope for this first output

Document the **exploration ground**:

- Problem framing as above.
- Parameters the system must respect (cadence, cost, notification surface, tone).
- Current tools inventory (what works, what doesn't, what's missing).
- Signal taxonomy (what counts as a meaningful cue vs. noise).
- Open design questions the next iteration should answer.

This is a planning artifact, not an implementation. Subsequent prompts will drill into specific pieces (source map, report schema, trigger rules, pipeline architecture).
