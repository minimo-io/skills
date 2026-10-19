---
name: bug-solution-planner
description: Proposal-only bug solver. Reads the known-bug list in .bugs/BUGS.md, picks the oldest open bug with no solution referencing it, re-verifies the bug still reproduces in current code, evaluates multiple approaches, writes a thorough professional solution to .bugs/solutions/SOLUTION-NNN.md, indexes it in .bugs/SOLUTIONS.md, and emails a branded Spanish proposal to cubiqlabs@gmail.com. It NEVER implements; implementation is the separate bug-implementer skill. Use whenever asked to propose a solution / plan a fix for an open bug.
---

# Bug Solution Planner

## ⚠️ CRITICAL — PROPOSAL-ONLY (highest rule; overrides everything below)

0. This skill analyzes and proposes — it **NEVER implements fixes**. No application-code edits, no scripts touching app files, no DDL, no config tweaks, no commits — not even for a "trivial" or "obvious" fix, and never "indirectly".
1. Write surface, exactly: `.bugs/SOLUTIONS.md` (index), one `.bugs/solutions/SOLUTION-NNN.md` (the proposal), the report email, and — only during verification — a tick plus dated note in `.bugs/BUGS.md` for a bug proven already fixed (rule 10).
2. Implementation belongs to the separate `bug-implementer` skill, which the user runs manually only after approving a proposal, and which must present its final plan to the operator for explicit approval before editing.
3. Every proposal is written as `PENDING APPROVAL — no code changed`.

## Where bugs and solutions live (separate domains — never mix them)

4. `.bugs/BUGS.md` — the bug list, owned by `bug-hunter`. Read it; never reword entries. The only allowed edit is ticking a bug proven already fixed during verification (rule 10).
5. `.bugs/SOLUTIONS.md` — the solution index only, newest proposal on top. The full plan always lives in the per-solution file.
6. `.bugs/solutions/SOLUTION-NNN.md` — one file per **solution**, created on the first proposal. The file name is the solution ID and must never equal a bug file/ID. Each solution file references the `BUG-NNN` it attempts to solve. **Never delete or rename a solution file**: it is the permanent audit trail (same principle as `.tasks/`).
7. Status values: `PENDING APPROVAL`, `APPROVED`, `IMPLEMENTED`. `APPROVED` and `IMPLEMENTED` are managed by `bug-implementer`. There is no rejection/blocking flow and no revision history: a bug with an existing solution is not proposed again.

## Selection

8. Read `.bugs/BUGS.md`. Eligible = an open `- [ ]` bug with **no solution referencing it** in `.bugs/SOLUTIONS.md` (search the index for `BUG-NNN`; never infer from a solution filename).
9. Pick the eligible bug with the **LOWEST `BUG-NNN`** (the oldest). The bug list is newest-first, so sort by bug number — never take the last line blindly. If none is eligible: report "No eligible bugs — no solution proposed.", send the Spanish no-eligible email (rule 18), and stop. No solution file, no index change.

## Run flow

