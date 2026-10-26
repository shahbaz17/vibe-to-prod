# The 17 prompts

No skill install needed. Paste these into Claude, Cursor, or any coding agent, **in order**. Start every session with the house rules.

## House rules (paste first)

```
Before changing anything: map my stack, database, hosting, paid services, and core user flows, and show me.
For each task: audit first, show me what you found with file and line, and tell me how you searched. Then propose fixes and wait for my OK.
Never connect to my production database.
Never edit a test just to make it pass. If a test really needs to change, show me the old vs new expectation and why it matters, and wait.
Ask before touching payments, auth, production data, or destructive migrations.
One fix per commit. Run the tests after every change.
After each fix, explain the problem in one plain sentence and give me a one-line lesson so I can spot it next time.
```

## Phase 1: Safety nets

**1. Spend alerts**
```
List every service I pay for or could be billed by (hosting, database, AI, email, SMS, storage, domains). For each, tell me how it bills and show me step by step how to set a spend alert, and a hard cap where one exists. Also add max token limits and per-user quotas on any endpoint that costs money per call.
```

**2. Backups**
```
What backups does my setup have, including user uploads? Turn on whatever is missing, then walk me through restoring the latest backup into a throwaway test project and checking the data. Write the steps to RESTORE.md.
```

**3. Error tracking**
```
Add error tracking (Sentry) to my frontend and backend with readable stack traces, release and environment tags, and no secrets or personal data. Alert me when a new error shows up. Find every place errors are silently swallowed and fix them.
```

**4. Uptime**
```
Add a /health endpoint that checks the database, and show me how to set up an external uptime monitor that alerts my phone when the site or the health check goes down.
```

**5. Rollback**
```
If my next deploy breaks, how do I roll back in under a minute on my host? Does my current branch touch the database? If it does, make the change backward compatible (expand then contract) so a code rollback is safe. Write the steps to ROLLBACK.md.
```

**6. Local vs production database**
```
Make sure my local code can never connect to my production database. Set up a local database, add .env.example, guard every seed and reset script against production, and turn every database change into a committed migration from now on.
```

## Phase 2: Harness

**7. Regression tests**
```
List every flow my users go through and confirm it with me. Write tests for them, money and auth first. Run them automatically on every change and in CI. Never change a test to make it pass: if one truly needs to change, ask me and show me why it matters.
```

## Phase 3: Security

**8. Never trust the client**
```
Does my server trust anything the browser sends: prices, totals, user IDs, roles, plan names? Can user A read or change user B's data by changing an ID? Is Row Level Security or the equivalent on for every table? Fix every place you find and add tests that try to break in.
```

**9. Secrets**
```
Are any secret keys in my frontend bundle, my repo, my logs, or my git history? Move them server-side, add a secret scanner, and give me the list of keys I need to rotate, in order.
```

**10. Webhooks**
```
Find every place payments or upgrades are handled. Is access granted on a success page or by a webhook? Move it to a verified webhook (check the signature with the raw body), handle cancellations, refunds, and failed payments, and make sure a replayed event does nothing twice.
```

**11. Rate limiting**
```
Which of my endpoints could someone call 1000 times a minute? Rank them by damage (auth, anything that costs money, expensive queries, public forms) and add rate limits that work on my host. Show me the limits before applying them.
```

## Phase 4: Correctness

**12. Race conditions**
```
Find every race condition in my app: check-then-write patterns, counters, credits, inventory, coupons, double submits, overlapping jobs. Fix them with database constraints, atomic updates, or transactions, and add tests that fire concurrent requests.
```

**13. Idempotency**
```
If any request, job, or webhook in my app runs twice, does it do the thing twice? Find every place that isn't idempotent (payments, orders, credits, emails) and fix it with idempotency keys or upserts.
```

## Phase 5: Performance

**14. N+1 and indexes**
```
Find every N+1 query in my app and fix it. Also find hot columns with no index and list endpoints with no pagination. Show me query counts per page before and after.
```

**15. Background jobs**
```
What does my app do inside a request that the user doesn't need to wait for? Move it to a background job with retries, make the jobs idempotent, and add timeouts to every external API call.
```

## Phase 6: Structure (only when tests pass and backups are verified)

**16. Abstraction**
```
Where is logic duplicated or tangled into UI components and route handlers? Propose a plan first, then refactor in small steps, running tests after each.
```

**17. Normalization**
```
Can my database store contradictory data? Look for duplicated facts, missing foreign keys and constraints, JSON blobs or comma lists holding relational data, and money stored as floats. Show me a migration plan with backfill and rollback before running anything.
```
