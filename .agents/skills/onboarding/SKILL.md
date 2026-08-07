---
name: onboarding
description: Load at session start for Ledgerful package-distribution repos (scoop-bucket, homebrew-tap, winget bootstrap). Establishes topology, invariants, and where to work. For track execution, also load implement + package-distribution.
---

# Package-distribution onboarding (cold start)

You are working on **Ledgerful install distribution** — metadata that points package managers at **already-published** release binaries. You are **not** in the product engine unless the task explicitly says so.

If you opened this skill from a blind session, read this file completely before editing anything.

---

## 1. Identity (do not invent names)

| Key | Value |
|---|---|
| Product | Ledgerful |
| Binary | `ledgerful` / `ledgerful.exe` |
| Engine repo | `C:\dev\ledgerful` · https://github.com/Ryan-AI-Studios/Ledgerful |
| Org | `Ryan-AI-Studios` |
| License | PolyForm Noncommercial 1.0.0 + `COMMERCIAL-EXCEPTION.md` (small-entity) |
| Language for public claims | **source-available** — never "open source" / MIT for this product |
| Conductor | `C:\dev\coordinated\conductor\` |
| Coordination contract | `C:\dev\coordinated\coordination.md` |
| Governing track | **0051-PackageManagerDistribution** |

---

## 2. Topology (where files live)

```text
C:\dev\ledgerful\                          # ENGINE — source of truth for templates + release CI
  packaging/homebrew/ledgerful.rb          # formula TEMPLATE (bump rewrites version+sha256)
  packaging/scoop/ledgerful.json           # scoop TEMPLATE
  scripts/bump-manifests.{ps1,sh}          # reads published *.sha256 ONLY
  .github/workflows/release.yml            # bump-manifests + winget-release jobs
  docs/package-distribution.md             # human architecture for this surface
  docs/installation.md

C:\dev\scoop-bucket\                       # THIS REPO (Scoop) — LIVE MANIFEST
  ledgerful.json                           # ROOT (NOT bucket/ledgerful.json)
  README.md, LICENSE, COMMERCIAL-EXCEPTION.md
  .github/workflows/ci.yml                 # pin + Windows zip smoke
  .agents/                                 # you are here

C:\dev\homebrew-tap\                       # Homebrew — LIVE FORMULA
  ledgerful.rb                             # ROOT (NOT Formula/ledgerful.rb)
  README.md, LICENSE, COMMERCIAL-EXCEPTION.md
  .github/workflows/ci.yml                 # pin + Linux archive smoke
  (.agents/ should mirror this repo if present)

