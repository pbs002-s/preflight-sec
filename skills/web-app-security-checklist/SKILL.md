---
name: web-app-security-checklist
description: A detailed 20-point security checklist for web apps, with extra focus on apps that use AI APIs, payments, and user accounts. Use when building, reviewing, or deploying a web app, or when asked for a security check, pre-launch review, "is this safe to ship?", or help with leaked keys, access control, rate limiting, uploads, or payment verification.
---

# Web App Security Checklist

A practical, beginner-friendly checklist to review a web app before launch. For each item you get: why it matters, what to check, how to test it, and how to fix it.

## How to run this skill

1. **Learn the app first.** Find out the stack, where it is hosted, and whether it has: user accounts, admin features, file uploads, payments, AI API calls, a database or storage service. Skip items that clearly do not apply and say so.
2. **Inspect the real code and config.** Read the repo, `.gitignore`, env files (names only), routes and API handlers, auth code, database rules, and deployment config.
3. **Mark every item** as `PASS`, `FAIL`, `PARTIAL`, `N/A`, or `NOT CHECKED` (with the reason).
4. **Rank fixes by severity** (see below) and give the smallest fix that works.
5. **Finish with the report template** at the bottom.

## Rules for the agent (important)

- **Never print a full secret.** If you find a key, show only the first 4 and last 2 characters, plus the file and line.
- **Only test apps the user owns** or has permission to test. Prefer local or staging, not production.
- **Ask before destructive actions**: rewriting Git history, deleting files or branches, changing production settings, or revoking keys.
- **Do not claim the app is "secure".** Say what was checked, what passed, and what was not covered.
- **Keep fixes minimal and simple.** Prefer built-in framework features and well-known libraries over custom security code.
- **Do not invent findings.** If you cannot verify something, mark it `NOT CHECKED`.

## Severity guide

| Level | Meaning | Examples |
|---|---|---|
| Critical | Fix before anything else. Direct data theft, account takeover, or money loss. | Leaked live secret key, public database, missing access checks, payment bypass, SQL injection |
| High | Likely to be abused soon. | No rate limit on login or AI endpoints, stored XSS, unrestricted uploads |
| Medium | Weakens defenses. | Missing security headers, verbose errors, no backup restore test |
| Low | Hardening and cleanup. | Minor logging issues, missing `.env.example` |

---

## Group 1: Secrets

### 1. Keep secret API keys out of your frontend
**Why:** Anything sent to the browser (JavaScript bundles, HTML, network requests) can be read by anyone.
**Check:**
- Secret keys (AI providers, payment secret keys, email or SMS providers, database admin keys, Supabase `service_role`, Firebase Admin credentials) appear only in server code.
- Variables with public prefixes (`NEXT_PUBLIC_`, `VITE_`, `REACT_APP_`, `EXPO_PUBLIC_`, `PUBLIC_`) are embedded in the browser bundle. Never put secrets in them.
- The browser never calls a paid or private API directly with a secret key.
- Keys meant to be public (Stripe publishable key, Firebase web config, Supabase anon key) are fine, but only safe if your database rules and server checks are correct.

**Test:** Build the app and search the output folder for your key. Open the browser DevTools Network tab and check request headers and bodies.
**Fix:** Move the API call to a server route that checks the user is logged in, then calls the provider.

### 2. Keep `.env` files out of Git and public folders
**Why:** Env files hold secrets. Once committed or served, they are exposed.
**Check:**
- `.gitignore` contains `.env` and `.env.*` (but allows `.env.example`).
- `git ls-files` does not list any real env file. Adding a file to `.gitignore` does not untrack a file that was already committed. Use `git rm --cached .env` for that.
- Env files are not inside `public/`, `static/`, `www/`, or any served folder.
- `.dockerignore` excludes env files. Secrets are not baked into Docker images or build logs.
- The `.git/` folder and backup files (`.env.bak`, `.zip`) are not reachable from the website.

**Test:** On the deployed site, open `/.env` and `/.git/config`. Both must return 404 or 403.
**Fix:** Add ignore rules, untrack the file, commit a `.env.example` with fake values, and set real values in your host's environment settings.

### 3. Scan your Git history for leaked secrets
**Why:** Deleting a secret in a new commit does not remove it from old commits, other branches, tags, or forks.
**Check:**
- Run a secret scanner over the **full history**, not just current files. Good options: `gitleaks` (see its README for the current command) and `trufflehog` (for example `trufflehog git file://. --only-verified`).
- Also check notebooks, config files, CI files, screenshots, logs, and docs.
- Turn on GitHub secret scanning and push protection if available.
- Consider a pre-commit hook so secrets are caught before they are committed.

