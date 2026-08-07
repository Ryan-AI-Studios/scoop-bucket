---
name: implement
description: Use when implementing a conductor track slice on package-distribution repos (scoop-bucket, homebrew-tap, winget-bootstrap, or engine packaging/). Load with onboarding + package-distribution.
---

# Implement — package-distribution tracks

Adapted from the Ledgerful implement skill for **manifest/tap/bucket/winget** work. Engine product gates (clippy/nextest/ledger) apply **only** when you edit the engine repo.

## Identity

```text
load_with: onboarding + package-distribution
source_of_truth:
  C:\dev\coordinated\conductor\conductor.md
  C:\dev\coordinated\conductor\<track>/spec.md
  C:\dev\coordinated\conductor\<track>/plan.md
do_not:
  - clear gate with open critical/high/medium findings
  - push to main without PR + green CI
  - recompute manifest hashes from local archives
  - open public winget-pkgs PRs without owner approval
```

## Loop

1. **Plan** — read conductor + track; name files; name channel (scoop/homebrew/winget/engine)  
2. **Implement** on `feature/<track-or-slice>`  
3. **Targeted checks** — channel smoke + schema/CI locally when possible  
4. **Review workflow** — subagent implement → review → fix → re-review (≤3 rounds)  
5. **Codex gate** — cross-model; above-low blocks; lows fix or `deferred.md`  
6. **PR** → wait for CI green (long wait / completion notify; do not busy-poll)  
7. **Squash-merge** only when CI green  
8. **Finalize** — plan/review/conductor/coordination/deferred updates  

## Plan before edit

- Which repo path(s)?  
- Does engine template need the same change?  
- Does `release.yml` push path still match?  
- What published tag/pins are authoritative?  
- What smoke proves DoD?  

## Targeted checks (by surface)

| Surface | Checks |
|---|---|
| scoop-bucket | JSON valid; pin vs `gh release download` sidecar; local `scoop install` if possible |
| homebrew-tap | Formula parses; pins vs sidecars; buildpath dual-glob install (`Pathname.glob`); brew install if host allows |
| winget-bootstrap | `python validate_manifests.py`; optional winget validate |
| engine packaging | `bump-manifests` fixture test; integration filter if present; normal engine gates if Rust/CI touched |

## Review workflow

```text
file: C:\dev\coordinated\conductor\<track>/review.md

internal:
  implement subagent → review subagent → address → re-review
  until clean of above-low (or max 3 rounds then escalate)

codex_gate:
  only after internal clean pass
  codex exec -s read-only with --add-dir coordinated (+ engine if needed)
  Claude fallback if codex unavailable (see codex-review skill)
  ANY finding above LOW blocks

low findings:
  fix if easy; else APPEND C:\dev\coordinated\conductor\deferred.md
```

### Review for (packaging)

- pin correctness  
- path contracts (root formula/manifest)  
- archive layout (zip root vs tar nest)  
- autoupdate/checkver / brew install()  
- CI soundness + SHA-pinned actions  
- README truth  
- secrets / outward PR safety  
- placeholders  

## Severity

Same as engine: critical/high/medium **block**; low only is deferrable.

## Full gate (packaging repos)

Not cargo. Instead:

1. Repo CI green on the PR  
2. Manual smoke recorded with exact commands/output  
3. Cross-model gate clean of above-low  
4. Conductor + deferred + coordination updated when the slice completes  

## Finalize checklist

- [ ] No open above-low findings  
- [ ] Lows fixed or in `deferred.md`  
- [ ] PR squash-merged (or explicit stop-before for winget submit)  
- [ ] `plan.md` phase marked done  
- [ ] `review.md` slice section written  
- [ ] `conductor.md` status line updated  
- [ ] `coordination.md` §11 entry if user-facing/distribution status changed  
- [ ] Residual ops (tokens, branch protection, AS brew) logged if unsolved  

## Stop before

- force-push  
- winget public submit without approval  
- setting `WINGET_TOKEN` pre-bootstrap  
- rewriting hashes without published sidecars  
- moving root paths without engine push update  