C:\dev\coordinated\conductor\
  0051-PackageManagerDistribution\
    spec.md, plan.md, review.md
    winget-bootstrap\                      # FIRST winget package prep (not a git "channel repo")
      README.md
      validate_manifests.py
      manifests/l/Ledgerful/Ledgerful/0.1.8/*.yaml
  conductor.md                             # track registry
  deferred.md                              # lows / residuals
```

Remote URLs:

| Channel | GitHub | Install command (after seed) |
|---|---|---|
| Scoop | https://github.com/Ryan-AI-Studios/scoop-bucket | `scoop bucket add ledgerful https://github.com/Ryan-AI-Studios/scoop-bucket` then `scoop install ledgerful` |
| Homebrew | https://github.com/Ryan-AI-Studios/homebrew-tap | `brew install Ryan-AI-Studios/tap/ledgerful` |
| winget | `microsoft/winget-pkgs` (external) | `winget install Ledgerful.Ledgerful` **only after** package is accepted |

Other install channels (not owned by these repos): one-line `install/install.{ps1,sh}`, `cargo binstall --git https://github.com/Ryan-AI-Studios/Ledgerful`, `@ledgerful/mcp-server` (npm; engine pin `ledgerfulEngineTag`; Gate B asserts published pin; `npm-publish` after release assets — **wired/gated, not yet exercised** as of 2026-07-29: latest was hand-published, `dist.attestations` absent; next tag is the proof).

---

## 3. What is already done (0051 status — re-check conductor)

As of scoop-bucket seed completion (2026-07-20), typical status is:

| Slice | Status | Notes |
|---|---|---|
| Engine packaging + bump + winget-releaser job | **Done** (engine PR) | Templates + automation live |
| homebrew-tap seed | **Done** (PR #1) | Root formula + CI |
| scoop-bucket seed | **Done** (PR #1) | Root manifest + CI + live scoop install |
| winget first package | **Bootstrap prepared; NOT submitted** | Needs **owner approval** before winget-pkgs PR |
| ledgerful-web `/install` PM tabs | May still be open | Separate repo; truth-check discipline |

**Always re-read** `C:\dev\coordinated\conductor\conductor.md` row for 0051 and `plan.md` — do not trust this table over conductor.

---

## 4. Invariants (violating these fails review)

1. **Published checksums are authoritative.** Manifest `hash` / formula `sha256` must match the first field of the release asset’s `*.sha256` sidecar. Never invent hashes; never “fix” a pin by re-hashing and writing that into the manifest unless you are deliberately recovering from a known bad pin **and** the sidecar is updated first in a release.
2. **Bump scripts never recompute hashes from archives** (`scripts/bump-manifests.*` read `*.sha256` only). Preserve that invariant if you touch automation.
3. **Root destinations are load-bearing** (engine `release.yml` `push_one`):
   - scoop-bucket → `ledgerful.json`
   - homebrew-tap → `ledgerful.rb`
   Do **not** move to `bucket/` or `Formula/` without changing engine push paths in the **same** change.
4. **Windows portable zip** has `ledgerful.exe` at **archive root**. Unix tarballs nest `ledgerful-{target}/ledgerful`. Scoop/winget require `.zip`, not `.tar.gz`.
5. **Distribution only** — no changes to engine runtime, Ed25519 signing basis, or network/privacy posture “while you’re here.”
6. **Truth gate** — install commands on READMEs / web must be real. Do not advertise winget install until the package is accepted. PolyForm NC is not MIT.
7. **Git hygiene** — feature branch → PR → CI green → squash-merge. Never force-push `main`. No secrets in git.
8. **Secrets**
   - `MANIFEST_PUSH_TOKEN` — required for a green release after 0098: `scripts/require-secret.sh` hard-fails when empty; `verify-manifests` asserts live tap/bucket pins moved. (Still the secret that authenticates the push.)
   - `WINGET_TOKEN` — **leave unset** until `Ledgerful.Ledgerful` exists in winget-pkgs.

---

## 5. Session start checklist

```text
1. Identify which surface you are in:
     scoop-bucket | homebrew-tap | winget-bootstrap | engine packaging
2. Read:
     .agents/skills/package-distribution/SKILL.md
     .agents/skills/package-distribution/references/<channel>.md
     C:\dev\coordinated\conductor\conductor.md   (0051 row)
     C:\dev\coordinated\conductor\deferred.md    (0051 residuals)
3. If track work: read 0051 spec.md + plan.md + review.md
4. git status / branch; never work directly on main for changes
5. Confirm latest published release tag + pins if editing version/hash:
     gh release view --repo Ryan-AI-Studios/Ledgerful
     gh release download <tag> --repo Ryan-AI-Studios/Ledgerful --pattern "*.sha256" --dir <tmp>
```

Engine-side doctor/ledger commands are **optional** here (these repos are not Rust product code). Prefer package-manager smokes and CI.

---

## 6. Authority order

1. User / run prompt  
2. `C:\dev\coordinated\conductor\conductor.md` + track `spec.md` / `plan.md`  
3. `C:\dev\coordinated\coordination.md` (if any shared claim or install-page contract)  
4. This onboarding + `package-distribution` skill  
5. Engine `docs/package-distribution.md` + `docs/installation.md`  
6. Live files in tap/bucket/winget-bootstrap  
7. External package-manager docs (Scoop / Homebrew / winget schemas)

When template and live manifest disagree after a release, **engine packaging template + published sidecars** win; live repos should be re-bumped, not hand-edited to a third truth.

---

## 7. How work is supposed to run

```text
plan → implement on feature branch → targeted checks / local smoke
  → subagent review → fix → re-review (until clean of above-low)
  → codex (or Claude fallback) cross-model gate
  → PR → wait for CI green (do not poll every few seconds; recheck on notification / long wait)
  → squash-merge
  → update conductor plan/review + coordination changelog + deferred.md as needed
```

Severity: **above-low (medium/high/critical) blocks**. Only **low** may go to `deferred.md` with date/repo/track/why.

---

## 8. Channel cheat-sheet (details in references/)

### Scoop (`C:\dev\scoop-bucket`)

- Manifest: root `ledgerful.json`
- Artifact: `ledgerful-x86_64-pc-windows-msvc.zip`
- Fields: `url`, `hash` (64 hex), `bin: ledgerful.exe`, `checkver`, `autoupdate` → `$url.sha256`
- Smoke:

```powershell
scoop bucket add ledgerful https://github.com/Ryan-AI-Studios/scoop-bucket
scoop install ledgerful
ledgerful --version
```

### Homebrew (`C:\dev\homebrew-tap`)

- Formula: root `ledgerful.rb` (CLI **formula**, not cask)
- Artifacts: three `tar.gz` (aarch64-darwin, x86_64-darwin, x86_64-linux)
- Install resolves binary via buildpath-anchored dual glob:
  `Pathname.glob(buildpath/"ledgerful")` first (brew stages tar top-level
  `ledgerful-{target}/` as buildpath), nested
  `Pathname.glob(buildpath/"ledgerful-*/ledgerful")` fallback
