# CLAUDE.md

Small Express 4 REST API (users + health check) backed by an in-memory store — the starter project for the Kodree Claude Code course.

## Commands

- `npm run dev` — start the API with auto-reload on http://localhost:3000 (`PORT` overrides)
- `npm test` — run all tests (Node's built-in test runner)
- `node --test --test-name-pattern="404" tests/users.test.js` — run a single test by name
- `npm run lint` — ESLint 8, `eslint:recommended` (`.eslintrc.json`)

CI (`.github/workflows/ci.yml`) runs `npm run lint` then `npm test` on Node 22 for every push and PR — run both before committing.

## Architecture

- `server.js` builds the Express app, mounts one router per resource (`app.use("/users", ...)`) and exports `app`. It only calls `listen()` when run directly (`require.main === module`), so tests can import `app` without opening a port — keep that guard.
- `routes/<resource>.js` — one `express.Router()` per resource; handlers validate input and shape the HTTP response.
- `db/store.js` — the only data layer: plain functions over a module-level array. Data resets on restart and is shared by everything in the same process, including all tests in a file.

## Conventions

- Don't make any assumptions. All data must be based on facts and measurements.
- If anything is unclear, please ask.
- At the end of the session, make a list of any tasks or assignments that were not finished.
- CommonJS only (`require` / `module.exports`), not ES modules — ESLint uses `sourceType: "script"`.
- Routes never touch data directly: add a function to `db/store.js` and call it from the route.
- New resource = new file in `routes/` + one `app.use()` line in `server.js`.
- Errors are JSON `{ error: "<message>" }` with a matching status: 400 for missing/invalid input, 404 for unknown ids; successful creates return 201.
- Route `:id` params are strings — convert with `Number()` before passing them to the store (it compares with `===`).
- Tests use `node:test` + `node:assert` + `supertest` against the exported `app` — not Jest or Mocha. Don't assume user counts or ids beyond the seed data; the store is shared across tests.
- Target Node 22 (the CI version): don't use newer Node APIs even if the local Node is newer.
