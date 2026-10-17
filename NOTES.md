# Notes: Claude Code setup

## What I put in CLAUDE.md

I kept `CLAUDE.md` short. It holds only facts that Claude cannot guess fast from the code, or that it would get wrong:

- **Commands.** How to start the API, run all tests, run one test file, filter tests by name, and lint. I also noted that CI runs lint, then tests, on Node 22, and that both must pass.
- **Conventions.**
  - Use CommonJS, not ES modules. ESLint uses `sourceType: "script"`, so `import` fails lint.
  - Use `node:test`, `node:assert`, and `supertest`, not Jest or Mocha. Never call `app.listen` in tests.
  - Return errors as `{ error: "<message>" }` with 400 or 404.
  - Put a one-line `// METHOD /path — what it does` comment above each route.
  - Routes must not hold data. They use only `db/store.js`.
- **Architecture.** How `server.js`, `routes/`, and `db/store.js` connect, and how to add a resource. I also wrote that the store is in memory and all tests share its state. This prevents tests that assert exact counts or IDs, because these tests break when the run order changes.

## What I left out, and why

- **A file-by-file tour.** Claude can read the tree itself. A list goes out of date and wastes context in each session.
- **Generic advice** ("write clean code", "add tests"). It does not change behavior and only adds noise.
- **Dependency versions and README content.** `package.json` and `README.md` already hold this. Duplicates drift apart.
- **My personal preferences** (writing style, working rules). They are in my global `~/.claude/CLAUDE.md`. The project file is committed and shared, so it must hold only rules for this repo.
- **Secrets and `.env` content.** They must never be in a committed file.

## Permission rules I added

File: `.claude/settings.json` (committed, shared with the team).

- **allow** — `npm test`, `npm run lint`, `node --test`. These are safe, read-only checks that Claude runs often. Allowing them removes repeated prompts.
- **ask** — `git push`. A push is outward-facing and other people see it, so I want to confirm each one.
- **deny**
  - `Read(./.env)` and `Edit(./.env)`.
  - `git push --force` and `git push -f`.

I also fixed `.gitignore`. The lines had leading spaces, so the patterns did not match. Now `.env` and `.claude/settings.local.json` are really ignored.

## What could go wrong without the deny rules

- **`.env` read.** Claude could read API keys and passwords into the conversation. Then the secrets go to the model provider, stay in session transcripts, and can appear in output, commit messages, or logs.
- **`.env` edit.** Claude could overwrite or break local config and real credentials, and I may not see it at once.
- **Force push.** Claude could overwrite remote history on `main` and delete commits from other people. This is hard to undo. The `ask` rule on `git push` is not enough here, because one fast "yes" by mistake is sufficient. A deny rule blocks it always.

## Known limits

- `Read(./.env)` blocks the Read tool only. A shell command like `cat .env` can still read the file. Add Bash deny rules or sandboxing for full protection.
- The rules cover `./.env` only. They do not cover `.env.local` or `.env.production`. Add `Read(./.env.*)` if the project starts to use these files.
- `git push --force-with-lease` and `git push origin +main` are not matched by the force-push rules.
