---
name: big-ball-of-mud-audit
description: Audit a codebase like a senior software architect and code-quality reviewer to detect "Big Ball of Mud" symptoms — unplanned growth, tight coupling between modules, temporary code that became permanent, eroded architectural boundaries, and critical technical debt. Use this skill whenever the user asks for an architecture review, code-quality review, tech-debt assessment, maintainability check, refactor plan, or "should I rewrite this?" decision — and also whenever they describe symptoms like "everything is connected to everything," "I'm scared to change anything," "the codebase is a mess," "spaghetti code," or "adding a feature touches a hundred files," even if they don't use the words "architecture" or "Big Ball of Mud." Produces a scored health report (0–10) with concrete file-level evidence and a prioritized, non-destructive action plan.
---

# Big Ball of Mud Audit

Review a codebase the way a seasoned architect would: not by listing generic best practices, but by *reading the actual code* and pointing at real files, folders, and functions. The output is a scored health report plus a realistic, staged plan to improve the architecture without breaking the running system.

A "Big Ball of Mud" is a system that grew without a plan: modules bleed into each other, business logic lives wherever it was convenient, temporary hacks calcified into load-bearing code, and any change risks breaking something far away. Your job is to measure how close this codebase is to that state and chart the cheapest credible path away from it.

## Core principle

Every claim must be backed by evidence from *this* codebase. Generic advice ("you should separate concerns") is worthless here. Instead: "`app/dashboard/page.tsx` (640 lines) fetches from the DB, formats currency, and renders the chart — it does three jobs that belong in three places." Always name the file, folder, or symbol. If you can't point at it, don't claim it.

The report is written in **English** regardless of the conversation language, unless the user explicitly asks otherwise — architecture reports are often shared with mixed or international teams.

## Step 1 — Map the codebase before judging it

Don't start writing the report until you understand the shape of the project. Build a mental model first.

1. **Read the manifest and config.** `package.json` (deps, scripts), framework configs (`next.config.js`, `tsconfig.json`, `vite.config`, etc.), and any monorepo config. This tells you the stack, the intended structure, and what tooling exists.
2. **Walk the directory tree** (e.g. `find . -type f -not -path '*/node_modules/*' -not -path '*/.next/*' | head -200`, or `view` the root). Note the top-level organization: is it organized by feature, by layer, by role, or is it a flat dump?
3. **Find the big files.** Size is the single loudest smell. Run something like:
   `find . -name '*.ts' -o -name '*.tsx' -o -name '*.js' -o -name '*.jsx' | grep -v node_modules | xargs wc -l 2>/dev/null | sort -rn | head -30`
   Files over ~300 lines are suspects; over ~500 are almost always doing too much.
4. **Map imports to detect coupling.** Grep for cross-feature / cross-layer imports — e.g. a component importing directly from another feature's internals, UI files importing the DB client, `../../../` import chains. `grep -rn "from '\.\./\.\./\.\." --include='*.ts*' src/` and similar reveal reach-across.
5. **Locate where business logic lives.** Search for DB/ORM calls (`db.`, `prisma.`, `drizzle`, `supabase`, raw SQL), `fetch`/API calls, and auth/role checks. *Where* these appear matters: in components and pages = smell; in a service/domain layer = healthy.

Adapt the exact commands to the stack you find. The goal is a map, not a fixed checklist.

## Step 2 — Inspect against the scope

Cover these areas, citing real files for each finding:

- **Folder structure** — feature-based, layered, role-based, or chaotic?
- **Routing** — clean and predictable, or logic-heavy and duplicated?
- **Components** — pure UI, or carrying business logic, data fetching, and side effects?
- **Business logic location** — centralized in a service/domain layer, or scattered across pages/components/route handlers?
- **API / server actions / services** — consistent pattern, or three different ways to do the same thing?
- **State management** — coherent, or a mix of prop drilling, global stores, context, and ad-hoc fetching?
- **Database / ORM usage** — accessed through a layer, or called directly from the UI?
- **Duplicated logic** — same rules/validation/formatting re-implemented in multiple places.
- **Utility / helper sprawl** — a `utils.ts` junk drawer, or focused modules?
- **Role-based access** — enforced consistently in one place, or re-checked (and re-copied) everywhere?
- **Panel separation (admin / tenant / building-owner / etc.)** — clean boundaries, or copy-pasted variants of the same screens?
- **UI component duplication** — repeated buttons/tables/forms that should be shared.
- **Temporary-turned-permanent code** — `TODO`, `FIXME`, `HACK`, `temporary`, `for now`, commented-out blocks, hardcoded values.
- **God files / over-responsibility components** — single files doing many unrelated jobs.
- **Blast radius** — how easily one change breaks unrelated parts.

