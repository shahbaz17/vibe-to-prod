# Checks reference

Read only the section for the check you are working on. Every section has the same shape: **Why** (say this to the user in plain words), **Look for**, **Fix**, **Verify**, and **Ask first** where relevant. Stack hints name common vibe-coding stacks (Next.js, Vite/React, Supabase, Firebase, Prisma, Drizzle, Stripe, Vercel, Railway, Render, Fly) but the ideas apply everywhere. Dashboards and settings move around, so when you give click paths, tell the user to confirm against the provider's current docs, and look them up if you have web access.

## Contents
1. Spend alerts
2. Backups and restore drill
3. Error tracking
4. Uptime and health check
5. Rollback
6. Local vs production database
7. Regression tests
8. Never trust the client
9. Secrets
10. Webhooks
11. Rate limiting
12. Race conditions
13. Idempotency
14. N+1 queries and indexes
15. Background jobs
16. Abstraction
17. Normalization

---

## 1. Spend alerts

**Why:** A bug, an infinite retry loop, or one abusive user can turn a $20 bill into $2,000 overnight, and most providers will happily let it happen.

**Look for:** Every paid service the app touches. Sources: dependencies (`package.json`, `requirements.txt`, `go.mod`), env var names (`*_API_KEY`, `STRIPE_*`, `OPENAI_*`, `ANTHROPIC_*`, `RESEND_*`, `TWILIO_*`, `AWS_*`, `SUPABASE_*`), SDK imports, and infra config (`vercel.json`, `fly.toml`, `render.yaml`, `railway.json`, Terraform). Do not forget the host, the database, the domain, and email.

**Fix:** Produce a table: Service | What the app uses it for | How it bills (usage, seats, flat) | Where to set an alert | Hard cap available? Then give step-by-step instructions per service. Prefer a hard cap where one exists (many AI providers and some hosts offer spend limits), and a budget alert at roughly 50% and 90% of expected monthly spend everywhere else. Also add code-level guardrails on anything usage-billed: `max_tokens` on AI calls, per-user daily quotas on AI or SMS endpoints, and rate limits (check 11).

**Verify:** The user confirms each alert is set. Mark ❓ until they do.

## 2. Backups and restore drill

**Why:** A backup you have never restored is a hope, not a backup.

**Look for:** The database provider and plan, whether automated backups or point-in-time recovery are on, retention length, and whether anything lives outside the database (user uploads in object storage are often not backed up at all).

**Fix:**
- Turn on automated backups or point-in-time recovery if the plan supports it. If it does not, schedule a logical dump (for Postgres, `pg_dump`) to separate storage owned by a different account or provider.
- Run a restore drill: restore the latest backup into a new, throwaway project, point a local copy of the app at it, compare row counts on the key tables, and time the whole thing.
- Write `RESTORE.md`: the exact steps, who has access, and how long it took.

**Verify:** A restore was completed and timed.

**Ask first:** Anything inside the production console. The user should click the buttons; you write the steps.

## 3. Error tracking

**Why:** Users do not report errors. They leave.

**Look for:** Existing error tracking, global error handlers, React error boundaries, unhandled promise rejections, and swallowed errors: `catch {}`, `catch (e) { console.log(e) }`, `.catch(() => null)`.

**Fix:**
- Add Sentry (or the equivalent the user prefers) to both frontend and backend.
- Upload source maps so stack traces are readable. Tag releases with the deploy version and tag the environment so local noise stays out of production alerts.
- Attach a user ID to events, not emails or other personal data unless the user wants that. Scrub tokens and secrets.
- Alert rules: email or Slack on every new issue, plus a spike alert on error rate.
- Replace swallowed errors with "capture and rethrow" or "capture and show the user a real message".

**Verify:** Throw a deliberate test error on the frontend and the backend, confirm each arrives with a readable stack trace and that the alert fires, then remove the test error.

## 4. Uptime and health check

**Why:** You should not learn your site is down from a user's tweet.

**Fix:** Add a `/health` (or `/api/health`) endpoint that does a cheap database check (`SELECT 1`) and returns 200 or 503. Set up an external uptime monitor (several have free tiers) that hits the homepage and the health endpoint every 1 to 5 minutes and alerts the user's phone.

**Verify:** Point the monitor at a broken URL briefly and confirm the alert arrives.

