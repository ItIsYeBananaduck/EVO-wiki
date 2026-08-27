---
title: Smart Context — First Trial Record (2026-08-21)
type: page
tags: [evo, marketing, smart-context, test-record, evidence]
sources: []
origin: wiki-native — first-trial record, superseded in narrative but retained as evidence
updated: 2026-08-24
---

> **Superseded narrative, retained as evidence.** The current account of this work is
> [smart-context-test-record.md](smart-context-test-record.md), written after the
> defects were fixed. This earlier record is kept because it captures the *pre-fix*
> state as directly observed output — baseline `warm.db` measurements, daemon-health
> probes, and the raw `THREAD[...]` hook trace — none of which survived the rewrite.
> Where the two disagree, the newer record is current; this one is the primary source
> for what was measured on 2026-08-21.

# Smart-Context Prototype — Test Record 2026-08-24

First clean end-to-end trial of the smart-context memory prototype, run on the `sara` profile during a live marketing session.

**Headline result:** the `on_memory_write` → curator trigger works. Curation produces output. **Persistence and retrieval do not work** — the curated note is generated and then discarded.

Everything below is observed tool output, not inference. Inferences are labelled.

---

## Environment

| Item | Value |
|---|---|
| Profile | `sara` |
| Session / context_id | `20260821_205826_4cdb22e9` |
| Provider | `smart-context` (`kind: exclusive`) |
| Plugin path | `/opt/data/profiles/sara/plugins/smart-context/provider.py` |
| Daemon | `http://127.0.0.1:7655` |
| Store | `/opt/data/profiles/sara/context/warm.db` |
| Curator model observed | `nvidia/nemotron-3-super-120b-a12b:free` via `openrouter-free` |

---

## Baseline (captured before the write test)

| Metric | Value |
|---|---|
| `warm.db` mtime | `2026-08-22 04:15:50` — two days stale |
| `warm.db` size | 57344 bytes |
| `working_context` | 2 rows |
| `memory_edge` | 0 rows |
| `summary` column | `NULL` on both rows |
| `/tmp/hook-debug.log` | 0 lines |
| Vault segments for session | daemon returns `{"error":"not_found"}` |

The two stored rows were an arbitrary adjacent pair from `2026-08-22T03:12:23` — one short user message and one assistant reply. Nothing from any substantive discussion in the session.

---

## Daemon health — confirmed working

Direct call to the curate endpoint with the profile token, bypassing the plugin:

```
POST /api/v1/curate
Authorization: Bearer <SMART_CONTEXT_TOKEN from profiles/sara/.env>
-> HTTP 200
{"working_note":"...", "archive_fields":{...}, "prompt_tokens":440, "completion_tokens":425}
```

`/health` returns 200 with no auth. **The daemon and curator are not the problem.**

---

## Write test — the trigger fires

A single `memory` tool call (`action=replace`, `target=memory`) produced this trace in `/tmp/hook-debug.log`, which grew 0 → 7 lines:

```
HOOK CALLED: action=replace target=memory content_len=370 pid=2568403
  RESOLVE: kwarg=None task='' session='20260821_205826_4cdb22e9'
           -> context_id='20260821_205826_4cdb22e9'
    THREAD[20260821_205826_] entered
    THREAD[20260821_205826_] client-ok token=yes url=http://127.0.0.1:7655
    THREAD[20260821_205826_] vault-query-ok segments=0
    THREAD[20260821_205826_] curate-start raw_len=370
    THREAD[20260821_205826_] curate-done note=yes len=340
```

Confirmed working:

- Hook is invoked on memory write
- `context_id` resolves from `self._session_id` when kwarg and task_id are empty
- Token resolves (`token=yes`)
- Daemon is reachable
- Curator returns a valid 340-char working note
- No errors on this path

---

## Defects

### D1 — Curated note is discarded (critical)

After the successful curation above, `warm.db` was **byte-identical to baseline**: same mtime `2026-08-22 04:15:50`, same 2 rows, `memory_edge` still 0.

`_refresh_working_memory()` calls `fetch_working_note()`, receives the note, traces `curate-done note=yes len=340`, and returns `None`. **There is no write to the store on that path.** The expensive curation work is performed and thrown away.

This is the load-bearing bug. Everything upstream of it works.

### D2 — Vault never captures the conversation

`vault-query-ok segments=0`, and a direct daemon query returns `not_found` for this session.

Vault capture (`_capture_to_vault`) is only called from `on_pre_compress`, not from `on_memory_write`. Compaction did fire earlier in this session, but the curate calls around that time failed on auth (see D5), so nothing was stored.