**Test:** Run the scanner and review every finding, including ones in old commits.
**Fix:** For every real finding, go to item 4.

### 4. Rotate any keys you have accidentally exposed
**Why:** Treat an exposed key as already stolen. Bots scan public repos within minutes.
**Do this, in order:**
1. **Revoke or rotate the key** at the provider first.
2. Put the new key in your deployment environment and local `.env`.
3. Check the provider's usage and billing logs for unexpected activity.
4. Optionally clean history with `git filter-repo` or BFG. This is cleanup only. It does not make the old key safe, because copies and forks may exist.
5. Ask the user before rewriting history, since it affects everyone with a clone.

**Note:** Making a repo private or deleting it does not undo the exposure.

---

## Group 2: Access control

### 5. Check permissions on the server, on every request
**Why:** Hiding a button in the UI is not security. Anyone can call your API directly with `curl` or a script.
**Check:**
- Every protected route, API handler, server action, GraphQL resolver, WebSocket message, and background job checks **who the user is** (authentication) and **what they may do** (authorization).
- Use middleware or a shared helper so checks are not forgotten. Default to deny.
- CORS is not authentication. It only controls browsers.

**Test:** Call each endpoint with no token, with an invalid token, and with a normal user token.
**Fix:** Add a server-side auth check at the top of every handler or in shared middleware.

### 6. Never trust user IDs, roles, or prices sent by the frontend
**Why:** Request data can be edited by the user.
**Check:**
- The current user ID comes from the session or verified token, not from the request body or URL.
- Roles and permissions are loaded from your database, not from the client.
- Prices, discounts, totals, and credits are calculated on the server from your own product data.
- **Mass assignment:** do not pass the whole request body into a database create or update. Allow only specific fields. Fields like `role`, `isAdmin`, `isPaid`, `balance`, and `credits` must never be settable by the client.

**Test:** Edit a request in DevTools or `curl`: change `userId`, add `"role":"admin"`, set `"price":0`.
**Fix:** Read identity from the session, look up prices server-side, and use an allowlist of editable fields.

### 7. Make sure users cannot access each other's data
**Why:** This bug is called IDOR or BOLA. It is one of the most common and damaging web bugs.
**Check:**
- Every read, update, and delete query is scoped to the owner (for example, `WHERE id = ? AND owner_id = current_user`).
- This also applies to list pages, search, exports, nested resources, file download URLs, and API responses that include extra fields.
- Hard-to-guess IDs (UUIDs) help but are **not** a fix. You still need the ownership check.

**Test:** Log in as user A, copy a resource ID, then request it as user B.
**Fix:** Add an ownership check in the data layer, or use database row-level security.

### 8. Lock down your database and private file storage
**Why:** Open databases and buckets are found and scraped automatically.
**Check:**
- The database is not open to the public internet. Use private networking or an IP allowlist, and strong unique credentials.
- The app connects with a least-privilege database user, not the admin account.
- **Supabase or Postgres:** Row Level Security is enabled on every table in an exposed schema, with real policies.
- **Firebase:** rules are not `allow read, write: if true`.
- Storage buckets are private by default. Use short-lived signed URLs for private files.
- MongoDB, Redis, Elasticsearch, and similar services are not exposed with default or no passwords.

**Test:** Try reading data with only the public anon key or no credentials. Try listing the bucket.
**Fix:** Enable RLS or rules, make buckets private, and restrict network access.

### 9. Protect admin actions, even when called directly
**Why:** Admin endpoints are often hidden but not protected.
**Check:**
- Every admin endpoint checks the admin role on the server.
- Hidden or internal routes are protected too: debug routes, seed or migration endpoints, Swagger or API docs, GraphQL introspection, admin panels.
- Sensitive actions (delete user, refund, change role) are logged. Consider asking for re-authentication.
- Use MFA for admin accounts if possible.

**Test:** Call each admin endpoint using a normal user token and with no token.
**Fix:** Add a role check in the endpoint itself, not only in the admin UI.

---

## Group 3: Login and abuse protection

