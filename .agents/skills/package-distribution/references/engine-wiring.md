# Engine packaging wiring — operator reference

Source of truth for **templates** and **release automation**. Live tap/bucket are consumers.

## Paths

```text
C:\dev\ledgerful\
  packaging/homebrew/ledgerful.rb
  packaging/scoop/ledgerful.json
  scripts/bump-manifests.ps1      # primary on Windows
  scripts/bump-manifests.sh       # CI / macOS bash 3.2+
  tests/fixtures/package-manifests/v0.1.8/*.sha256
  tests/integration/bump_manifests.rs
  .github/workflows/release.yml   # bump-manifests + winget-release
  docs/package-distribution.md
  docs/installation.md
  Cargo.toml                      # [package.metadata.binstall]
```

## Release assets (canonical names)

| Asset | Layout |
|---|---|
| `ledgerful-x86_64-pc-windows-msvc.zip` | `ledgerful.exe` at **root** |
| `ledgerful-x86_64-unknown-linux-gnu.tar.gz` | nested `ledgerful-*/ledgerful` |
| `ledgerful-x86_64-apple-darwin.tar.gz` | nested |
| `ledgerful-aarch64-apple-darwin.tar.gz` | nested |
| `*.sha256` sidecars | first whitespace-separated field = pin |

URL scheme:

```text
https://github.com/Ryan-AI-Studios/Ledgerful/releases/download/v{VERSION}/{asset}
```

## Bump job behavior

On each engine tag release, after publish (post-0098 — push is **not** optional):

1. Download published `*.sha256` for the tag  
2. Run `bump-manifests`  
3. `scripts/require-secret.sh MANIFEST_PUSH_TOKEN` — **hard-fails** if empty  
4. Clone + commit + push:
   - `Ryan-AI-Studios/homebrew-tap` → **`ledgerful.rb`**
   - `Ryan-AI-Studios/scoop-bucket` → **`ledgerful.json`**
5. `verify-manifests` reads live pins via `gh api` and **fails** if they did not move

**Invariant:** hashes read **only** from sidecars. Never recompute from zip/tar in the bump script.

Release path also has **Gate A** (preflight on tag push) and **Gate B** (scheduled drift check, including npm pin). See [docs/package-distribution.md](https://github.com/Ryan-AI-Studios/Ledgerful/blob/main/docs/package-distribution.md) — do not re-author that runbook here.

### Local fixture test

```powershell
pwsh -File C:\dev\ledgerful\scripts\bump-manifests.ps1 `
  -Version 0.1.8 `
  -ChecksumsDir C:\dev\ledgerful\tests\fixtures\package-manifests\v0.1.8 `
  -PackagingDir C:\dev\ledgerful\packaging `
  -OutDir $env:TEMP\bump-out

# Engine integration (from engine repo):
# cargo nextest run --test integration -E 'test(bump_manifests)'
```

## Secrets

| Secret | Repo | Purpose | Caution |
|---|---|---|---|
| `MANIFEST_PUSH_TOKEN` | engine | Push bumped formula/manifest to tap + bucket | **Required** for green release (not optional). **Minimal required scope:** `contents: write` on `homebrew-tap` **and** `scoop-bucket` only (nothing else). Live token may currently be wider; narrowing is an open owner action in `deferred.md` (0102 docs half only — do not change credentials from this track). |
| `WINGET_TOKEN` | engine | winget-releaser PRs | **Unset until** `Ledgerful.Ledgerful` accepted |
| `GITHUB_TOKEN` | default Actions | Download public release assets | Least privilege |

## cargo-binstall

- Configured in engine `Cargo.toml` `[package.metadata.binstall]`  
- Maps to same release archives (Windows zip override for root layout)  
- No crates.io publish required: `cargo binstall --git https://github.com/Ryan-AI-Studios/Ledgerful`  
- Not maintained in scoop/homebrew repos

## When live repo and template drift

1. Prefer fixing the **template** + next release bump  
2. For urgent seed fixes, patch live repo **and** template in the same track window  
3. CI on live repos should catch pin mismatches vs **published** tags, not vs local builds  

## Do not from packaging slices

- Edit `src/ledger/crypto.rs` or signing basis  
- Add network calls to the engine “for package managers”  
- Publish to crates.io “to make install easier” (explicit non-goal for distribution)  