## 5. Rollback

**Why:** At 2am you want one button, not a debugging session.

**Look for:** Whether the host supports instant rollback to a previous deployment (most modern platforms do in some form; confirm for this one), whether migrations run automatically on deploy, and whether recent migrations are backward compatible with the previous code.

**Fix:**
- Write `ROLLBACK.md` with the exact steps to roll code back in under a minute on this platform.
- Answer "does the next deploy touch the database?" literally: diff the branch against the production branch for migration files and schema changes, and report yes or no with the list.
- If it does, use **expand and contract**: first add new columns or tables in a way the old code ignores, deploy the code that uses them, backfill, and only remove old columns in a later deploy once nothing reads them. Never ship a destructive migration in the same deploy as the code that depends on it, because rolling back the code will not roll back the schema.
- For risky features, put them behind a simple feature flag (an env var is fine) so "rollback" can mean flipping a switch.

**Verify:** Practice a rollback on a preview or staging deploy.

## 6. Local vs production database

**Why:** One wrong command on your laptop, like a reset or a seed script, can wipe real users' data.

**Look for:** `.env`, `.env.local`, and similar files that contain production URLs; hardcoded connection strings; seed or reset scripts with no environment guard; schema changes made by clicking in a dashboard (which creates drift between environments); `prisma db push` or equivalents used against production; no migrations folder at all.

