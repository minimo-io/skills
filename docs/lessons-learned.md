# Lessons learned

## Tasks skill lives in `.tasks/`

- The `tasks` skill stores task files in `.tasks/` (hidden folder), not `tasks/`.
- The authoritative task list lives in `.tasks/TASKS.md`, not in `AGENTS.md`.
- `AGENTS.md` carries only a single pointer line to `.tasks/TASKS.md`.
- When creating, picking up, or finishing a task, reference `.tasks/` and keep the file's `Status:` line and the `.tasks/TASKS.md` list in sync.
- When migrating a repo to use the skill, move any `## Tasks` list out of `AGENTS.md` into `.tasks/TASKS.md`, keep all status markers and ordering identical, and leave AGENTS.md with only the pointer line.

## Installable skills live in a `skills/` container

- For the `npx skills` CLI to discover and install individual skills, each skill folder must sit under a top-level `skills/` container, one level deep: `skills/<skill-name>/SKILL.md`. This matches the cloudflare/security-audit-skill layout.
- Users install one skill at a time: `npx skills add <repo> --skill <name>`.
- Install URLs in `README.md` must point to the canonical repo `https://github.com/minimo-io/skills`. The legacy `borumbombum/skills` repo is stale and misses recent skills.
- This repo has no version file, tags, or bump convention. Do not add a version-bump mechanism; commit changes without a bump.
- Pushing to GitHub failed over SSH (unrecognized identity). Fixed by switching the remote to HTTPS (`git@github.com:...` → `https://github.com/...`) so `gh`'s credentials are used.

## Skills can bundle executable scripts

- The `security-audit-runner` skill carries a shell script under `skills/security-audit-runner/scripts/`. The `npx skills` layout tolerates extra files next to `SKILL.md`; the script must be `chmod +x` and referenced with paths relative to the skill root (`$SCRIPT_DIR`).
- When a script that once lived in a standalone repo is converted into a skill, the script's default "project root" must change. `$SCRIPT_DIR/..` no longer points at the target project once the script sits in `skills/<name>/scripts/`; default to `$PWD` instead and keep an explicit `TARGET` override.

## Bug skills imported from malthaus-app

- `bug-hunter`, `bug-solution-planner`, and `bug-implementer` were copied verbatim from the malthaus-app project (`malthaus-app/.agents/skills/`) and are kept as-is here.
- They are intentionally project-coupled: Malthaus email branding and recipient, the `POST $PUBLIC_API_BASE_URL/notifications/email` + `$CUBIQ_API_KEY` transport, SvelteKit scope paths, `npm run check`, and the Tauri version-bump rule. Each project that installs them must adapt those specifics.
- Shared `.bugs/` domain: `BUGS.md` is owned by `bug-hunter` (only it adds/ticks bugs), `SOLUTIONS.md` indexes `solutions/SOLUTION-NNN.md` proposals, and `bug-implementer` applies exactly one approved solution after explicit operator approval.
