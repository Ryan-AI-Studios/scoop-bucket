---
name: package-distribution
description: Use for Scoop bucket, Homebrew tap, winget bootstrap, or engine packaging/bump work for Ledgerful. Load with onboarding. Channel details in references/.
---

# Ledgerful package distribution

## Mission

Ship and maintain **one-command installs** (`scoop` / `brew` / `winget` / `cargo binstall` / npm `@ledgerful/mcp-server`) that download **checksum-pinned** release binaries. Metadata only — binaries are built by engine release CI. npm is **engine-owned** (not these scoop/homebrew repos); listed so distribution agents know the surface exists.

## Load order

1. `../onboarding/SKILL.md`
2. This skill
3. `references/<scoop|homebrew|winget|engine-wiring>.md` for the active channel
4. Track 0051 `spec.md` / `plan.md` when implementing governed work

## Ownership map

| Concern | Owner |
|---|---|
| Release binaries + `*.sha256` | Engine release pipeline |
| Formula/manifest **templates** | Engine `packaging/` |
| Live formula/manifest in GitHub | homebrew-tap / scoop-bucket |
| Version+hash rewrite on tag | Engine `bump-manifests` → required push via `MANIFEST_PUSH_TOKEN` (`require-secret.sh` + `verify-manifests`) |
| First winget package | Track `winget-bootstrap/` then owner PR to winget-pkgs |
| Subsequent winget bumps | Engine `winget-releaser` + `WINGET_TOKEN` (after first package) |
| `/install` web copy | `ledgerful-web` (truth-check; only show live channels) |

## Golden rules

1. Pin to **published** release assets only.  
2. Hash = first field of `asset.sha256` sidecar.  
3. Keep **root** `ledgerful.rb` / `ledgerful.json`.  
4. Windows = portable **zip**, binary at zip root.  
5. Unix = nested `ledgerful-*/ledgerful` in tar.gz.  
6. README install commands must work **today** or be labeled not-yet-live.  
7. Public license language: PolyForm Noncommercial + commercial exception; source-available.

## When editing a live repo (scoop / homebrew)

```text
1. Branch from main: feature/<track-or-fix>
2. Prefer updating engine packaging/ template first if the change is the long-term source of truth,
   then sync live repo to match (or document one-shot live-only fix).
3. Keep CI guards that:
   - forbid wrong path layout (bucket/ or Formula/)
   - parse version + pins
   - download published *.sha256 and compare
   - download archive, verify hash, extract, run --version when feasible
4. Local smoke with real package manager when host allows
5. PR + CI green + squash-merge
6. Update coordinated conductor/plan/review/deferred + coordination.md changelog
```

## When editing engine packaging

Touch only:

- `packaging/homebrew/ledgerful.rb`
- `packaging/scoop/ledgerful.json`
- `scripts/bump-manifests.*`
- `tests/fixtures/package-manifests/**`
- `tests/integration/bump_manifests.rs` (and related)
- `release.yml` bump/winget jobs
- `docs/package-distribution.md` / `docs/installation.md`

Run engine gates appropriate to the change (`bump_manifests` tests, fmt/clippy if Rust touched). **Do not** broaden into unrelated engine work.

## Verification matrix

| Channel | Minimum proof |
|---|---|
| Scoop | CI pin+zip+`--version`; preferably `scoop install` from GitHub bucket |
| Homebrew | CI pin+Linux layout+`--version`; preferably `brew install` on macOS AS (Gatekeeper) |
| winget bootstrap | `python validate_manifests.py`; optional `winget validate` / install --manifest |
| Bump automation | Fixture test: hashes match published sidecars exactly; no local recompute |

## Review focus (packaging-specific)

- Wrong path (`Formula/`, `bucket/`)  
- Hash inventiveness / mismatch vs sidecar  
- `.tar.gz` on Windows channels  
- Nested vs root binary layout  
- Broken autoupdate/checkver  
- Unpinned Actions  
- Secrets in repo  
- Advertised-but-dead install commands  
- winget identifier drift away from `Ledgerful.Ledgerful`  
- Claim language (open source / certified / compliant)

## Out of scope unless explicitly assigned

- homebrew-core submission  
- Linux distro packages (apt/dnf/AUR/nix)  
- Engine feature work  
- Notarization implementation (document interim only until engine pipeline has it)  
- crates.io publish for distribution  

## References

- `references/scoop.md`
- `references/homebrew.md`
- `references/winget.md`
- `references/engine-wiring.md`