**Fix:**
- Set up a local database (a Docker Postgres container, or the provider's local dev tooling such as the Supabase CLI).
- Add `.env.example` with placeholder values. Real production credentials live only in the host's environment settings.
- Add a guard to seed, reset, and destructive scripts that refuses to run when the database host looks like production or `NODE_ENV=production`.
- Create a baseline migration that matches the current production schema, then make every future schema change a committed migration file. Migrations run through the deploy pipeline or CI, not by hand from a laptop.

**Verify:** No production URL in the repo or local env files. A fresh clone can run migrate, seed, and start with nothing but `.env.example` filled in locally.

**Ask first:** Generating the baseline by introspecting production. It is read-only, but confirm with the user and use a read-only credential if one exists.

## 7. Regression tests

**Why:** Tests let you, and your AI, change things without breaking what users rely on. Without them every fix is a gamble.

**Fix:**
- Take the core flows from recon and confirm the list with the user. Prioritize money, auth, and the main action of the app over everything else.
- Write end-to-end tests (Playwright is a good default for web apps) for the top flows, and integration tests for API handlers that hold business logic. Do not chase a coverage percentage.
- Tests run against a test database, never production. Third parties run in test mode or are stubbed (payment test mode, a fake email sender).
- Automate: a single `test` command, plus CI on every pull request (GitHub Actions is fine). While using this skill, run the suite after every change.

**The rule:** Never change a test to make it pass. If a test really needs to change, ask in this shape: "Test `<name>` expects A. After my change it would get B. This matters because ... Should B be the new behavior?"

**Verify:** Deliberately break one flow and confirm a test fails, then undo.

## 8. Never trust the client

**Why:** Anything in the browser can be edited. DevTools is a free hacking tool, and every request can be replayed with different values.

**Look for:**
- Prices, totals, discounts, quantities, or plan names sent from the client and used by the server.
- `userId`, `role`, `isAdmin`, or `orgId` read from the request body or query instead of the server-side session.
- Insecure direct object references: does `/api/orders/123` check that order 123 belongs to the logged-in user? Try the same for every route with an ID in it.
- Authorization that only exists in the UI (a hidden button) and not on the server.
- Supabase: tables with Row Level Security off or with permissive policies, or the service role key used anywhere in frontend code. Firebase: security rules that allow broad read or write.
- No server-side validation of inputs. Client-side form validation does not count.
- Mass assignment: `update(req.body)` or spreading the body straight into a database write.
- File uploads checked for type or size only in the browser.
- Redirects to a URL taken from user input (open redirect).

**Fix:** Derive identity only from the session. Look up prices server-side from a product or price ID. Validate every input with a schema (zod, valibot, pydantic). Scope every query by owner (`where: { id, userId: session.user.id }`). Turn on RLS with policies per table. Allowlist the fields a user may update. Allowlist redirect targets.

**Verify:** Write tests where user B tries to read and modify user A's data and gets 403 or 404, and where a tampered price is ignored.

## 9. Secrets

**Why:** A leaked key means someone else spends your money or reads your users' data, and bots scan public repos for keys within minutes.

**Look for:** Secrets in client-exposed env vars (`NEXT_PUBLIC_*`, `VITE_*`, `EXPO_PUBLIC_*` holding anything secret), committed `.env` files, keys in git history (`git log -p` with a search for key patterns, or a scanner like gitleaks), keys printed to logs, server-only keys imported in frontend code.

**Fix:** Move secret usage server-side. Add env files to `.gitignore`. Add a secret scanner to CI or a pre-commit hook. **Any key that was ever exposed must be rotated**: removing it from the repo does not un-leak it.

**Ask first:** Rotation. It can briefly break production, and the user does it in each provider's dashboard. Give them the list and the order.

## 10. Webhooks

**Why:** The success page lies. Users close the tab before it loads, some payments settle later, and refunds, cancellations, and failed renewals happen entirely outside your app. If access is granted by the success page, people can get paid features without paying, and people who cancel keep them.

**Look for:** Entitlements granted on a success or redirect page (for example `/success?session_id=...`); existing webhook handlers; whether signatures are verified; whether the raw request body is available (body parsers often break signature checks); handling of cancellation, refund, dispute, and failed payment events; duplicate event handling; slow handlers.

**Fix:**
- Grant and revoke access only from verified webhook events. The success page just says "processing" and polls your own database.
- Verify the signature with the provider's SDK using the raw body and the webhook secret.
- Store processed event IDs with a unique constraint so redelivered events are ignored (see check 13).
- Return 2xx quickly and push heavy work to a background job (check 15).
- Events can arrive out of order. When in doubt, fetch the current state of the subscription or payment from the provider's API instead of trusting the event payload's order.

**Verify:** Use the provider's CLI or dashboard to send test events. A bad signature returns 400. Replaying the same event does not grant access twice. A cancellation event actually removes access.

## 11. Rate limiting

**Why:** Without limits, one script can brute-force logins, drain your AI budget, or spam your signup form with fake accounts.

**Look for and rank** endpoints by damage:
- Auth: login, signup, password reset, OTP and magic link sends.
- Anything that costs money per call: AI, SMS, email.
- Expensive reads: search, exports, reports.
- Public forms: contact, waitlist, comments.
- File uploads.
- Webhook endpoints are the exception: protect them with signature verification, not rate limits.

**Fix:** Limit per IP and per user, stricter on auth and paid endpoints. Propose a table of endpoint and limit for the user to approve. On serverless hosts, in-memory counters do not work because each request can hit a different instance; use a shared store (a hosted Redis, the database, or the platform's firewall or WAF rules). Return 429 with a `Retry-After` header. Add a bot challenge to signup if bots are a real problem.

**Verify:** A small script that loops requests against each limited endpoint gets 429 after the expected count.

## 12. Race conditions

**Why:** Two requests arriving at the same moment both read "1 seat left", and both buy it. This almost never shows up when one person tests the app, and shows up constantly with real traffic and double clicks.

**Look for:** Read-then-write patterns:
- Check a balance or credit count, then deduct it in a separate query.
- Check whether something exists, then insert it (usernames, one-per-user rows), with no unique constraint behind it.
- Check whether a coupon or invite was used, then mark it used.
- Counters computed in app code (`count = row.count + 1`) and written back.
- Inventory, seats, credits, quotas.
- A webhook and a success page both writing the same record.
- Scheduled jobs that can overlap with themselves.
- Forms that can be double submitted.

**Fix:** Let the database enforce the rule: unique constraints and check constraints (for example `credits >= 0`) are the last line of defense. Use atomic updates, for example `UPDATE accounts SET credits = credits - 1 WHERE id = $1 AND credits >= 1`, then check the affected row count. Use transactions with row locks (`SELECT ... FOR UPDATE`) where a multi-step change must be all or nothing. Use upserts instead of check-then-insert. Disable submit buttons while pending and send an idempotency key (check 13). Give scheduled jobs a lock.

**Verify:** A test fires many concurrent requests (for example with `Promise.all`) and asserts the invariant holds: no negative balance, no duplicate row, no oversold seat.

## 13. Idempotency

**Why:** Doing something twice should have the same effect as doing it once, because things will run twice: networks retry, users double click, webhooks are redelivered, and jobs are retried after a crash. If an action is not idempotent, fix it.

**Look for:** Payment or charge creation, order creation, credit grants, emails and notifications, webhook handlers, background job handlers, and any POST the client may retry.

**Fix:**
- Idempotency keys: the client generates a unique ID per user action, the server stores the key and the result with a unique constraint, and a repeat returns the stored result instead of acting again.
- Pass idempotency keys through to providers that support them (many payment APIs do).
- Use natural keys and upserts where they exist (one subscription row per user).
- Jobs check "already done?" before doing work, and record completion in the same transaction as the work when possible.

**Verify:** Calling the same action twice with the same key produces one effect.

## 14. N+1 queries and indexes

**Why:** A page that runs one query per row is fast with 10 rows and falls over with 10,000. Missing indexes do the same thing more quietly.

**Look for:**
- Queries inside loops: `for (...) { await db... }`, `items.map(async (i) => db...)`.
- ORM lazy loading of relations while rendering lists. GraphQL resolvers that query per item without batching.
- Columns used in `WHERE`, `JOIN`, or `ORDER BY` on hot paths without an index, especially foreign keys (Postgres does not index them automatically).
- Queries with no `LIMIT`, and list endpoints with no pagination.

**Fix:** Use the ORM's include or join, or batch with `WHERE id IN (...)`, or a dataloader. Add indexes through migrations. Paginate lists (cursor pagination for anything that grows). Turn on query logging locally and count queries per request before and after.

**Verify:** Report query count per page before and after, and run `EXPLAIN` (or the database's equivalent) on the hottest queries.

**Ask first:** Adding an index to a large production table can lock it. On Postgres use `CREATE INDEX CONCURRENTLY`, outside a transaction.

## 15. Background jobs

**Why:** Users should not wait for work they do not need to see. Slow requests also time out, and a timeout halfway through can leave work half done.

**Look for** inside request handlers: sending emails, third-party API calls whose result is not shown immediately, image or video processing, PDF generation, AI calls that run in the background from the user's point of view, analytics events, fan-out to many recipients, bulk imports. Also look for external calls with **no timeout**.

**Fix:** Move that work to a job queue that fits the stack, preferring options that need no new infrastructure (a managed queue service or a Postgres-backed queue). Give jobs retries with backoff, make them idempotent (check 13), and send failures to error tracking (check 3). Add a timeout to every external call. On serverless platforms, work started after the response is sent may be killed; use the platform's supported "after response" mechanism or a real queue.

**Verify:** Compare request latency before and after, and confirm a deliberately failing job shows up in error tracking.

## 16. Abstraction

**Gate:** Only after check 7 has passing tests for the core flows.

**Why:** Duplicated logic drifts: you fix a bug in one copy and the other three keep it. Clean seams also make the app easier to test and easier for an AI to change safely.

**Look for:** The same logic copy-pasted (auth checks, price calculation, permission rules, data access); business logic inside UI components or route handlers; giant files; third-party SDK calls scattered across the codebase.

**Fix:** Propose a plan before refactoring. Extract on the third repeat, not the first. Keep route handlers thin and move logic into plain functions or a service layer. Wrap each external service (payments, email, AI) in one module so it can be swapped and mocked. Do not add patterns for their own sake. Refactor in small steps and run tests after each.

## 17. Normalization

**Gate:** Last. Requires passing tests (check 7), a verified backup (check 2), and migrations in place (check 6).

**Why:** The goal is not textbook normal forms. It is a schema that cannot hold contradictory data. Deliberate denormalization (a cached count, a snapshot of the price at purchase time) is fine when it is intentional and documented.

**Look for:** The same fact stored in several places that can drift apart (a user's email copied into every order); comma-separated lists or JSON blobs holding relational data; missing foreign keys; missing `NOT NULL` or unique constraints; free-text columns that should be enums; money stored as floats.

**Fix:** Plan each change as expand and contract (see check 5): add the new structure, backfill with a script, switch reads and writes, then drop the old structure in a later deploy. Add foreign keys and constraints. Store money as integer minor units (cents) or a fixed-precision decimal.

**Ask first:** Always. Show the migration plan, the backfill, and the rollback plan before running anything.
