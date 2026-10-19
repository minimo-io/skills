---
name: bug-hunter
description: Repo bug audit loop. Reads the known-bug list in .bugs/BUGS.md and skips the run if the last audit was less than 15 minutes ago (no commit-based gate). Otherwise audits the code (starting from the stored Hit, prioritizing changes since the last audit), ticks bugs proven fixed, adds at most ONE new important bug (low-hanging fruit first) with a Hint, updates the audit state, and emails a branded Spanish report to cubiqlabs@gmail.com stating whether new bugs were found. Use whenever asked to run a bug hunt / bug audit.
---

# Bug Hunter

## ⚠️ CRITICAL — READ-ONLY AUDITOR (highest rule; overrides everything below)

0. This skill audits, lists, and reports — it **NEVER changes application code**. No fixes, no edits, no refactors, no config tweaks, no "quick fix while we're here" — not even bugs that look trivial or whose fix is obvious, and never "indirectly" (via scripts or tools touching app files).
1. The skill's entire write surface is exactly two things: the update to `.bugs/BUGS.md` (rules 12–15) and the report email (rule 16). Everything else is strictly read-only.
2. Fixing any listed bug is a separate task that happens ONLY on an explicit user order outside this skill (normal task flow). If asked mid-audit to also fix something, refuse, finish the audit, and point the user to order the fix separately.

## Where bugs live

1. All known bugs live in one file: `.bugs/BUGS.md`. It is the bug-list counterpart of `.tasks/TASKS.md`.
2. The file has exactly two sections, in this order: `## Audit state` and `## Bug list`.
3. `## Audit state` holds three fields — `Last audit commit:` (short + full sha), `Last audit date:` (ISO UTC), and `Hit:` (one line: where the NEXT audit starts analyzing).
4. `## Bug list` entries are checklist lines: `- [ ] **BUG-NNN [SEVERITY] One-line title**`, followed by indented `Found:` (file:line), `Evidence:` (short, concrete), and exactly one `Hint:` line.
5. Severity tags, pick the tightest fit: `[LOW-HANGING FRUIT]`, `[CRITICAL]`, `[IMPORTANT]`, `[MINOR]`.
6. New entries go at the TOP with the next free `BUG-NNN` (latest added first) — except a `[LOW-HANGING FRUIT]`, which always floats above everything (tackle those first).
7. Resolution = tick the checkbox `- [x]` in place, same style as finished tasks. Never delete or reword an entry; append a one-line dated note when ticking.

## Run flow

8. Read `.bugs/BUGS.md`. If it does not exist, create it with `## Audit state` seeded to the current HEAD (commit + date, `Hit: none yet`) and an empty `## Bug list`, then continue with the audit.
9. **Cooldown check (exit fast):** the audit runs at most once every 15 minutes. If `Last audit date` (in `## Audit state`, ISO UTC) is less than 15 minutes ago, do nothing. Report: "Cooldown active — last audit at `<date>`, bug hunt skipped." and stop. There is NO commit-based gate: every run past the cooldown performs the full audit, updates the list only if needed, and sends the report.
10. **Scope the audit:** start where the stored `Hit:` points; when there are commits newer than `Last audit commit`, `git diff --name-only <last-audit-commit>..HEAD` are the primary suspect files; otherwise sweep the house high-risk zones: `src/lib/stores/*`, `src/lib/api/**`, `src/routes/+layout.ts`, `hooks.server.ts`, cart/checkout flow, i18n keys used in code vs `messages/*.json`, SSR/prerender guards.
11. **Hunt:** find the next most important bug that is (a) not already listed in `## Bug list` (any checkbox state), (b) present in current code. Preference order: low-hanging fruit > critical > important. Every claim must cite exact file:line evidence of code actually read this run — no speculation.
12. **Tick resolved:** if you can now prove a listed open bug is fixed by current code (file:line proof), tick it `- [x]` with a one-line dated note. When fixes happen outside an audit run, any agent fixing a listed bug must tick it in the same session.
13. **Exactly one new bug per run.** If a real search finds nothing worth listing, add none — never invent filler; say so in the report.

