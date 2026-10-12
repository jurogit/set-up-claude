# Notes — Claude Code setup

## What's in `CLAUDE.md`, and what I left out

I started from `/init` and kept only what Claude can't cheaply work out by itself. The file has a one-line description, the commands (`dev`, `test`, `lint`, plus running a single test by name), how CI checks the code, a three-line architecture overview (`server.js` → `routes/` → `db/store.js`) and the project's rules. Every command was run before it went in: the four sample tests pass, lint is clean, and `--test-name-pattern` runs only the matching test.

Apart from three general working rules of my own at the top (no assumptions, only verified facts; ask if anything is unclear; list unfinished tasks at the end of the session), the rules come from patterns in the code, not from generic advice. They cover CommonJS rather than ES modules, data access only through the store, the `{ error }` JSON format with 400/404/201, converting `:id` with `Number()`, and testing with `node:test` + supertest. They also say to target Node 22, which matters because CI runs Node 22 while my machine has Node 26. Each rule is a direct instruction, often in the form "do X, not Y", so Claude can follow it without asking. I added the three working rules because I consider them very important.

I deliberately left out:

- the endpoint and file listings, and the dependency versions — Claude can read these from the code and `package.json`, and they would go stale;
- the README's setup steps and course tasks — they are one-off and don't help with daily work;
- anything from `.env`, or any secret;
- the permission rules themselves — `CLAUDE.md` only guides Claude, while `settings.json` actually enforces the rules.

## Permission rules

- **allow** `npm test *` and `npm run lint *`: safe commands I run constantly, and the two checks `CLAUDE.md` asks for before every commit. Without these rules, each check would stop for a prompt.
- **ask** `git push *`: a push leaves my machine, so I want to confirm each one.
- **deny** `Read(./.env)`: keeps real secrets out of Claude's context. Without it, Claude could read `.env` with the Read tool or `cat` while debugging config. The values would then be sent to the model and could end up in the transcript, in code or in a commit message.
- **deny** force push (`--force`, `-f`, `--force-with-lease`, both before and after the remote and branch name): without it, a single push could overwrite the remote history and destroy other people's commits. The original single rule `git push --force:*` only matched when `--force` came directly after `push`, so `git push -f` or `git push origin main --force` fell back to the ask prompt. Deny rules are checked before ask rules, so these now block even though they also match `git push *`.

Known limits: Bash rules match the command text, so they are not a security boundary. `git push origin +main` (still caught by the ask rule) and `git -C . push --force` are not denied. The `.env` rule doesn't stop a script that opens the file by itself. A PreToolUse hook or the sandbox would close these gaps.

I also fixed `.gitignore`. Its lines from `.env` downward started with spaces, which git treats as part of the pattern. As a result, `.env` was **not** actually ignored, and `.claude/settings.local.json` was only ignored by my global gitignore. The deny rule keeps Claude from reading `.env`, and the fixed `.gitignore` stops it from being committed.

## Verification

### `/memory`

```text
Memory

   ❯ Auto-memory  true

     Project instructions   Checked in at ./CLAUDE.md
     Open auto-memory folder
```

Two entries are left out here: the user-level `~/.claude/CLAUDE.md` and the parent course repo's `CLAUDE.md`. They don't come from this project.

### `/permissions`

```text
Allow
  1.  Add a new rule…
  2.  Bash(npm run lint *)
  3.  Bash(npm test *)

Ask
  1. Add a new rule…
  2. Bash(git push *)

Deny
  1. Add a new rule…
  2. Bash(git push --force*)
  3. Bash(git push -f*)
  4. Bash(git push * --force*)
  5. Bash(git push * -f*)
  6. Read(./.env)
```

Items 4–27 of the Allow list are left out here: 24 `mcp__computer-use__*` rules that don't come from this project. The rest matches `.claude/settings.json`: 2 allow, 1 ask, 5 deny.

### "How do I run the tests here?"

Claude answered in Hungarian, as my global `~/.claude/CLAUDE.md` asks. This is the English translation it gave on request:

> Tests run on Node's built-in test runner (`node:test`), not Jest or Mocha.
>
> **Run all tests:**
>
> ```bash
> npm test
> ```
>
> This runs `node --test`, which finds the `*.test.js` files on its own. Right now the only one is `tests/users.test.js`. If `node_modules` doesn't exist yet, run `npm install` first, because the tests need `supertest`.
>
> **Run a single test by name:**
>
> ```bash
> node --test --test-name-pattern="404" tests/users.test.js
> ```
>
> `--test-name-pattern` is a regex matched against the test name. Replace `"404"` with part of the name of the test you want to run.
>
> **Before committing**, run the linter too, because CI (`.github/workflows/ci.yml`) runs both on Node 22:
>
> ```bash
> npm run lint && npm test
> ```

### `git push`

When Claude ran `git push -u origin set-up-claude`, the `Bash(git push *)` ask rule showed me a permission prompt before the push ran.
