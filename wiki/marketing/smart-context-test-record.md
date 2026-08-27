---
title: Smart-Context Prototype — Test Record 2026-08-24
type: report
description: End-to-end trial and repair of the smart-context memory prototype on the sara profile. Five defects fixed with evidence; one correction to an earlier wrong diagnosis.
tags: [smart-context, memory, prototype, testing, evo, alice]
updated: 2026-08-24
---

# Smart-Context Prototype — Test Record 2026-08-24

Live trial and repair of the smart-context memory prototype, run on the `sara` profile during a marketing session.

**Result: the architecture works.** Every stage — trigger, curation, structured archive output — was proven functional once five separate infrastructure defects were cleared. None of the failures were design flaws; all were configuration, contract, or ordering bugs.

Everything below is observed tool output. Inferences are labelled.

---

## Environment

| Item | Value |
|---|---|
| Profile | `sara` |
| Session / context_id | `20260821_205826_4cdb22e9` |
| Plugin | `/opt/data/profiles/sara/plugins/smart-context/` |
| Daemon | `/opt/data/smart-context/smart-context-js/`, `http://127.0.0.1:7655` |
| Daemon store | `/opt/data/smart-context/db/smart-context.db` |
| Supervisor | **s6** (`PID 1 = s6-svscan`) — not systemd |
| Curator model | `nvidia/nemotron-3-super-120b-a12b:free` via OpenRouter |

---

## Defects found and fixed

### F1 — Daemon never loaded its config (startup)

`src/main.js` reads its config path from `argv[2]`. Started with no argument it falls back to a hardcoded default it has no permission to create:

```
Fatal startup error: EACCES: permission denied, mkdir '/var/lib/smart-context-daemon'
```

It died before ever reading the `config.json` beside it. **Fix:** always pass the path explicitly. Verified: `smart-context-daemon listening on 127.0.0.1:7655`.

### F2 — Dead OpenRouter API key

The key in `config.json` returned `401 User not found`. The key in `/opt/data/.env` returned `200`. **Fix:** swapped in the working key. Config backed up first.

### F3 — Curator model could not emit JSON

Configured model was `nemotron-3-nano-30b-a3b:free`. It spends its entire token budget on reasoning and never reaches the JSON body:

```
finish_reason: "length",  content: "{\"l"
```

Producing `Missing or invalid l1_note in curator response`. **Fix:** switched to `nemotron-3-super-120b-a12b:free`. Verified HTTP 200 with a complete `l1_note` and 15 populated `l2_fields`.

`inferred`: any curator model must be checked for reasoning-token overrun, not just JSON capability. A model that "supports JSON" can still fail by never reaching it.

### F4 — Startup ordering silently disabled the provider (architectural)

Hermes calls `provider.is_available()` **once** at gateway startup and latches the provider off for the life of the process if it fails. Observed timeline:

| Time | Event |
|---|---|
| 07:40:31 | Gateway started → health check → daemon **down** → provider gated off |
| 07:44:40 | Daemon started (health 200) |
| 07:47 | Memory write → no hook, no trace, nothing persisted |

The daemon was running unsupervised under a shell session, so it died whenever that session ended. Every memory write in the gap was silently lost.

**Fix:** registered the daemon as an s6 service at `/run/service/smart-context-daemon/` (`type: longrun`, run script modeled on `gateway-sara`). Restart proven by killing PID 10203 — s6 respawned 11851 within 6 seconds, health back to 200.

**Two follow-ups remain:**
- `/run` is tmpfs, so the service dir does not survive a container restart. Permanence needs a file under `/etc/s6-overlay/s6-rc.d/`, which requires root or an image change.
- The one-shot health gate should retry rather than latch off permanently. Any momentary daemon blip costs the provider until the next gateway restart.

### F5 — Field-name mismatch discarded every successful curation

The last bug in the chain, and the most deceptive. The daemon returns:

```json
{"l1_note": "...", "l2_fields": {...}}
```

`daemon_client.py` read:

```python
working_note=result.get("working_note", "")
archive_fields=result.get("archive_fields", {})
```

Both `.get()` calls silently defaulted to empty, so the caller discarded a perfectly good note as *"daemon returned an empty working_note."* The daemon's audit log recorded `curate granted=True` at the same instant the plugin logged failure — the two halves disagreed because they used different vocabularies for the same payload.

**Fix:** accept either name, preferring the daemon's current contract:

```python
working_note=(result.get("l1_note") or result.get("working_note") or "")
archive_fields=(result.get("l2_fields") or result.get("archive_fields") or {})
```

Verified against the live daemon: `working_note len: 84`, `archive_fields: 15 keys`, `RESULT: PASS`.

### F6 — Client never sent `segment_id`, so no store could succeed

With F5 fixed, curation succeeded (`curate-done note=yes len=355`) and the store then failed:

