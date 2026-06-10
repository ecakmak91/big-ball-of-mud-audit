# Big Ball of Mud Audit

A [Claude](https://claude.ai) skill that reviews a codebase like a senior software architect and code-quality reviewer, detects **"Big Ball of Mud"** symptoms, and produces a scored health report (0–10) with a realistic, non-destructive refactor plan.

A *Big Ball of Mud* is a system that grew without a plan: modules bleed into each other, business logic lives wherever it was convenient, temporary hacks calcify into load-bearing code, and any change risks breaking something far away. This skill measures how close a codebase is to that state — and charts the cheapest credible path away from it.

## What it does

Instead of generic best-practice advice, the skill **reads the actual code** and backs every finding with real file, folder, and function references. It:

- Maps the project (manifest, structure, biggest files, cross-module coupling, where DB/business logic lives)
- Inspects folder structure, routing, components, business-logic placement, services/APIs, state management, ORM usage, duplication, helper sprawl, role-based access, panel separation, UI duplication, calcified temporary code, god files, and blast radius
- Answers 10 architectural diagnostic questions
- Emits a scored report with a prioritized, staged action plan

## Scoring

| Score | Meaning |
|------|---------|
| 0–3  | Critical Big Ball of Mud risk |
| 4–6  | Moderate technical debt |
| 7–8  | Manageable architecture |
| 9–10 | Healthy, sustainable architecture |

## Report structure

- **Overall Assessment**
- **Big Ball of Mud Symptoms** — concrete, file-level evidence
- **Highest-Risk Areas**
- **Sources of Technical Debt**
- **Architectural Boundary Problems**
- **Quick Wins** (1–2 days)
- **Mid-Term Refactor Plan** (1–2 weeks)
- **Is a Full Rewrite Needed?** — a clear decision
- **Prioritized Action List** (Critical / High / Medium / Low)

Reports are written in English by default. To change this, edit the one line in the `SKILL.md` frontmatter/body that specifies report language.

## Installation

This skill is distributed as a `.skill` file.

1. Download `big-ball-of-mud-audit.skill` from this repo.
2. In Claude, go to **Settings → Capabilities → Skills** (or the Skills section of your client) and upload the `.skill` file.

To edit the source, the `.skill` file is just a zip:

```bash
cp big-ball-of-mud-audit.skill big-ball-of-mud-audit.zip
unzip big-ball-of-mud-audit.zip   # -> big-ball-of-mud-audit/SKILL.md
```

## Usage

Once installed, the skill triggers on architecture-review style requests. Examples:

- "Review my project's architecture and tell me if it's a Big Ball of Mud."
- "Do a code-quality and tech-debt assessment of this repo."
- "Everything feels connected to everything and I'm scared to change anything — what's wrong?"
- "Should I rewrite this project or refactor it incrementally?"

It works best when Claude can see the codebase (e.g. in Claude Code, or with files/repo provided). When only part of the codebase is visible, it scopes the score to what it can see and says so.

## Design principles

- **Evidence over advice** — every claim points at a real file or symbol.
- **Cheap before expensive** — small incremental refactors are evaluated before any rewrite is suggested.
- **Don't break the running system** — recommendations favor strangler-fig / extract-and-delegate migrations that keep the app working throughout.
- **Honest scoring** — the score reflects the code as it is, not the stack's reputation.

## License

MIT — do what you like.
