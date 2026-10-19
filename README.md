# skills

Skills, skills, skills for harnessing these things.

## Available Skills

| Skill | Description |
| --- | --- |
| [tasks](skills/tasks/SKILL.md) | How tasks work in this repo — the .tasks/ folder, the task list in .tasks/TASKS.md, and the status markers agents must keep updated. |
| [git-commit-workflow](skills/git-commit-workflow/SKILL.md) | Rules for committing, bumping versions, and pushing to git. |
| [borum-writer](skills/borum-writer/SKILL.md) | Write blog posts, updates, logs, or essays in the Gonzo Coder / Borum voice. |
| [security-audit-runner](skills/security-audit-runner/SKILL.md) | Automated security audits powered by the Cloudflare security-audit skill via headless opencode, with Telegram notifications. |
| [youtube-search](skills/youtube-search/SKILL.md) | Find and verify exact-whisky YouTube review/tasting videos across languages using multiple search sources. |
| [bug-hunter](skills/bug-hunter/SKILL.md) | Read-only repo bug audit loop over `.bugs/BUGS.md`: ticks fixed bugs, adds at most one new bug per run, emails a Spanish report. |
| [bug-solution-planner](skills/bug-solution-planner/SKILL.md) | Proposal-only bug solver: writes a thorough solution to `.bugs/solutions/SOLUTION-NNN.md`, indexes it, emails the proposal; never implements. |
| [bug-implementer](skills/bug-implementer/SKILL.md) | Implements ONE approved solution from `.bugs/solutions/SOLUTION-NNN.md` after explicit operator approval, verifies, then ticks the bug. |

## Install

Install a single skill with the [Skills CLI](https://skills.sh):

```bash
npx skills add https://github.com/minimo-io/skills --skill tasks
npx skills add https://github.com/minimo-io/skills --skill git-commit-workflow
npx skills add https://github.com/minimo-io/skills --skill borum-writer
npx skills add https://github.com/minimo-io/skills --skill security-audit-runner
npx skills add https://github.com/minimo-io/skills --skill youtube-search
npx skills add https://github.com/minimo-io/skills --skill bug-hunter
npx skills add https://github.com/minimo-io/skills --skill bug-solution-planner
npx skills add https://github.com/minimo-io/skills --skill bug-implementer
```

Add `--global` for a user-level installation.
