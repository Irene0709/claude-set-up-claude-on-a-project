# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

A small Express REST API (users + health check) backed by an in-memory store.

## Commands

- `npm run dev` — start the API on http://localhost:3000 with `node --watch` (set `PORT` to change it)
- `npm test` — run all tests with the built-in Node test runner
- `node --test tests/users.test.js` — run one test file
- `node --test --test-name-pattern="404"` — run tests whose name matches a pattern
- `npm run lint` — ESLint (`eslint:recommended`)

CI (`.github/workflows/ci.yml`) runs `npm run lint` then `npm test` on Node 22. Both must pass.

## Conventions

- Use CommonJS (`require` / `module.exports`), not ES modules. ESLint is set to `sourceType: "script"`.
- Use the Node built-ins `node:test` and `node:assert` for tests, not Jest or Mocha. Use `supertest` against the exported `app`; never call `app.listen` in tests.
- Return errors as JSON `{ error: "<message>" }` with the matching status code (400 for bad input, 404 for a missing resource).
- Put a one-line comment above each route in the form `// METHOD /path — what it does`.
- Routes must not hold data. Read and write only through functions in `db/store.js`.

## Architecture

- `server.js` creates the app, adds `express.json()`, mounts one router per resource, and exports `app`. It calls `listen` only when run directly (`require.main === module`), so tests can import it.
- `routes/<resource>.js` exports an `express.Router()`. To add a resource, create the router file and mount it in `server.js`.
- `db/store.js` keeps data in module-level variables. Data resets on restart, and all tests in one process share the same state, so do not assert exact counts or IDs.