```
store-FAIL-unavailable HTTP 400: {"error":"missing_required_fields"}
```

The server's `createSegment()` requires **all three** of `context_id`, `segment_id`, and `content`, and does **not** generate an id server-side:

```js
if (!body || !body.context_id || !body.segment_id || !body.content) {
  return sendJSON(res, 400, { error: 'missing_required_fields' });
}
```

`store_vault()` sent `context_id` and `content` but no `segment_id`, assuming the daemon would mint one.

**Fix:** mint the id client-side (`seg-<uuid4>`), plus the missing `uuid` import.

---

## VERIFIED WORKING — full path, 2026-08-24 12:55

A single memory write, no test harness, no manual steps:

```
HOOK CALLED: action=replace target=memory content_len=342 pid=41334
  RESOLVE: kwarg=None task='' session='20260821_205826_4cdb22e9'
           -> context_id='20260821_205826_4cdb22e9'
    THREAD entered
    THREAD client-ok token=yes url=http://127.0.0.1:7655
    THREAD vault-query-ok segments=0
    THREAD curate-start raw_len=342
    THREAD curate-done note=yes len=398
    THREAD store-ok persisted len=398
```

Database, against a baseline of 1 row:

| Metric | Before | After |
|---|---|---|
| `segments` | 1 | **2** |
| New row | — | `seg-9a487b6125f243b4a0...`, agent `sara`, 398 chars, `12:55:17` |
| New failures | — | none |

**The architecture is validated.** Trigger → curation → durable persistence, driven by nothing but a memory write.

---

## Verdict

Six defects, **all infrastructure — none architectural.**

| # | Defect | Root cause |
|---|---|---|
| F1 | Daemon wouldn't start | Config path from `argv[2]`; bare start fell back to an unwritable `/var/lib/` default |
| F2 | Curator 401 | Dead OpenRouter key in `config.json` |
| F3 | Malformed curator output | `nano-30b` exhausted its token budget on reasoning before emitting JSON |
| F4 | Provider silently disabled | One-shot `is_available()` health gate + unsupervised daemon |
| F5 | Every good curation discarded | Field-name mismatch between daemon and client |
| F6 | No store ever succeeded | Client omitted the required `segment_id` |

The summarize-and-quarry design did what it was built to do as soon as the plumbing was correct. **Every one of these six hid behind the host's `except Exception: pass`** and a failure log nothing reads — which is the single most important finding in this record.

## Superseded trace (kept for the record)

Hook trace after F4 was fixed but before F5 and F6 — shows curation reached but the note discarded:

```
HOOK CALLED: action=replace target=memory content_len=342 pid=17040
  RESOLVE: session='20260821_205826_4cdb22e9' -> context_id resolved
    THREAD entered
    THREAD client-ok token=yes url=http://127.0.0.1:7655
    THREAD vault-query-ok segments=0
    THREAD curate-start raw_len=342
    THREAD curate-done note=NONE          <- F5, now fixed
```

And a complete curation from the daemon:

```json
"l1_note": "Phil confirmed that no video will be available until Beta, so Alice
             adaptation demos must remain honest. Trainer recruitment is
             identified as load-bearing, with Kevin Miller on the critical path."
"l2_fields": { "result_kind": "decision", "decisions": "...", "open_loops": "...",
               "important_entities": "Phil, Alice, Kevin Miller", ... }
```

That is the design doing exactly what it was built to do: read a memory write, produce a compact working note plus a structured archive record.

---

## Correction to an earlier diagnosis

An earlier version of this record claimed a **sha1/sha256 token hash mismatch**. That was wrong on both counts and is retracted.

The daemon ignores `token_store_path` entirely — `config.js` marks it *"reserved for a future operational split; ignored by Phase 1"* — and reads a `tokens` table **inside `smart-context.db`**. The separate `tokens.db` file is vestigial. Acting on the wrong diagnosis, `tokens.db` was edited; it was restored from backup and the token was then minted correctly through the daemon's own `sha256` + `DatabaseSync` path.

Lesson worth keeping: verify which file a process actually holds open (`/proc/<pid>/fd`) before editing a store.

---

## Remaining defects (not fixed)

### R1 — Endpoint contract mismatch

The Python client calls routes the server does not serve:

| Client calls | Server serves |
|---|---|
| `/api/v1/segments` | ✓ exists |
| `/api/v1/archive/promote` | ✗ missing |
| `/api/v1/vault/expire` | ✗ missing |

Two of six client endpoints would 404. `store_vault` happens to use `/api/v1/segments`, which exists, so persistence is unaffected — but archive promotion and vault expiry are broken. The naming is also inconsistent: the plugin says "vault," the daemon says "segments." Same concept, two vocabularies, which is how F5 and R1 both crept in.

### R2 — `hermes gateway restart` assumes systemd

