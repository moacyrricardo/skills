Start (or confirm) this project's local dev server so other steps can drive it. Reads the project's CLAUDE.md "Dev server" section for the start command, port, and readiness signal — stays generic across projects. Idempotent: if it's already up, it just reports the URL.

---

## 1. Find the recipe

Read `./CLAUDE.md` and look for a **"Dev server"** section (or equivalent) that documents:
- the **start** command and **profile**,
- the **port** (default `8080` if unspecified),
- the **readiness** signal (a log line, or just the port answering),
- optionally the **base URL** and a **dev-login** recipe.

If there is no such section, infer the start command from the project type (e.g. Spring
Boot `mvn spring-boot:run`, Node `npm run dev`, etc.) and **tell the user** you're guessing
— offer to add a "Dev server" section to CLAUDE.md so this is deterministic next time.

## 2. Don't double-start

```bash
PORT="${PORT:-8080}"
if curl -fsS -o /dev/null "http://localhost:$PORT/" 2>/dev/null; then
  echo "Already up on :$PORT"; exit 0
fi
```
If it's already answering, report the base URL and stop — do not launch a second instance.

## 3. Start it in the background and wait for ready

Run the project's start command in the background, logging to a temp file, then poll until
the readiness signal appears or the port answers (cap the wait, ~120s). Example shape
(substitute the project's actual command/log line):
```bash
<start-command> > /tmp/dev-<project>.log 2>&1 &
for i in $(seq 1 60); do
  curl -fsS -o /dev/null "http://localhost:$PORT/" 2>/dev/null && break
  grep -qiE "FAILED TO START|BUILD FAILURE" /tmp/dev-<project>.log && { echo "startup failed"; tail -20 /tmp/dev-<project>.log; exit 1; }
  sleep 2
done
```

## 4. Report

Print the **base URL** (`http://localhost:$PORT`) and the **LAN URL**
(`http://$(hostname -I | awk '{print $1}'):$PORT`) for on-device checks, plus the **kill**
command from CLAUDE.md (or `kill $(lsof -ti :$PORT)`). Leave the server **running** — it's
meant to persist for the rest of the session / for `/spec-workflow:spec-test-live`.

## 5. Concurrency / isolation (opt-in)

By default this is a **singleton**: one shared server on the documented port, reused by every
agent. That's correct for the common case and what `/spec-workflow:spec-test-live` expects. Multiple capture
agents can drive that one server concurrently **as long as none of them mutate shared state**
(e.g. the credential hash-swap — see CLAUDE.md dev-login) — read-only captures are fine.

For genuinely isolated parallel runs (two agents that both write data, or both swap
credentials), a shared port is **not** enough — they'd still race on the same DB. Real
isolation needs *both*:
- **A free app port** — start with the project's "custom port" flag bound to an OS-assigned
  port (e.g. `-Djetty.port=0`-style, or pick a free port and pass it), and **skip the
  "already up?" short-circuit** in step 2 so a second instance actually starts.
- **A per-agent DB/schema** — point that instance at its own database (a throwaway
  `..._evid_<n>` schema or a transient clone), so writes/credential swaps don't collide.

Discovery contract: whichever component starts the isolated instance must hand the chosen
**base URL** to its capture agent via the `BASE_URL` env var (the agent already reads
`$BASE_URL`); don't assume the default port. This track is heavier (multiple JVMs + DB
provisioning) — only reach for it when two simultaneous *stateful* captures are actually
required; otherwise keep the singleton.