10. **Verify the bug still stands (mandatory, before any analysis).** Re-read every `Found:` / `Evidence:` file:line and re-trace the failure path in the current code. If the code no longer exhibits the bug: tick `- [x]` in `.bugs/BUGS.md` in place and append ONE dated note with the file:line proof (never reword the entry), then return to rule 8 and continue with the next eligible bug — at most ONE proposal per run. If the bug still reproduces: continue.
11. **Analyze, evidence-based.** Read the cited files, their call sites, and copy-pasted sibling components (the repo's recurring drift source — `grep` for the same pattern). Check types, existing paraglide keys in `messages/*.json` (search by translation value before proposing a new key), store mutation ordering, SPA/Tauri constraints, and the rules in `AGENTS.md`. Every claim must cite code actually read this run — no speculation. State confidence and unknowns.
12. **Evaluate at least TWO candidate approaches.** For each: sketch, pros/cons, blast radius, regression risk, idiomatic fit. Choose ONE with explicit rationale — the best professional solution, not the first idea.
13. **Allocate the solution ID and write the proposal.** Scan `.bugs/SOLUTIONS.md` and `.bugs/solutions/` for the highest `SOLUTION-NNN`; the next number is one greater (zero-padded to 3 digits; never reuse). Write `.bugs/solutions/SOLUTION-NNN.md` (format below) and add its line at the TOP of the `## Solutions` list in `.bugs/SOLUTIONS.md`: ``- **SOLUTION-NNN** — <title> — `PENDING APPROVAL` — <YYYY-MM-DD> — fixes **BUG-NNN [SEVERITY]** — [solution](./solutions/SOLUTION-NNN.md)``.
14. **Email the proposal** (rules 18–19), then report and stop.

## Proposal file format (must be thorough and specific)

15. Header: `# SOLUTION-NNN — <title>`, then:
    - `- **Solution ID:** SOLUTION-NNN`
    - `- **Bug:** [BUG-NNN](../BUGS.md) — [SEVERITY]`
    - `- **Analyzed commit:** <short> (<full sha>)`
    - `- **Date:** <ISO UTC>`
    - ``- **Status:** `PENDING APPROVAL` — no code changed; do not implement without an explicit order.``
16. Sections, in this order:
    - `## Verification — the bug still reproduces` — exact current code + step-by-step failure path, with file:line.
    - `## Root cause` — why it happens.
    - `## Options considered` — ≥2 approaches, each with sketch, pros/cons and blast radius; then `### Decision` with the chosen approach and rationale.
    - `## Proposed changes` — per file: `path:line`, exact before/after snippets, why; ordered as the implementation sequence.
    - `## Edge cases`
    - `## Risks and mitigations`
    - `## Do not touch` — files/behaviors that must remain unchanged.
    - `## Verification plan` — exact commands and expected results (`npm run check`, playwright L0–L3 per AGENTS.md for UI, curl/HTTP, BR/UY/manual scenarios).
    - `## Acceptance criteria` — checkable.
    - `## Rollback`
    - `## Open questions` — or "None."
17. The body is always the current plan. There is no revision history: a bug with an existing solution is not re-proposed.

## Email the proposal (same pipeline as bug-hunter)

18. Send ONE email to `cubiqlabs@gmail.com` right after the files are updated:
    - **Transport:** `POST $PUBLIC_API_BASE_URL/notifications/email` with header `Authorization: Bearer $CUBIQ_API_KEY` (both from this repo's `.env`; never print or commit the key).
    - **Payload:** `to`, `subject`, `html`, `text`, `telegramMessage`.
    - **Branding:** mirror the contact form template (dark `#111` card, orange `#F79319` gradient header, Malthaus logo `${app.productionBaseUrl}/favicon-96x96.png`, white table rows).
    - **Language:** `html`/`text` in Spanish; `telegramMessage` in English.
    - **Subject + inner H2:** `Propuesta de solución para el bug BUG-NNN` (identical in both places; substitute the real bug ID). Keep it short — the solution ID goes in the body, not the subject.
    - **Content — ID order is mandatory:** the bug ID line first and the solution ID **directly below it** — `Bug: BUG-NNN — <title>` then `Solución: SOLUTION-NNN` — followed by severity, `PENDING APPROVAL`, analyzed commit, path to the solution file, root cause, proposed changes (files), chosen approach + why, risks, verification plan, acceptance criteria; state explicitly that no code changed and implementation waits for manual approval. If verification ticked any already-fixed bug this run, mention it in one line.
    - **No-eligible email:** when rule 9 finds no eligible bug, send the same branded email with subject + inner H2 `Sin bugs elegibles para propuesta de solución` (identical in both places). Body in Spanish: state that no bug needs a solution right now, distinguish "no open bugs" from "all open bugs already have a solution", include counts (open `- [ ]` bugs and open bugs already covered by a solution), and note that no solution file was created and the index was not modified. `telegramMessage` in English. Same failure handling as above.
    - **Failure:** keep the files, log the error, retry once, then report the failure; never block the proposal on the email.
19. Never claim the bug is fixed or the proposal approved — the email is a proposal notification.

## After the run

20. Report to the user: selected bug, solution ID, any bug ticked as already fixed, proposal file + index updated, email sent/failed; when no bug was eligible, report that plus the no-eligible email. State explicitly that nothing was implemented and that implementation awaits manual approval plus an explicit `bug-implementer` run. Then stop.
21. House style: terse (1–2 sentences per item).
