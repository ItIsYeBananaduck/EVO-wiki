---
title: "EVOS1-112 — Pre-Migration Testing Baseline Reconstruction"
type: audit
tags: ['lsctech', 'audit', 'source-material', 'canonical', 'evo']
updated: 2026-03-31
---

> **Status: Historical Reference**
> Audit record from EVO cluster review process. Historical investigation or compliance snapshot.

# EVOS1-112 — Pre-Migration Testing Baseline Reconstruction

**Date**: 2026-03-31  
**Branch analyzed**: `work` (testing-line history)  
**Method**: first-parent merged PR chain reconstruction + merge-parent base SHA mapping

---

## Recommendation

### Recommended baseline SHA (pre-migration)

- **`9de4ce1c2d84553bc4ebc665617f4ee1ebed14f6`**
- Why: this is the **first-parent base SHA of PR #152**, which is the first merged PR in the substantive migration run previously referenced as `#152..#193`.

In practical terms, `9de4ce1c` is the last commit on testing immediately before the migration-wave implementation commits began landing.

### Optional alternate baseline SHA

- **`ecf59d87ef8c2d8341f5f4f3db4315e30fceaa91`**
- Why: this is the base SHA before PR #145, which starts the earlier EVOS1 `02.xx` contract-definition chain. If stakeholders classify the contract/interface setup as part of migration, this earlier point is the stricter pre-migration anchor.

---

## Reconstructed merged PR chain (relevant segment)

From first-parent merge history:

- PR #145..#151: EVOS1 `02.xx` contract-definition and package scaffolding sequence
- **PR #152..#180: substantive runtime/domain/package extraction and migration implementation sequence**
- PR #181, #183..#193: audit/governance and follow-up validation sequence over the migrated state

This aligns with prior audit framing that called out the migration wave as `#152..#193`, while also exposing a credible ambiguity boundary at `#145`.

---

## Substantive migration PRs mapped to base SHAs

The table below maps representative substantive migration PR merges to the testing base SHA they were merged onto (`merge_commit^1`):

|   PR | Merge commit | Base SHA (`^1`) | Interpreted role                                  |
| ---: | ------------ | --------------- | ------------------------------------------------- |
| #152 | `c203a2ff`   | `9de4ce1c`      | Runtime ownership lifecycle implementation start  |
| #160 | `d15925dc`   | `94191f9b`      | `packages/delegator` creation                     |
| #165 | `2fb1634f`   | `ebe317b4`      | Training domain package scaffolding               |
| #166 | `ae8fdd96`   | `2fb1634f`      | Hive identity/task execution extraction           |
| #169 | `6c9a89f1`   | `3fae4805`      | Hive lease/work/inference orchestration move      |
| #172 | `99bb680a`   | `c16f46b5`      | Shared task runtime extraction                    |
| #179 | `51109977`   | `37d278ae`      | Shared UI primitives + Alice rendering extraction |
| #180 | `9750357a`   | `51109977`      | Training recomposition using shared packages      |

Observation: these merges form a mostly linear first-parent chain where each PR base is the prior merged state, indicating an intentionally stacked migration series.

---

## Why the selected baseline is correct

1. **Boundary consistency with prior migration-wave framing**: prior audit material explicitly treated the migration wave as beginning at PR #152.
2. **Substantive-change threshold**: PR #152 is the first merge in the sequence that moves from interface/contract setup into concrete runtime ownership implementation.
3. **Deterministic ancestry point**: selecting `merge(#152)^1` yields a clean, reproducible compare anchor (`9de4ce1c`) for downstream diff audits.

---

## Ambiguity and alternate candidate

There is a legitimate interpretation that migration starts earlier at PR #145 (EVOS1 02.01), because that is where cross-package runtime ownership model contracts begin landing.

- If this interpretation is used, baseline shifts to **`ecf59d87`**.
- Recommended operational approach:
  - use **`9de4ce1c`** as the primary baseline for **substantive migration implementation** audit,
  - retain **`ecf59d87`** as a secondary control point for **pre-contract-era** comparison.

---

## Repro commands

```bash
git log --first-parent --merges --reverse --pretty=format:'%H\t%h\t%ad\t%s' --date=short

# map PR merge commits to first-parent base SHA
python - <<'PY'
import subprocess,re
log=subprocess.check_output("git log --first-parent --merges --reverse --pretty=format:'%H\t%s'",shell=True,text=True)
for line in log.splitlines():
    h,s=line.split('\t',1)
    m=re.search(r'Merge pull request #(\d+)',s)
    if not m:
        continue
    pr=int(m.group(1))
    if 145 <= pr <= 193:
        p1=subprocess.check_output(f'git rev-parse {h}^1',shell=True,text=True).strip()
        print(f'PR #{pr}: merge={h[:8]} base={p1[:8]}')
PY
```