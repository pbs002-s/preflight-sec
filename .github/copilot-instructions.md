# Web App Security Checklist

When reviewing, building, or deploying a web app, or when asked for a security check, follow the 20-point checklist in `AGENTS.md`.
For full details, tests, and fixes, read `skills/web-app-security-checklist/SKILL.md`.

Key rules:
- Never put secret keys in frontend code. Never commit `.env` files.
- Check authentication and authorization on the server for every request.
- Never trust user IDs, roles, or prices from the client. Scope every query to the owner.
- Validate input on the server and use parameterized queries.
- Rate-limit login, signup, and expensive or AI endpoints, and add per-user usage caps.
- Verify payments on the server (signed webhooks) before unlocking paid features.
- Never print full secrets in your output. Do not claim an app is "secure". Report what was and was not checked.
