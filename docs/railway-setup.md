# Running the dashboard on a server (Railway)

The other deployment option is publishing as an AI link — free, no infrastructure, but the numbers are baked in at each refresh and the shared board only works on Claude. Pick a server instead when several people edit the board at once, or when the dashboard should *act* on ads rather than recommend.

**This repo ships no server code.** What follows is the spec and the runbook: Part 1 is what the code must do, Part 2 is the clicks, Part 3 is what to do when the database won't connect. Build against Part 1 before touching Railway.

## What changes on a server

| | AI link | Server |
| --- | --- | --- |
| Board storage | `data/board.json` in the published page | Postgres |
| Saving | Explicit **Save** button, last Save wins | Saves as you drag; several editors, no conflict |
| Numbers | Baked in at each refresh | Read live |
| Ad actions | Handed to the assistant, or a deep link | Run server-side with real credentials |
| Sheet | Optional one-way mirror | Optional one-way mirror, written from the database |

The `NOTES` array in the HTML becomes the **seed** for the cards table on first run, not the live board.

---

## Part 1 — Code requirements (before deploying)

### 1. `/healthz` must answer "whose fault is it"

```
{ ok, db, dbSource, dbConfigured, dbError, storeFile, storeError }
```

- `db` — `"postgres"` | `"file"` | `"memory"` — where a write actually goes
- `dbConfigured` — true when a database is configured by **either** style in (2)
- `dbSource` — `"DATABASE_URL"` | `"PG* env vars"`
- `dbError` — why the connection failed, when it did

Never collapse "no variable arrived" and "connection failed" into one state. Different fixes, identical symptoms from outside.

### 2. Accept both connection styles

`DATABASE_URL` when set; otherwise connect from `PGHOST` / `PGPORT` / `PGUSER` / `PGPASSWORD` / `PGDATABASE` with **no connection string** (`pg` reads them natively).

Required, not polish: some Railway Postgres services publish only the `PG*` values, and a reference to a variable the target service doesn't publish resolves to an **empty string with no error**.

**Never concatenate a URL from the parts.** It breaks on any password containing `@ : / # ? %`:

```
postgresql://user:sekret@p:ss/w#rd@host:5432/db
  -> {"db":"memory","dbConfigured":true,"dbError":"Invalid URL"}
```

### 3. Negotiate TLS

Railway's public proxy speaks TLS; the private network host (`*.railway.internal`) rejects it outright. Try TLS, fall back to plain **only** on that specific rejection — any other error must fail immediately with its own message, or a wrong password hides behind a TLS error. Honour `sslmode=disable` and `PGSSLMODE=disable`.

### 4. Store precedence: Postgres → file → memory

Never let the host filesystem be the store for shared data — Railway rebuilds it on every deploy, so file writes silently reset. File store is correct locally, wrong in production.

### 5. `Cache-Control: no-cache` on the page if the file is ever the store

### 6. Two passwords

View (read) and edit (write), over a signed httpOnly cookie. Report whether auth is on, so the page never implies protection it doesn't have.

### 7. `railway.json`

```json
{
  "$schema": "https://railway.app/railway.schema.json",
  "build": { "builder": "NIXPACKS" },
  "deploy": {
    "startCommand": "npm start",
    "healthcheckPath": "/healthz",
    "healthcheckTimeout": 30,
    "restartPolicyType": "ON_FAILURE",
    "restartPolicyMaxRetries": 3
  }
}
```

---

## Part 2 — Railway clicks, in order

Browser work in the Railway account. Verify each step with `/healthz`.

1. **New Project → Deploy from GitHub repo.** Pick repo and branch. Confirm `/healthz` answers.
2. **+ New → Database → Add PostgreSQL.**
3. **Click the Postgres service → Variables. Read what it actually publishes.** A `DATABASE_URL`, or only `PGHOST` / `PGPORT` / `PGUSER` / `PGPASSWORD` / `PGDATABASE`? Everyone skips this step; it decides the next one.
4. **App service → Variables** (not the database):
   - Publishes `DATABASE_URL` → add `DATABASE_URL`, type `${{` and pick the service by **name** from the picker. Never hand-type the reference, never use a service UUID.
   - Publishes only `PG*` → add all five the same way. Do **not** assemble a URL.
5. **Redeploy**, then check `/healthz`. Variables are injected at container start, so one added after the last deploy does nothing until you redeploy.
6. **Set `VIEW_PASSWORD` and `EDIT_PASSWORD`.** While both are empty the deploy is PUBLIC.
7. **Settings → Networking → Generate Domain.** Share the URL with the view password.

**Done when** `/healthz` returns `{"ok":true,"db":"postgres","dbSource":"…"}` and the save indicator reads "Changes save automatically".

---

## Part 3 — When it won't connect

Read `dbConfigured` first. It halves the problem.

| `/healthz` says | Meaning | Look at |
|---|---|---|
| `dbConfigured:false` | Variable **absent from the process env** | Platform config, not the database |
| `dbConfigured:true` + `dbError` | Resolved, **connection failed** | `dbError`: host, port, credentials, TLS |
| `db:"postgres"` | Connected | Done |

**`dbConfigured:false`** — in order, cheapest first:

1. **Not redeployed since the variable was added.** Listed ≠ applied. Free to rule out.
2. **Reference points at nothing** — service renamed, a UUID instead of a name, or a `DATABASE_URL` the service never publishes (step 3). Use **Raw Editor** to see literal values.
3. **A manual variable shadowing Railway's own**, including when blank. Delete the manual copy rather than fixing it.
4. **Wrong environment.** Variable in one environment, domain served by another.

**`dbConfigured:true` + `dbError`** — an invalid-connection-string error means the URL was hand-built and the password needs encoding; switch to the `PG*` variables. `ECONNREFUSED` means host/port. Auth failure with credentials you know are right is usually also an unencoded password.

---

## Switching stores does not migrate data

Attaching a database seeds an empty table from the code's seed array. Anything written while the app was on the file or memory store lived on a container disk and is **stranded**.

Export the board (`GET /api/cards`) and bake it into the seed **before** switching. Warn the user first.

## Before sharing the link

- [ ] `/healthz` returns `db:"postgres"` — not `file`, not `memory`
- [ ] The URL prompts for a password before showing any spend figure
- [ ] Add a card, reload, redeploy — board intact through all three
- [ ] The save indicator names where it saves, so "Saved" is never ambiguous
