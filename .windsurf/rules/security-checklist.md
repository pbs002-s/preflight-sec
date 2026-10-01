---
trigger: model_decision
description: 20-point web app security checklist. Use when building, reviewing, or deploying a web app, or when asked for a security check or pre-launch review.
---

Follow the 20-point checklist in `AGENTS.md`.
For full details, tests, and fixes, read `skills/web-app-security-checklist/SKILL.md`.

Key rules:
- No secret keys in frontend code. Never commit `.env` files.
- Check auth and permissions on the server for every request.
- Never trust user IDs, roles, or prices from the client.
- Validate input on the server and use parameterized queries.
- Rate-limit sensitive and AI endpoints. Add per-user usage caps.
- Verify payments on the server before unlocking paid features.
- Never print full secrets. Do not claim an app is "secure".
