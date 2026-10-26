---
name: prod-ready
description: Audit and harden a web app so it survives its first few thousand real users. Runs a phased, teach-as-you-go checklist covering spend alerts, backups, error tracking, uptime, rollbacks, local vs prod database and migrations, regression tests, never-trust-the-client, secrets, webhooks, rate limiting, race conditions, idempotency, N+1 queries, background jobs, abstraction, normalization. Use this skill whenever the user asks if their app is "production ready", "ready to launch", "ready for users" or "ready to scale", wants a launch or pre-launch checklist, a hardening pass, or a security/reliability audit of their own codebase, or asks about any single item on the list (race conditions, N+1, rate limiting, Sentry, webhooks, idempotency, backups, rollbacks, migrations, spend alerts, RLS) in their app, even if they do not name this skill. Especially useful for vibe-coded or AI-generated apps and junior engineers shipping their first real product.
---

# Prod Ready

Take an app that works on the builder's laptop and make it survive real users, real money and real mistakes. The target is "boringly reliable at a few thousand users", not "Google scale". Do not over-engineer: no Kubernetes, no microservices, no new infrastructure the app does not need. The cheapest fix that removes the risk wins.

## Who you are working with

Often a vibe coder or a junior engineer. They can ship features but may never have been paged at 2am. So for every finding, say in one plain sentence what breaks, for whom, and what it costs (money, data, trust). After each fix, add one "Learn it" line: the pattern to recognize next time. The goal is a safer app and a better engineer.

## Ground rules

1. **Audit before you fix.** Report findings for a check first, then propose fixes. Apply them once the user agrees. If the user says "just fix everything", still pause for anything in rule 2.
2. **Ask first** before: anything that touches production data or production dashboards, destructive migrations (drop, rename, type change), changing auth or payment behavior, adding a paid service, deleting code, or changing an existing test.
3. **Never connect to the production database.** If the only database URL available points at production, stop and do check 6 (Local vs production database) before anything else.
4. **Tests are the contract.** Never edit a test just to make it pass. If a test genuinely needs to change, stop and show the user: what the test expects today, what it would expect after your change, and why that difference matters. Wait for a yes.
5. **Show your search.** When a check says "find every X", tell the user how you searched (patterns, directories, files) so they can trust the coverage, and list anything you could not verify from code.
6. **Small, reversible steps.** One check per commit (or branch). Run the test suite after each change.
7. **Be honest about "not applicable".** Do not invent problems or bolt on tooling the app does not need. A clean result is a good result.
8. **Things only the user can do** (dashboard settings, billing, rotating keys, alert rules) go in a separate list with exact steps. Mark them as "needs you" until the user confirms they are done.

## Step 0: Recon (always do this first)

Build a short map of the app and show it to the user before running checks. Discover from the code what you can, and only ask about what code cannot tell you.

- Stack: frontend, backend, framework, language, package manager
- Database and ORM, where it is hosted, how schema changes happen today (migrations, dashboard edits, `db push`)
- Hosting and deploys: platform, how a deploy is triggered, whether migrations run on deploy
- Auth provider, payment provider, and every other paid API (AI, email, SMS, storage, maps)
- Environments: which env files exist and which database each one points to
- Existing tests, CI, error tracking, monitoring
- Core user flows (signup, login, the main action, pay, cancel, delete account): infer from routes and pages, then confirm with the user

## The checklist, in order

Order matters. Safety nets come first because they are cheap, carry no code risk, and protect the user while everything else changes. Tests come before any refactor. Structural changes come last.

| # | Phase | Check | The question it answers |
|---|---|---|---|
| 1 | Safety nets | Spend alerts | Will I find out before a bug or abuser runs up a huge bill? |
| 2 | Safety nets | Backups and restore drill | If the database dies today, can I actually get the data back? |
| 3 | Safety nets | Error tracking | Will I know about errors before users tell me (or silently leave)? |
| 4 | Safety nets | Uptime and health check | Will I know the site is down before a user tweets about it? |
| 5 | Safety nets | Rollback | If the next deploy breaks, can I undo it in under a minute? Does it touch the database? |
| 6 | Safety nets | Local vs production database | Can my laptop ever touch real user data? Is every schema change a migration? |
| 7 | Harness | Regression tests | Do my core flows have tests that run on every change? |
| 8 | Security | Never trust the client | Does the server believe anything the browser says (prices, user IDs, roles)? Can user A see user B's data? |
| 9 | Security | Secrets | Are any keys in the frontend bundle, the repo, or git history? |
| 10 | Security | Webhooks | Are upgrades and payments granted by a verified webhook, not a success page? |
| 11 | Security | Rate limiting | Which endpoints could someone call 1000 times a minute? |
| 12 | Correctness | Race conditions | What happens when two requests do the same thing at the same moment? |
| 13 | Correctness | Idempotency | If a request, job, or webhook runs twice, does it do the thing twice? |
| 14 | Performance | N+1 queries and indexes | Do my pages run one query per row? Are hot columns indexed? Are lists paginated? |
| 15 | Performance | Background jobs | What work makes users wait that they do not need to wait for? |
| 16 | Structure | Abstraction | Is logic duplicated or tangled into UI and route handlers? |
| 17 | Structure | Normalization | Can my schema store contradictory data? |

Before working on any check, read its section in `references/checks.md`. Each section explains why it matters, what to search for, how to fix it, how to verify the fix, and what to ask before touching.

Checks 16 and 17 are gated: only run them when check 7 has passing tests for the core flows and check 2 has a verified backup. Without those, refactors and schema changes are how working apps get broken.

## Modes

- **Full audit** ("is my app prod ready?"): recon, run all 17 checks in audit mode, write the scorecard, then fix in phase order with approval.
- **Single check** ("add rate limiting", "find my N+1s"): do a light recon covering only what that check needs, run it, and flag any missing prerequisite (for example, no tests before a refactor, no local database before writing migrations).
- **Re-run** (before a launch, or after a big feature): run the audit again and compare against the existing `PROD_READY.md`.

## Output: the scorecard

After the audit, write `PROD_READY.md` at the repo root and give a short summary in chat. Use this structure:

```markdown
# Prod Ready Scorecard
App: <name> | Stack: <short stack> | Audited: <date>

## Summary
| # | Check | Status | Risk | Effort | Fix |
|---|---|---|---|---|---|
| 1 | Spend alerts | ❓ Needs you | High | 15 min | Set budget alerts on 4 services (steps below) |
| 8 | Never trust the client | ❌ Broken | Critical | 1 hr | Price is read from request body in api/checkout.ts:42 |
...

Status: ✅ Good, ⚠️ Risky, ❌ Broken, ➖ N/A, ❓ Needs you (lives in a dashboard the code cannot see)

## Fix this week (top 3)
1. ...

## Only you can do these
- [ ] ... (exact click path or command)

## Details
### 8. Never trust the client
What I searched: ...
Findings: file:line, what breaks, who it affects
Fix: ...
Learn it: ...
```

Rank risk by blast radius: losing money or user data is Critical, a broken core flow is High, slowness or annoyance is Medium, polish is Low. Keep the file updated as fixes land so it doubles as a launch log.

## Fix loop

For each approved fix:

1. Explain the problem in plain words, with the exact file and line.
2. Make the smallest change that removes the risk.
3. Add a test that would have caught it (a new test, not a changed expectation in an old one).
4. Run the full test suite.
5. Commit with a message like `prod-ready(rate-limit): limit login to 5/min per IP`.
6. Update `PROD_READY.md`.
7. Write the "Learn it" line.
