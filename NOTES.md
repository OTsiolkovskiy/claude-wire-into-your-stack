# NOTES

## Server (MCP)
I connected the `docs` server (`@modelcontextprotocol/server-filesystem`) at project scope, so it lives in the committed `.mcp.json`. It is pointed only at `./docs`, needs no credentials, and gives Claude the API reference (`docs/api.md`) as a source of truth for the endpoints. The permission rule in `.claude/settings.json` (`permissions.allow`) allows only the read-only tools `mcp__docs__read_text_file`, `mcp__docs__list_directory` and `mcp__docs__search_files`, not the whole server, so Claude cannot write, edit or move files through it. I used it on a real task: Claude read `api.md` through `docs` and checked it against `routes/`. All routes were documented, and no permission prompt appeared.

## Skill
The `express-route` skill (`.claude/skills/express-route/SKILL.md`) captures how a route is written in this project: one router file per resource mounted in `server.js`, data access only through `db/store.js`, `400`/`404` validation, `{ "error": "message" }` errors, tests with `node:test` + `supertest` + `store.reset()`, an entry in `docs/api.md`, and a final `npm test` and `npm run lint`. The description starts with "Use when adding, changing or removing an HTTP endpoint/route", gives concrete example requests, and ends with "Not for config, CI or non-route changes", so it fires on endpoint work and nothing else. I checked that it triggers when asked to add an endpoint without naming the skill.

## Command
`/check-conventions [file]` (`.claude/commands/check-conventions.md`) reviews a file passed as `$ARGUMENTS`, or the current `git diff main` when empty, against the conventions in CLAUDE.md. It then runs `npm test` and `npm run lint` and returns a ✅/❌ checklist with `file:line` for each failure and a one-line verdict. It is worth a shortcut because I run the same review before every PR. It is read-only (`allowed-tools` limits it to read/search, `git diff`, `npm test`, `npm run lint`). Run on `routes/users.js`, it worked as intended and reported real gaps: incomplete input validation (e.g. a non-numeric `:id`, empty or `null` `name` on PUT) and missing `400` tests for `POST /users` and `PUT /users/:id`.