## Persist the audit state

14. Immediately after the hunt: update `## Audit state` — `Last audit commit:` = HEAD of this run (short + full sha; anchors the next run's diff scope), `Last audit date:` = now (ISO UTC — this field activates the 15-minute cooldown, stamp it on EVERY run), `Hit:` = the new entry's Hint, or the carried-over Hit if no new bug.
15. The stored Hit must give the next audit a concrete starting point (file, function, or flow), not a vague area.

## Email the report (same pipeline as the contact form)

16. Right after the file is updated (the file is the source of truth; the email is the notification), send ONE email to `cubiqlabs@gmail.com`:
    - **Transport:** `POST $PUBLIC_API_BASE_URL/notifications/email` with header `Authorization: Bearer $CUBIQ_API_KEY` (both vars read from this repo's `.env`; never print or commit the key). This is the same backend mail service (`NotificationsService.send_email` → Mailgun, `SendEmailDto`) that the contact form's `notificationService.sendOnboardingEmailVerified` (`src/lib/api/services/notification.service.ts`) reaches via `/notifications/email/verified`; that sibling needs a live Cloudflare Turnstile widget token which a shell run cannot mint, so use the plain endpoint with the same app token the `/api/proxy` injects for the contact form.
    - **Payload (contact-form DTO):** `to`, `subject`, `html`, `text`, plus `telegramMessage` (same admin Telegram ping the contact form sends) — the `telegramMessage` must also state explicitly whether new bugs were found this run (yes/no) and compactly list the active (open, not-tackled) bugs, `▶ NEW`-marked on the just-added one.
    - **Template:** mirror the contact form branding in `buildHtml` (same file): dark `#111` card, orange `#F79319` gradient header, Malthaus logo `${app.productionBaseUrl}/favicon-96x96.png`, white table rows. Inner H2 title (under the gradient header, before the summary): **«Informe de bugs en Malthaus»**.
    - **Language:** the email (`html` and its plain `text` alternative) is written in **Spanish** (developer-readable); `.bugs/BUGS.md` entries and the `telegramMessage` stay in English.
    - **Structure:** right under the title, BEFORE the audit-state table, a short Spanish summary paragraph that ALWAYS states explicitly whether this run found new bugs (Sí/No), plus what was audited (commit + scope) and the total open-bug count — e.g. «Nuevos bugs: SÍ — 1 agregado. Se auditó el commit `38e53f2` (banderas + settings); quedan **2 bugs pendientes** en total.» On a run with no new bug: «Nuevos bugs: NO. Se auditó `<scope>`; siguen X bugs pendientes en total.» The email never claims resolution — an audit only reports findings.
    - **Rows:** Audit date, Commit, then **Open bugs** — ALL pending `- [ ]` entries from `## Bug list` (severity, title, file refs, Hint), new-first, with the just-added bug highlighted (HTML: `NEW` badge + orange `#F79319` left-border row; `text`: `▶ NEW` prefix; `telegramMessage` keeps `▶ NEW` too; no highlight on a run that added nothing), then the total open-bugs count row. No "resolved" row — that concept lives only in `.bugs/BUGS.md`.
    - **Subject:** `Informe de bugs en Malthaus` — fixed string, identical to the inner email title (no commit/date markers; keeps threading stable across runs).
17. If the send fails, keep the file changes, log the error, retry once, then report the failure. Never block the audit result on the email.

## After the run

18. Report to the user: new bugs found (yes/no), bugs ticked, new `BUG-NNN` added, new Hit, email sent/failed. Then **stop** — per the ⚠️ CRITICAL rule (top of this file): the bug hunter never fixes; fixing a listed bug is a separate, explicit user order (normal task flow).
19. House style: keep all output terse (1–2 sentences per item).
