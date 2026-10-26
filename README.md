# vibe-to-prod

**AI built your app in a weekend. Here's what it didn't tell you about your first 1,000 users.**

`vibe-to-prod` is a Claude skill (and a plain copy-paste checklist) that audits your web app and hardens it for real traffic, real money, and real mistakes. Built for vibe coders and junior engineers: every finding comes with what breaks, why it matters, and a one-line "Learn it" so you spot it yourself next time.

## What it checks

17 checks, run in this order. Safety nets first, tests before refactors, structural changes last.

| Phase | Checks |
|---|---|
| 1. Safety nets | Spend alerts, Backups and restore drill, Error tracking, Uptime, Rollback, Local vs production database |
| 2. Harness | Regression tests for every core flow |
| 3. Security | Never trust the client, Secrets, Webhooks, Rate limiting |
| 4. Correctness | Race conditions, Idempotency |
| 5. Performance | N+1 queries and indexes, Background jobs |
| 6. Structure | Abstraction, Normalization |

How it works:

1. **Recon:** maps your stack, database, hosting, paid services, and core user flows.
2. **Audit:** runs the checks and writes a `PROD_READY.md` scorecard (✅ Good, ⚠️ Risky, ❌ Broken, ➖ N/A, ❓ Needs you).
3. **Fix:** one check at a time, with your approval, a new test for each fix, and a commit per change.

Guardrails it always follows: it never connects to your production database, never edits a test just to make it pass, and asks before touching payments, auth, production data, or destructive migrations.

## Repo layout

```
vibe-to-prod/
├── skills/prod-ready/          # the Claude skill
│   ├── SKILL.md                # workflow, rules, check order, scorecard format
│   └── references/checks.md    # deep dive per check: why, look for, fix, verify
├── dist/prod-ready.skill       # packaged skill for one-click install in Claude.ai
├── PROMPTS.md                  # no-skill version: 17 copy-paste prompts
└── ARTICLE.md                  # the long-form write-up
```

## Install

**Claude.ai:** download [`dist/prod-ready.skill`](dist/prod-ready.skill) and upload it in your Claude skills settings.

**Claude Code (all projects):**
```bash
git clone https://github.com/<your-username>/vibe-to-prod.git
cp -r vibe-to-prod/skills/prod-ready ~/.claude/skills/
```

**Claude Code (one project):** copy `skills/prod-ready` into `.claude/skills/` in your repo.

**Cursor, Windsurf, Codex, other agents:** point the agent at `skills/prod-ready/SKILL.md`, or add it to your rules file. Or skip the skill entirely and use [`PROMPTS.md`](PROMPTS.md).

## Use

Full audit:
```
Is my app prod ready?
```

One check at a time:
```
Find every race condition in my app and fix it.
Which of my endpoints could someone call 1000 times a minute? Add rate limiting.
```

## Contributing

Stack-specific tips are the most valuable contribution (Supabase, Firebase, Rails, Django, Laravel, Convex, and so on). Open a PR against `skills/prod-ready/references/checks.md`, or open an issue with a production horror story the checklist would have caught.

## License

MIT
