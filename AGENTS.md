# Web App Security Checklist (agent instructions)

Use this file when the user asks to build, review, or deploy a web app, or asks for a security check, pre-launch review, or "is this safe to ship?".

**Full details, tests, and fixes for every item:** `skills/web-app-security-checklist/SKILL.md`. Read it before starting a review.

## How to work

1. Identify the stack and which features exist (accounts, admin, uploads, payments, AI calls, database or storage).
2. Inspect the real code and config. Do not guess.
3. Mark each item `PASS`, `FAIL`, `PARTIAL`, `N/A`, or `NOT CHECKED`.
4. Rank fixes: Critical, High, Medium, Low. Give the smallest fix that works.
5. End with a short report (template is in the SKILL.md).

## The 20 checks

1. Keep secret API keys out of your frontend.
2. Keep `.env` files out of Git and public folders.
3. Scan your Git history for leaked secrets.
4. Rotate any keys you have accidentally exposed.
5. Check permissions on the server, on every request.
6. Never trust user IDs, roles, or prices sent by the frontend.
7. Make sure users cannot access each other's data.
8. Lock down your database and private file storage.
9. Protect admin actions, even when called directly.
10. Test login, logout, and password resets.
11. Rate-limit login, signup, and expensive API calls.
12. Add usage caps so one user cannot drain your AI budget.
13. Validate inputs on the server.
14. Use safe database queries to prevent injection.
15. Make sure user content cannot run scripts.
16. Restrict file types and sizes for uploads.
17. Verify payments on the server before unlocking paid features.
18. Turn off debug mode and keep secrets out of errors and logs.
19. Back up your data and test restoring it.
20. Test with two accounts, then test without logging in.

## Rules

- Never print a full secret. Show only the first 4 and last 2 characters, plus file and line.
- Only test apps the user owns or has permission to test. Prefer local or staging.
- Ask before destructive actions (rewriting Git history, deleting files, revoking keys, changing production).
- Do not claim the app is "secure". Report what was checked and what was not.
- Do not invent findings. Use `NOT CHECKED` when you cannot verify.
- When writing new code, follow these checks from the start (server-side auth, validated input, parameterized queries, no secrets in the frontend).
- Keep fixes simple and beginner-friendly. Prefer framework built-ins and well-known libraries.