Grepping for `TODO|FIXME|HACK|XXX|temporary|for now|hack` across the source is a fast way to surface calcified hacks — quote the real lines you find.

## Step 3 — Answer the ten diagnostic questions

Work through these explicitly; they drive the score and the report:

1. Is there a clear architectural boundary in the code?
2. Are features isolated, or is everything wired to everything?
3. Do components contain only UI, or business logic too?
4. Is the same logic repeated in different places?
5. Is there needless copying or confusion across role-based panels?
6. Is adding a new feature easy, or does it require touching many files?
7. Do bugs stay local, or propagate across the system?
8. Is there code that looks like "let's just do it this way for now"?
9. Are there areas that clearly need refactoring but were deferred?
10. Will this structure stay sustainable as the project grows?

## Step 4 — Score and report

ALWAYS start with the architecture health score, then use this EXACT section structure.

### Scoring rubric
- **0–3: Critical Big Ball of Mud risk** — boundaries absent, change is dangerous, growth is unsustainable.
- **4–6: Moderate technical debt** — works, but coupling and duplication are accumulating; intervention needed soon.
- **7–8: Manageable architecture** — mostly sound with isolated, fixable problem spots.
- **9–10: Healthy, sustainable architecture** — clear boundaries, isolated features, low blast radius.

Anchor the number to evidence. A project with 5 god files, DB calls in components, and duplicated role logic is not a 7 no matter how modern the stack looks. Don't grade on intentions or tooling; grade on the code as it is.

### Report template

```
# Architecture Health Score: X/10 — [Rubric label]

## Overall Assessment
A short, direct verdict on the architectural state of the project.

## Big Ball of Mud Symptoms
Concrete symptoms, each with a file / component / folder name. No symptom without evidence.

## Highest-Risk Areas
The fragile, hard-to-maintain, over-responsible parts. Explain *why* each is risky (blast radius, coupling, size).

## Sources of Technical Debt
Duplicated code, tangled state, misplaced business logic, temporary solutions — explained with examples.

## Architectural Boundary Problems
Where feature / role / page / component / service / database layers have bled into each other.

## Quick Wins
Small, high-impact fixes doable in 1–2 days. Concrete and safe.

## Mid-Term Refactor Plan
A realistic 1–2 week plan. Staged, ordered, each step shippable without breaking the system.

## Is a Full Rewrite Needed?
A clear decision: rewrite from scratch, or improve incrementally? Justify it. Default strongly toward incremental unless the evidence genuinely forbids it.

## Prioritized Action List
- **Critical** — ...
- **High** — ...
- **Medium** — ...
- **Low** — ...
```

## Rules

- **Evidence over advice.** Reference real files and code, not generic principles. Every section should be traceable to something you actually read.
- **Cheap before expensive.** Evaluate small, incremental refactors before proposing any rewrite. A rewrite is a last resort, recommended only when incremental improvement genuinely can't reach a sustainable state — and say so explicitly when that's the call.
- **Actionable and staged.** Solutions must be concrete, step-by-step, and realistic for the team to execute. "Extract a service layer" is not a step; "move the DB queries from `app/dashboard/page.tsx` into `lib/services/dashboard.ts`, have the page call that, repeat for the 3 other pages doing the same" is.
- **Don't break the running system.** For every recommendation, favor approaches that keep the app working throughout: strangler-fig migration, extract-and-delegate, parallel implementations behind a flag. Call out where a change is risky and how to de-risk it (tests first, one module at a time).
- **Be honest about the score.** Don't soften a 3 into a 6 to be kind, and don't catastrophize a 7 into a 4. The user needs an accurate read to make decisions.
- **Calibrate to project size.** A 20-file side project and a 2,000-file SaaS have different bars. A little pragmatic mess in a tiny app is fine; the same patterns at scale are a crisis. Judge sustainability relative to where the project is heading.

## Notes on partial codebases

If you can only see part of the codebase (a few uploaded files, a single directory), say so plainly, scope the score to what you can see, and note which areas you couldn't assess. Don't extrapolate a whole-system verdict from a fragment — but do give a thorough read of what's in front of you.
