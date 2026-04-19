# Israel OSINT Planner

[![Claude Code Projects Index](https://img.shields.io/badge/Claude%20Code-Projects%20Index-blue?style=flat-square&logo=github)](https://github.com/danielrosehill/Claude-Code-Repos-Index)

A public Claude Code research workspace for **planning open-source intelligence (OSINT) investigations focused on Israel**.

This repo is for the *planning* step — defining the question, mapping sources, choosing methods, and laying out an analysis structure — not for live collection or monitoring. Outputs are investigation briefs, source maps, collection plans, and analytic frameworks.

## What's here

- `CONTEXT.md` — always-on background: topic, framing, key domain facts.
- `SCOPE.md` — what's in and out of bounds for this workspace.
- `CLAUDE.md` — agent instructions and the research workflow contract.
- `MEMORY.md` — persistent-memory policy.
- `WORKSPACE.md` — folder-by-folder map.
- `prompts/` — research prompts (queue → run → archive).
- `outputs/` — investigation plans, briefs, source maps, frameworks.
- `context/` — human-supplied and agent-retained background.
- `voice-notes/` — voice-captured prompts (transcribed via AssemblyAI).
- `private/` — gitignored personal/working notes.

## How to use it

1. Drop a research question into chat, place a prompt in `prompts/queue/`, or record a voice note.
2. Claude runs the prompt under the workflow defined in `CLAUDE.md` and writes an investigation plan to `outputs/`.
3. Iterate — prior outputs feed back as context for deeper or follow-on planning.

## Posture

Public, transparent, methodology-first. Open sources only. No live monitoring, no targeting of private individuals, no operational planning. See `SCOPE.md` for the full boundary.

## Template origin

Scaffolded from [Claude-Research-Space-Public-Template](https://github.com/danielrosehill/Claude-Research-Space-Public-Template).

## License

MIT. Outputs are AI-assisted and should be reviewed before being treated as authoritative.
