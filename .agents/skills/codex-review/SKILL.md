---
name: codex-review
description: Cross-model completion review for package-distribution track slices. Prefer codex exec read-only; Claude fallback when codex unavailable. Load after internal review is clean.
---

# Cross-model review — package distribution

Full ledgerful skill lives at `C:\dev\ledgerful\.agents\skills\codex-review\SKILL.md` (use that for engine tracks). This file is the **packaging-surface** subset.

## When

After internal implement→review loops are clean of above-low findings on:

- scoop-bucket  
- homebrew-tap  
- winget-bootstrap  
- engine packaging/bump slice  

## Rules

- **READ-ONLY** — reviewer never edits code, git, or deferred.md  
- Orchestrator applies fixes and re-invokes until PASS / PASS WITH DEFERRED P3  
- Above-low (P0–P2) **blocks**; only qualifying P3 may be deferred  

## Handoff template

```text
TRACK: 0051-PackageManagerDistribution (or full track dir)
REPOS: C:\dev\scoop-bucket  (add homebrew-tap / ledgerful / winget-bootstrap as needed)
SCOPE: branch + commits / PR
IMPLEMENTED: brief
KNOWN GATES: CI run URLs, scoop/brew/winget commands + outputs
FOCUS: pins, root paths, layout, CI, README truth, secrets, outward-submit safety
```

## Codex invocation (primary)

```powershell
$TrackDir = "C:\dev\coordinated\conductor\0051-PackageManagerDistribution"
$PrimaryRepo = "C:\dev\scoop-bucket"   # or homebrew-tap / ledgerful
$Prompt = @'
You are the independent completion reviewer for Ledgerful packaging track slice.
READ-ONLY. Audit DoD against implementation. P0-P3. Verdict PASS | PASS WITH DEFERRED P3 | FAIL.
'@

codex exec -C $PrimaryRepo -s read-only `
  -m gpt-5.6-luna -c 'model_reasoning_effort="high"' `
  --add-dir "C:\dev\coordinated" --add-dir "C:\dev\ledgerful" --ephemeral `
  -o "$TrackDir\review.codex.<slice>.md" $Prompt
```

**Do not** pass `-a never` (invalid on current codex CLI). Prefer `-s read-only`.

## Claude fallback

If codex is rate-limited/unavailable, use `claude -p` with read-only tools (Read/Glob/Grep + read-only git). Write `review.claude.<slice>.md`. Still require a **fresh** clean final gate after last fix (do not reuse a stale PASS from before CI/fix commits).

## Packaging audit checklist

1. Every assigned DoD item met or honestly residual (external host / owner approval)  
2. No placeholders / wrong paths / invented hashes  
3. Engine template ↔ live manifest identity when claimed  
4. CI proves pin + layout + version when present  
5. README install commands real for this channel’s status  
6. No secrets; Actions SHA-pinned; least privilege  
7. winget: stop-before-submit honored if not approved  

## Output path

Raw: `C:\dev\coordinated\conductor\<track>\review.<reviewer>.<slice>.md`  
Orchestrator maintains canonical `review.md`.