### 10. Test login, logout, and password resets
**Why:** Auth flows have many small mistakes that lead to account takeover.
**Check:**
- Passwords are hashed with a modern algorithm (argon2id, bcrypt, or scrypt), never stored in plain text. Or use a trusted auth provider instead of building your own.
- Session cookies use `HttpOnly`, `Secure`, and `SameSite`. Session IDs change after login.
- **Logout** invalidates the session on the server, and old tokens stop working.
- Changing a password signs out other sessions.
- **Reset links** are random, single-use, short-lived (for example 15 to 60 minutes), and stored hashed. The app responds the same way whether or not the email exists (prevents account enumeration).
- If cookies are used for auth, CSRF protection is in place (SameSite cookies plus CSRF tokens for sensitive actions).
- Email verification works. Login errors do not reveal whether the email exists.

**Test:** Log in, log out, then replay the old token. Use a reset link twice. Let a reset link expire. Try a reset for a non-existent email.
**Fix:** Use your framework's or auth provider's built-in features instead of custom code.

### 11. Rate-limit login, signup, and expensive API calls
**Why:** Without limits, attackers can guess passwords, create spam accounts, and run up costs.
**Check:**
- Limits exist on login (per IP and per account), signup, password reset, OTP or email verification, email or SMS sending, AI endpoints, search, exports, and uploads.
- Return HTTP `429` when limits are hit.
- If you are behind a proxy or CDN, make sure the limiter sees the real client IP.
- Avoid hard account lockouts that attackers can use to lock out real users. Prefer slowing down (backoff) or CAPTCHA.

**Test:** Send 100 quick requests to the endpoint and confirm it starts returning 429.
**Fix:** Use a rate-limit library, or your host's or CDN's built-in rate limiting.

### 12. Add usage caps so one user cannot drain your AI budget
**Why:** AI calls cost money per request. One abusive user or a leaked key can create a huge bill.
**Check:**
- AI features require login.
- Per-user daily or monthly quotas are enforced **on the server**.
- Requests have a maximum input size and a `max_tokens` (output limit).
- A hard spending limit and billing alerts are set in the provider dashboard where available.
- Separate keys for development and production.
- **Prompt injection:** treat AI output and any user-provided or fetched text as untrusted. Do not give the model access to secrets or powerful tools it does not need. Sanitize AI output before showing it as HTML or running it as code or queries.

**Test:** As a normal user, send many large AI requests and confirm the quota and size limits stop you.
**Fix:** Track usage per user in your database and block when over the limit.

---

## Group 4: Input and content safety

### 13. Validate inputs on the server
**Why:** Browser checks can be bypassed completely.
**Check:**
- Validate body, query, path params, and headers with a schema library. Check type, length, range, format, and allowed values.
- Reject unknown fields.
- If the server fetches a URL provided by a user, block internal addresses (localhost, private IP ranges, cloud metadata endpoints) to prevent SSRF.
- Never build file paths from user input (path traversal such as `../../`).

**Test:** Send empty values, huge strings, wrong types, extra fields, and `../` in paths.
**Fix:** Add a server-side schema check at the start of each handler.

### 14. Use safe database queries to prevent injection
**Why:** Joining user text into a query lets attackers run their own queries.
**Check:**
- All queries use parameters, prepared statements, or an ORM or query builder.
- Raw SQL is reviewed carefully. Dynamic column names and `ORDER BY` values are checked against an allowlist.
- **NoSQL:** do not pass raw request objects into queries (for example Mongo operators like `$ne` or `$gt`). Validate that values are plain strings or numbers first.
- The database user has least privilege.

**Test:** Enter `'` and `' OR '1'='1` in inputs and look for errors or changed results.
**Fix:** Replace string-built queries with parameterized ones.

### 15. Make sure user content cannot run scripts (XSS)
**Why:** If someone's comment or profile text runs JavaScript in other users' browsers, they can steal sessions or act as them.
**Check:**
- Use your framework's default escaping.
- Search for risky APIs: `dangerouslySetInnerHTML`, `innerHTML`, `v-html`, `document.write`.
- If you must show rich text or Markdown, sanitize it with a trusted sanitizer library using an allowlist.
- Block `javascript:` links in user-provided URLs.
- SVG and HTML uploads can contain scripts. Do not serve them inline from your main domain.
- Add a Content-Security-Policy header as extra protection.
- AI-generated output shown as HTML must be sanitized too.

**Test:** Save `<img src=x onerror=alert(1)>` as a name or comment and view it as another user.
**Fix:** Stop using raw HTML insertion, or sanitize first.

### 16. Restrict file types and sizes for uploads
**Why:** Uploads can carry malware, scripts, huge files, or overwrite other files.
**Check:**
- Allowlist file extensions **and** check the real file type (file signature or magic bytes). Do not trust the filename or the client's Content-Type.
- Enforce a size limit at the server and proxy, and a limit on the number of uploads.
- Generate random file names. Never use the user's file name for storage paths.
- Store files outside the web root or in a private bucket. Never execute uploaded files.
- Serve downloads with the correct content type, and ideally from a separate domain or with `Content-Disposition: attachment`.
- Consider malware scanning if files are shared between users.
- Be careful with archives (zip bombs) and image processing.