Consequence: with no vault material, the curator fell back to `raw_material = memory_content` — curating only the 370 characters just written rather than the conversation. **It is summarizing a summary.**

### D3 — `search_memory` fails on any multi-word query

```
InvalidURL: URL can't contain control characters.
'/api/v1/query/l2?q=Kevin Miller trainer recruitment&limit=20' (found at least ' ')
```

The query string is not URL-encoded. Fix: `urllib.parse.quote` on the `q` parameter. Affects effectively every realistic query.

### D4 — `recall_context` / `search_memory` had no dispatcher

For most of the session both tools were advertised via `get_tool_schemas()` while `handle_tool_call()` was unimplemented. Every call returned:

```
Provider smart-context does not handle tool recall_context
```

`handle_tool_call` was patched in at `2026-08-24 04:41:10 UTC` (file grew 561 → 646 lines mid-session). It required a gateway restart to take effect. **Now resolved** — `recall_context` returns a structured response instead of an error, though it returns 0 results because of D2.

### D5 — Failures are silent

`/tmp/smart-context-plugin-failures.log` had been accumulating unseen:

| Timestamp (UTC) | Stage | Detail |
|---|---|---|
| 2026-08-23 16:22 | `curate_request` | HTTP 401 `invalid_token` |
| 2026-08-23 21:41 | `on_memory_write` | no context_id provided |
| 2026-08-24 00:32 | `curate_request` | HTTP 401 `missing_token` |
| 2026-08-24 03:54 | `curate_request` | timeout after 5.0s |
| 2026-08-24 03:58 | `curate_request_unexpected` | `RemoteDisconnected` |
| 2026-08-24 04:01 | `curate_request` | timeout after 45.0s |

The host wraps provider hooks in `except Exception: pass`, so none of this surfaced to the agent or the operator. The plugin's own `_report_failure` log is the only visibility, and nothing reads it.

### D6 — Memory is externally rewritten mid-session

A `replace` operation failed with `no entry matched` against text that had been present minutes earlier. The returned entry list came back reworded and compressed, and `.curator_backups/blobs/` holds 5 blobs.

Some curator process edits `MEMORY.md` independently of the agent. Consequence: a write the agent believes succeeded can be silently reshaped afterward, and the agent's next edit against remembered text fails.

---

## Agent behaviour under failure

Worth recording because it masks the problem.

With retrieval unavailable, the agent compensated by writing durable artifacts to `EVO/wiki/marketing/` and repeatedly compressing `MEMORY.md` against its 2,200-character cap. Recovery after compaction came from reading files off disk, not from the context system.

That looks like the session going fine. It was the agent routing around a broken subsystem — which is exactly the outcome a prototype trial needs to catch.

**Observed loss from the character cap:** the entry `"EVOtraining is a living training system — not a plan generator. Alice adjusts one variable at a time, evaluates over ~1 month adaptation periods"` was compressed across four successive rewrites to `"living training system"`. The mechanism detail survives only because it is in the wiki. No warning was issued at any point that detail was being dropped.

---

## Implications for Alice `inferred`

Alice must retain a person for decades, which is the same problem at a harder scale. Two transferable findings:

**1. A fixed character budget forces silent lossy rewrites.** Truncation-under-pressure degrades invisibly — there is no signal distinguishing "compressed" from "lost." Alice needs tiering (small hot set, everything else quarryable), not a hard cap on a flat file.

**2. The artifact was more reliable than the summary.** Recovery worked because durable files existed. If Alice's knowledge base is the store of record and the working note is only an index into it, she degrades gracefully. If the working note *is* the memory, she degrades invisibly.

Both are inferred from one session with a broken retrieval path, not validated findings.

---

## Fix order

| Priority | Defect | Scope |
|---|---|---|
| 1 | D1 — persist the curated note | `_refresh_working_memory` must write to `working_context` |
| 2 | D2 — capture vault on memory write, not only pre-compress | curator needs real raw material |
| 3 | D3 — URL-encode query params | one line |
| 4 | D5 — surface failures to the operator | failure log is written but never read |
| 5 | D6 — reconcile agent vs. curator writes to MEMORY.md | ownership/locking question |

D4 is resolved. Re-test after D1 and D2: a memory write should grow `warm.db`, and `recall_context` should then return non-empty results.

---

## Related

- [[EVO Marketing]]
- [[EVO Architecture Bible]]

^[workspace/marketing/]