- `license :cannot_represent` is intentional (PolyForm NC + exception)
- Quarantine interim until notarization: `xattr -d com.apple.quarantine "$(which ledgerful)"`
- Smoke (macOS/Linuxbrew): `brew install Ryan-AI-Studios/tap/ledgerful && ledgerful --version`
- Residual: live Apple Silicon brew matrix may still be deferred if host is Windows-only

### winget (`winget-bootstrap` under track 0051)

- Identifier: **`Ledgerful.Ledgerful`** (permanent once accepted)
- Installer: same portable Windows zip as Scoop
- Bootstrap version tracks **latest published tag** (e.g. 0.1.8), not necessarily Cargo.toml pre-release version
- **STOP before submit** without explicit owner approval
- Validate: `python validate_manifests.py` in winget-bootstrap dir
- After Microsoft merge: owner sets engine `WINGET_TOKEN` for later auto-bumps

---

## 9. Common failure modes (seen in real seeds)

| Symptom | Likely cause |
|---|---|
| CI PowerShell parse error near `$version:` | Use `${version}:` in throw strings |
| Scoop install wants 7-Zip | Manifest pointed at `.tar.gz` instead of `.zip` |
| Binary not found after extract | Expected root `ledgerful.exe` but archive nested (or reverse for brew) |
| Empty `MANIFEST_PUSH_TOKEN` / unmoved live pin | Empty secret fails `bump-manifests` hard (`require-secret.sh`); unmoved live pin after publish fails `verify-manifests` |
| winget-release job fails every tag | `WINGET_TOKEN` set before first package exists |
| Install page lies | Advertised channel before live; fails truth-check discipline |

---

## 10. Stop before

- Opening a **public** winget-pkgs PR without owner approval  
- Setting `WINGET_TOKEN` pre-bootstrap  
- Force-push / direct push to `main`  
- Recomputing and writing hashes without published sidecars  
- Moving root formula/manifest paths without engine `push_one` update  
- Editing engine `crypto.rs` / network policy “as packaging cleanup”  
- Claiming “open source” / MIT for Ledgerful in package metadata  

---

## 11. Key paths to open next

```text
.agents/skills/package-distribution/SKILL.md
.agents/skills/package-distribution/references/scoop.md
.agents/skills/package-distribution/references/homebrew.md
.agents/skills/package-distribution/references/winget.md
.agents/skills/package-distribution/references/engine-wiring.md
.agents/skills/implement/SKILL.md
C:\dev\coordinated\conductor\0051-PackageManagerDistribution\spec.md
C:\dev\ledgerful\docs\package-distribution.md
```
