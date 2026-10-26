# AI built your app. It didn't tell you these 17 things will break at 1,000 users.

AI built your app in a weekend. Here's what it didn't tell you about your first 1,000 users.

Your app works. You clicked through every page, signed up with three test emails, and paid yourself $1 with a test card. It's ready.

Then real users show up. They double click. They close the tab mid-payment. They open DevTools. One of them writes a script. Your AI bill triples overnight and you find out from your bank.

None of this is your AI's fault. It built exactly what you asked for. You just didn't ask for any of this.

So here are the 17 things to ask for, in the order to ask. Each one has what breaks, the prompt to paste, and the lesson to keep. All of it is packaged as a free Claude skill at the end.

**The order matters.** Safety nets first, because they are cheap and protect you while you change everything else. Tests before refactors. Structural cleanup last.

---

## Phase 1: Safety nets

### 1. Spend alerts
**What breaks:** A retry loop or one abusive user turns a $20 bill into $2,000 overnight. Most providers will let it happen.
> List every service I pay for, and show me how to set a spend alert (and a hard cap where possible) for each one.

**Learn it:** Every usage-billed API is a credit card attached to the internet.

### 2. Backups
**What breaks:** Your database dies, and you discover your plan never had backups, or you have one but have no idea how to restore it.
> What backups does my setup have? Walk me through restoring one into a test project.

**Learn it:** A backup you've never restored is a hope, not a backup.

### 3. Error tracking
**What breaks:** Checkout has been failing for 30% of users for a week. Nobody told you. They just left.
> Add error tracking to my frontend and backend and alert me when a new error shows up.

**Learn it:** Users don't report errors. They leave.

### 4. Uptime
**What breaks:** Your site is down for six hours and you learn about it from a reply on X.
> Add a health check endpoint and set up an uptime monitor that alerts my phone.

**Learn it:** You should always find out first.

### 5. Rollback
**What breaks:** A deploy breaks login at 2am and you're debugging live while users can't get in.
> If my next deploy breaks, how do I roll back in under a minute? Does this change touch the database?

**Learn it:** Rolling back code doesn't roll back your database. Make schema changes backward compatible.

### 6. Local vs production database
**What breaks:** You run a reset script locally. Your `.env` pointed at production. Every user is gone.
> Make sure my local code never connects to my production database. Set up a local database and turn every database change into a migration.

**Learn it:** Your laptop should be physically unable to hurt real users.

## Phase 2: The harness

### 7. Regression tests
**What breaks:** You ask AI to fix a button. It quietly breaks signup. You find out three days later.
> Write tests for every flow my users go through, and run them automatically every time anything changes.

**Pro tip:** Tell your AI to never change a test to make it pass. If a test really needs to change, it should ask you and show you why it matters. Otherwise your tests become decoration.

**Learn it:** Tests are what let you change things without fear.

## Phase 3: Security

### 8. Never trust the client
**What breaks:** Someone opens DevTools, changes the price from 49 to 0.01, and checks out. Or changes `/orders/123` to `/orders/124` and reads a stranger's order.
> Does my server trust anything my browser sends? Can user A see user B's data? Fix every place you find.

**Learn it:** Anything in the browser can be edited. The server decides prices, identity, and permissions.

### 9. Secrets
**What breaks:** Your OpenAI key ships inside the frontend bundle. Bots find it in hours.
> Are any secret keys in my frontend, my repo, or my git history? Move them and tell me what to rotate.

**Learn it:** Deleting a leaked key from the repo doesn't un-leak it. Rotate it.

### 10. Webhooks
**What breaks:** You grant Pro on the success page. People who close the tab never get it. People who cancel keep it forever. Someone who finds the success URL gets it free.
> Does any upgrade happen via a success page? Move it to a webhook and verify the signature.

**Learn it:** The success page is a thank-you note, not a receipt.

### 11. Rate limiting
**What breaks:** One script hammers your login endpoint, or your AI endpoint, 1,000 times a minute.
> Which of my endpoints could someone call 1,000 times a minute? Add rate limiting to them.

**Learn it:** Every public endpoint is an invitation. Limit the ones that cost money or guard accounts.

## Phase 4: Correctness

### 12. Race conditions
**What breaks:** Two requests arrive at the same millisecond. Both see "1 seat left." Both buy it.
> Find every race condition in my app and fix it.

**Learn it:** "Check, then write" is a bug waiting for traffic. Let the database enforce the rule.

### 13. Idempotency
**What breaks:** The network hiccups, the payment retries, the customer gets charged twice.
> Is my code idempotent? Fix every place it isn't.

**Learn it:** Everything will run twice eventually. Make twice do the same thing as once.

## Phase 5: Performance

### 14. N+1 queries
**What breaks:** Your dashboard runs one query per row. Fast with 10 rows, a timeout with 10,000.
> Find every N+1 query in my app and fix it. Add missing indexes and paginate long lists.

**Learn it:** A query inside a loop is a loop inside your database.

### 15. Background jobs
**What breaks:** Signup takes 8 seconds because it sends a welcome email, syncs to your CRM, and generates an avatar before responding.
> What does my app do inside a request that the user doesn't need to wait for? Move it to a background job.

**Learn it:** If the user doesn't need to see it now, they shouldn't wait for it now.

## Phase 6: Structure (only when tests pass and backups work)

### 16. Abstraction
**What breaks:** Your price calculation lives in four places. You fix a bug in one. The other three keep charging the wrong amount.
> Is my code well abstracted? If not, propose a plan and refactor in small steps.

**Learn it:** Extract on the third copy, not the first.

### 17. Normalization
**What breaks:** A user changes their email. Half your tables still have the old one.
> Can my database store contradictory data? Show me a migration plan before changing anything.

**Learn it:** A good schema makes bad data impossible, not just unlikely.

---

## Get it as a skill

I packaged all 17 into a free Claude skill called **prod-ready**. It maps your stack, audits every check, writes a scorecard (✅ ⚠️ ❌), then fixes things one at a time with your approval. It never touches your production database and never edits a test to make it pass.

Works with Claude.ai, Claude Code, Cursor, or any agent. There's also a copy-paste version if you don't use skills.

👉 github.com/shahbaz17/vibe-to-prod

Ship fast. Then make it survive.