The gateway logs `so systemd Restart=on-failure can revive the gateway`, but PID 1 is `s6-svscan`. Every restart attempt through the CLI path had no effect in this deployment. The working command is:

```bash
/command/s6-svc -r /run/service/gateway-sara
```

### R3 — Vault never captures raw conversation

`vault-query-ok segments=0` on every run. Vault capture only happens in `on_pre_compress`, not `on_memory_write`, so the curator falls back to `raw_material = memory_content` — summarizing the agent's own summary rather than the conversation. Needs a decision about whether memory writes should also seed the vault.

### R4 — Failures are invisible by default

The host wraps provider hooks in `except Exception: pass`. Every defect above was diagnosable **only** via the plugin's private `/tmp/smart-context-plugin-failures.log`, which nothing reads. Six failures had accumulated unseen across two days.

---

## Agent behaviour under failure

Worth recording because it masks the problem.

With retrieval unavailable, the agent compensated by writing durable artifacts to `EVO/wiki/marketing/` and repeatedly compressing `MEMORY.md` against its 2,200-character cap. Recovery after each compaction came from reading files off disk, not from the context system. The session looked healthy while the subsystem under test was entirely dead.

**Observed loss from the character cap:** the entry *"EVOtraining is a living training system — not a plan generator. Alice adjusts one variable at a time, evaluates over ~1 month adaptation periods"* was compressed across four successive rewrites down to *"living training system."* The mechanism detail survives only because it is in the wiki. **No warning was issued at any point that detail was being dropped.**

---

## Implications for Alice `inferred`

Alice must retain a person for decades — the same problem at a harder scale.

**1. A fixed character budget forces silent lossy rewrites.** Truncation-under-pressure degrades invisibly; nothing distinguishes "compressed" from "lost." Alice needs tiering — a small hot set plus a quarryable archive — not a hard cap on a flat file.

**2. The artifact was more reliable than the summary.** Recovery worked because durable files existed. If Alice's knowledge base is the store of record and the working note is only an index into it, she degrades gracefully. If the working note *is* the memory, she degrades invisibly.

**3. Silent failure is the real enemy.** Five defects hid behind `except Exception: pass` and a log nobody reads. A memory system that fails quietly is worse than one that fails loudly, because the agent keeps performing and nobody investigates. Alice needs memory failures surfaced to the user, not swallowed.

---

## Next verification step

The field-name fix (F5) was applied to `daemon_client.py` after the 07:59:39 gateway start, so the running process still holds the old module. **After the next gateway restart**, a memory write should produce:

- `curate-done note=yes len=N` (not `note=NONE`)
- `store-ok persisted len=N`
- `segments` count > 0
- `recall_context` returning non-empty results

That is the outstanding test.

---

## Outstanding work

Nothing blocks the core path — it is verified working. These remain:

| Item | Why it matters |
|---|---|
| **s6 service dir is on tmpfs** | `/run/service/smart-context-daemon/` does not survive a container restart. Permanence needs a file under `/etc/s6-overlay/s6-rc.d/` (root or image change). |
| **Health gate should retry** | The one-shot `is_available()` check still latches the provider off permanently on any momentary daemon outage. |
| **R1 endpoint mismatch** | `/api/v1/archive/promote` and `/api/v1/vault/expire` are called by the client but not served. Archive promotion and vault expiry are still broken. |
| **R3 vault never sees raw conversation** | `vault-query-ok segments=0` every run. Curation summarises the agent's own memory text, not the conversation. Needs a decision on whether memory writes should also seed the vault. |
| **R4 silent failure** | The host's `except Exception: pass` is what allowed six defects to hide for two days. Highest-value structural fix. |
| **Retrieval returns empty** | `recall_context` / `search_memory` now work correctly but return `count: 0` — a consequence of R3, not a bug in the handlers. |

---

## Files changed and backups

| File | Change | Backup |
|---|---|---|
| `smart-context-js/config.json` | F2 key swap, F3 model swap | `config.json.bak-*` (in project dir) |
| `plugins/smart-context/daemon_client.py` | F5 field names, F6 `segment_id` + `uuid` import | `/opt/data/smart-context/daemon_client.py.bak-1787558529` |
| `db/smart-context.db` | token minted via daemon's own sha256 path | `db/smart-context.db.bak-1787556907` |
| `db/tokens.db` | wrongly edited, then **restored** | `db/tokens.db.bak-1787556820` |
| `/run/service/smart-context-daemon/` | new s6 service (tmpfs) | source at `/opt/data/smart-context/s6/run` |

`plugins/smart-context/provider.py` was **not** modified by this session — the persistence block at lines 465–483 was added by another party at 05:44 before this work began.

---

## Related

- [[EVO Marketing]]
- [[EVO Architecture Bible]]

^[workspace/marketing/]