**Test:** Upload a `.html`, `.svg`, `.php`, a renamed `.exe`, and a very large file.
**Fix:** Add allowlists, size limits, and private storage.

---

## Group 5: Payments

### 17. Verify payments on the server before unlocking paid features
**Why:** Anyone can fake a "payment success" page or request.
**Check:**
- Prices and products are set on the server when creating checkout.
- Paid access is granted **only** after a verified webhook or a server-side check with the payment provider, never from the success redirect page or a client message.
- Webhooks verify the provider's **signature** (using the raw request body and your webhook secret).
- Check the event type, amount, currency, and product.
- Handle webhooks idempotently (store the event ID so repeats do not grant access twice).
- Handle refunds, cancellations, and failed payments by removing access.
- Use test keys in development and live keys in production only.

**Test:** Visit the success URL without paying. Replay a webhook. Send a webhook with a bad signature.
**Fix:** Move unlock logic into a signature-verified webhook handler.

---

## Group 6: Production readiness

### 18. Turn off debug mode and keep secrets out of errors and logs
**Why:** Debug pages and logs leak code, paths, tokens, and personal data.
**Check:**
- Production mode is on (for example `NODE_ENV=production`, `DEBUG=False`).
- Users see simple error messages. Stack traces go to logs only, with a request ID.
- Logs never contain passwords, tokens, API keys, `Authorization` headers, or full card or personal data.
- Dev tools are off or protected: API docs, GraphQL playground, admin panels.
- HTTPS everywhere. Add security headers: HSTS, `X-Content-Type-Options`, `Referrer-Policy`, CSP or `frame-ancestors`.
- CORS allows only specific origins. Never use `*` together with credentials.
- Dependencies are updated and scanned (`npm audit`, `pip-audit`, Dependabot).

**Test:** Trigger an error on purpose. Check the response, then check the logs.
**Fix:** Set production mode, add an error handler, and redact sensitive fields in logs.

### 19. Back up your data and test restoring it
**Why:** A backup you have never restored may not work.
**Check:**
- Automatic, regular backups of the database **and** uploaded files.
- Backups are stored separately from the main system, and are encrypted and access-controlled.
- Point-in-time recovery is enabled if your provider supports it.
- A **restore test** has been done in a fresh environment, and the steps are written down.

**Test:** Restore the latest backup to a new database and confirm the app works with it.
**Fix:** Turn on provider backups, schedule restore tests, and document the steps.

### 20. Test with two accounts, then test without logging in
**Why:** This one test finds many of the problems above.
**Procedure:** Create account A and account B, then test as A, as B, and as a logged-out visitor.

| Test | Expected result |
|---|---|
| A opens B's data by changing an ID | Blocked (403 or 404) |
| A edits or deletes B's data | Blocked |
| A calls admin endpoints | Blocked |
| Logged-out visitor opens private pages and APIs | Blocked (401 or redirect) |
| Reuse a token after logout | Rejected |
| A changes `userId`, `role`, or `price` in a request | Ignored or rejected |
| A opens B's private file URL | Blocked or link expired |

**Fix:** Turn these into automated tests so they run on every change.

---

## Stack hints

| Stack | Extra things to check |
|---|---|
| Next.js / Vite / React | Public env prefixes, server actions and API routes have auth checks, no secrets in client components |
| Supabase | RLS on all tables, `service_role` only on server, storage policies |
| Firebase | Firestore and Storage rules, Admin SDK only on server |
| Express / Node | Helmet, rate limiter, input schema validation, CORS allowlist |
| Django | `DEBUG=False`, `ALLOWED_HOSTS`, CSRF middleware, secret key in env |
| Laravel | `APP_DEBUG=false`, `.env` outside public, mass assignment `$fillable` |
| Spring Boot | Actuator endpoints protected, method security, no secrets in `application.properties` |

## Report template

```
# Security Review: <app name>

Summary: <1-2 sentences on overall risk>

## Critical
- [#] <finding> | where: <file or endpoint> | fix: <short fix>

## High / Medium / Low
- ...

## Results (1-20)
| # | Item | Status | Notes |
|---|------|--------|-------|

## Not checked
- <item> | reason / what is needed

## Next steps
1. ...
```
