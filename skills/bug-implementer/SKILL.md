---
name: bug-implementer
description: Implements ONE already-written solution from .bugs/solutions/SOLUTION-NNN.md when the human explicitly runs it. Presents the final plan to the operator and waits for explicit approval before editing. Guards against a dirty tree and a stale analyzed commit, applies exactly the approved changes, runs the proposal's verification plan, then ticks the bug in .bugs/BUGS.md and marks the solution and .bugs/SOLUTIONS.md index as IMPLEMENTED. Never invents fixes, never edits unrelated code, never sends email. Use when the user explicitly orders implementation of a proposed bug solution.
---

# Bug Implementer

## ⚠️ CRITICAL — OPERATOR APPROVAL + IMPLEMENT FROM THE PROPOSAL ONLY (highest rule; overrides everything below)

0. This skill implements ONE existing solution from `.bugs/solutions/SOLUTION-NNN.md`. It never invents a fix, never edits unrelated code, never refactors "while here", and never adds opportunistic changes.
1. It runs ONLY when the user explicitly invokes it (human in the loop). The invocation may name `SOLUTION-NNN` or `BUG-NNN`; resolve the pair through `.bugs/SOLUTIONS.md`. If no ID is given, list available proposals (`PENDING APPROVAL` / `APPROVED`) and ask — never auto-pick.
2. **The manual invocation is NOT approval.** Before touching any file, present the final implementation plan to the operator (solution ID, bug ID, files/lines, exact changes, risks, verification) and ask explicitly for approval. Do NOT edit anything until the operator replies with an explicit approval. If the operator rejects or requests changes, stop; leave the solution `PENDING APPROVAL` and report.
3. Preconditions: the solution file exists, the matching bug is still open in `.bugs/BUGS.md`, and the solution is not already `IMPLEMENTED`. Otherwise report and stop.
4. No email is sent by this skill.

## Pre-flight

5. Read `.bugs/solutions/SOLUTION-NNN.md` completely and the matching entry in `.bugs/BUGS.md`.
6. **Dirty-tree guard:** run `git status --short`. If any file named in the proposal has uncommitted changes, stop and ask the user before touching it.
7. **Staleness guard:** compare current `HEAD` with the proposal's `Analyzed commit`. If they differ, run `git diff --name-only <analyzed-commit>..HEAD -- <files named in the proposal>`. If any proposed file changed, report "proposal stale — re-run bug-solution-planner", leave the status unchanged, and stop. Never improvise.
8. **Operator approval gate:** present the final plan and wait for explicit approval (rule 2). Only after approval set the solution status to `APPROVED` with a dated note (`Implementation approved by operator — <date>`) and proceed to the first code edit.

## Implementation

9. Apply EXACTLY the `## Proposed changes` plan, in its stated order. No extra edits, renames, formatting sweeps, or dependency changes.
10. If the proposal is insufficient or wrong at implementation time: STOP before improvising and ask the user. If unresolved, revert your own partial edits to the pre-run state, report, leave the status unchanged, and stop.
11. If the user explicitly approves a deviation, record it in `## Implementation record` and implement it — never make silent deviations.
12. Follow the proposal's `## Do not touch` list strictly.

## Verification

13. Run the proposal's `## Verification plan` verbatim: `npm run check` (compare against the known baseline — no new errors), playwright L0–L3 per AGENTS.md when the fix is UI-facing, curl/HTTP checks, and the stated BR/UY/manual scenarios.
14. Report actual evidence (commands + results). Never claim verification without running it.

## Marking (same session, only on success)

15. Tick the bug in `.bugs/BUGS.md` in place: `- [x]`, then append ONE dated note (e.g. `2026-10-19 — Fixed; see .bugs/solutions/SOLUTION-001.md.`). Never reword or delete the entry. Do not touch `## Audit state`.
16. Set the solution status to `IMPLEMENTED` and append/update `## Implementation record`: date, operator approval, files touched, deviations (or "none"), verification evidence (commands + results), commit sha if one was ordered.
17. Update the matching index line in `.bugs/SOLUTIONS.md` to `IMPLEMENTED`.
18. Update `docs/lessons-learned.md` with a short dated entry (what was fixed, key gotcha) per AGENTS.md.

## On failure

19. Leave the bug open, revert partial edits that do not compile/pass, leave the solution status unchanged, and report the exact blocker (file:line + error). Do not tick anything.
20. Never delete or rename anything under `.bugs/solutions/`.

## Guardrails and after the run

21. Never `git commit`, `git push`, or bump `package.json` / `src-tauri/tauri.conf.json` without an explicit order (AGENTS.md). If asked to commit, bump both files by one patch version first, then commit with the `AGENT:` prefix.
22. Report to the user: solution ID, bug ID, final status (`IMPLEMENTED` or unchanged), files changed, verification results, deviations, and whether a commit/bump still awaits an order. Then stop.
23. House style: terse (1–2 sentences per item).
