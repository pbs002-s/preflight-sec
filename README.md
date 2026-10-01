# Web App Security Checklist for AI Agents

A 20-point security checklist for web apps, packaged so **any AI coding agent** can use it to build safer apps or review existing ones before launch.

It covers the mistakes that hurt beginners most: leaked API keys, broken access control, missing rate limits, runaway AI bills, unsafe uploads, and fake payments.

> Works with Claude, Codex, Gemini CLI, Cursor, GitHub Copilot, Windsurf, Cline, and anything that reads `AGENTS.md`.

---

## What's inside

```
web-app-security-checklist/
├── README.md                                  <- you are here
├── LICENSE
├── AGENTS.md                                  <- universal instructions (Codex, Amp, Jules, and many others)
├── CLAUDE.md                                  <- Claude Code (imports AGENTS.md)
├── GEMINI.md                                  <- Gemini CLI
├── skills/
│   └── web-app-security-checklist/
│       └── SKILL.md                           <- full detailed checklist (Claude Skill format)
├── .cursor/rules/security-checklist.mdc       <- Cursor
├── .github/copilot-instructions.md            <- GitHub Copilot
├── .windsurf/rules/security-checklist.md      <- Windsurf
└── .clinerules/security-checklist.md          <- Cline
```

`SKILL.md` is the single source of truth. All other files are short and point to it, so you only edit one place.

---

## Quick start

Copy the files for your agent into **your own project**.

| Agent | What to copy | Notes |
|---|---|---|
| **Claude Code** | `skills/web-app-security-checklist/` into `.claude/skills/` (or `~/.claude/skills/` for all projects). Optionally also `CLAUDE.md` and `AGENTS.md`. | Claude loads the skill when your request matches its description. |
| **Claude (web app)** | Zip the `web-app-security-checklist` skill folder and upload it as a custom skill in Claude's skill settings. | The folder name must match the `name` in `SKILL.md`. |
| **Codex, Amp, Jules, and other `AGENTS.md` agents** | `AGENTS.md` plus the `skills/` folder, in your project root. | |
| **Gemini CLI** | `GEMINI.md`, `AGENTS.md`, and the `skills/` folder. | |
| **Cursor** | `.cursor/rules/`, `AGENTS.md`, and the `skills/` folder. | Rule is applied when relevant to the task. |
| **GitHub Copilot** | `.github/copilot-instructions.md`, `AGENTS.md`, and the `skills/` folder. | |
| **Windsurf** | `.windsurf/rules/`, `AGENTS.md`, and the `skills/` folder. | |
| **Cline** | `.clinerules/`, `AGENTS.md`, and the `skills/` folder. | |
| **Any other AI** | Paste the contents of `SKILL.md` into the chat. | Then ask: "Review my app using this checklist." |

Agent tools change over time. If a file location stops working, check your agent's docs for where it reads project instructions.

---

## How to use it

Open your project in your agent and try one of these prompts:

- `Run the web app security checklist on this project and give me a report.`
- `Check my repo for leaked secrets and tell me what to rotate.`
- `Review my auth, admin routes, and database rules using the security checklist.`
- `Before I deploy, do a pre-launch security review. Skip anything that does not apply.`
- `Build a login and payments feature. Follow the security checklist while writing the code.`

The agent will inspect your code, mark each item `PASS`, `FAIL`, `PARTIAL`, `N/A`, or `NOT CHECKED`, and give you a prioritized list of fixes.

---

## The 20 checks

| # | Check | Group |
|---|---|---|
| 1 | Keep secret API keys out of your frontend | Secrets |
| 2 | Keep `.env` files out of Git and public folders | Secrets |
| 3 | Scan your Git history for leaked secrets | Secrets |
| 4 | Rotate any keys you have accidentally exposed | Secrets |
| 5 | Check permissions on the server, on every request | Access control |
| 6 | Never trust user IDs, roles, or prices sent by the frontend | Access control |
| 7 | Make sure users cannot access each other's data | Access control |
| 8 | Lock down your database and private file storage | Access control |
| 9 | Protect admin actions, even when called directly | Access control |
| 10 | Test login, logout, and password resets | Login and abuse |
| 11 | Rate-limit login, signup, and expensive API calls | Login and abuse |
| 12 | Add usage caps so one user cannot drain your AI budget | Login and abuse |
| 13 | Validate inputs on the server | Input safety |
| 14 | Use safe database queries to prevent injection | Input safety |
| 15 | Make sure user content cannot run scripts | Input safety |
| 16 | Restrict file types and sizes for uploads | Input safety |
| 17 | Verify payments on the server before unlocking paid features | Payments |
| 18 | Turn off debug mode and keep secrets out of errors and logs | Production |
| 19 | Back up your data and test restoring it | Production |
| 20 | Test with two accounts, then test without logging in | Production |

Each item in [`SKILL.md`](skills/web-app-security-checklist/SKILL.md) explains **why** it matters, **what to check**, **how to test**, and **how to fix** it. It also includes a severity guide, stack-specific hints (Next.js, Supabase, Firebase, Express, Django, Laravel, Spring Boot), and a report template.

---

## Safety rules built into the agents

The instructions tell agents to:

- never print a full secret (only a masked preview and the file and line),
- only test apps you own or have permission to test,
- ask before destructive actions like rewriting Git history or revoking keys,
- never claim an app is "secure", and instead report what was and was not checked,
- never invent findings.

---

## Contributing

Pull requests are welcome. Good contributions:

- fixes for anything inaccurate or outdated,
- new stack hints,
- clearer explanations for beginners,
- config files for other agents.

When editing the checklist, change `skills/web-app-security-checklist/SKILL.md` first, then update the 20-item list in `AGENTS.md` and this README if item names change.

---

## Disclaimer

This checklist is a practical starting point, not a complete security audit. Passing every item does not guarantee your app is secure. For apps that handle sensitive data or large amounts of money, get a professional security review.

---

## License

[MIT](LICENSE)
