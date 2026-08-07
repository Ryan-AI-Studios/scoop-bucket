# Homebrew tap — operator reference

## Repo

| | |
|---|---|
| Local | `C:\dev\homebrew-tap` |
| Remote | https://github.com/Ryan-AI-Studios/homebrew-tap |
| Formula path | **`ledgerful.rb` at repo root** (not `Formula/ledgerful.rb`) |
| Engine template | `C:\dev\ledgerful\packaging\homebrew\ledgerful.rb` |
| Engine push dest | `ledgerful.rb` (root) via `release.yml` `push_one` |
| Kind | CLI **formula**, not cask |

## Install (users)

```bash
brew install Ryan-AI-Studios/tap/ledgerful
# or:
brew tap Ryan-AI-Studios/tap
brew install ledgerful

ledgerful --version
```

## Formula contract

- `version "X.Y.Z"`
- Per-arch `url` + `sha256` for:
  - `ledgerful-aarch64-apple-darwin.tar.gz`
  - `ledgerful-x86_64-apple-darwin.tar.gz`
  - `ledgerful-x86_64-unknown-linux-gnu.tar.gz`
- `install` resolves the binary with buildpath-anchored dual glob:
  `Pathname.glob(buildpath/"ledgerful")` first (Homebrew stages the tar’s
  top-level `ledgerful-{target}/` as `buildpath`, so the binary is usually a
  direct child), then nested fallback
  `Pathname.glob(buildpath/"ledgerful-*/ledgerful")` → `bin.install … => "ledgerful"`
- `license :cannot_represent` — Homebrew cannot encode PolyForm NC + commercial exception as one SPDX id; product license is still PolyForm NC + exception files in the repo
- `caveats` document Gatekeeper interim if unsigned/un-notarized:

```bash
xattr -d com.apple.quarantine "$(which ledgerful)"
```

Proper long-term fix = codesign + notarize in **engine** release pipeline (not a tap workaround).

## Archive layout

Unix release tarballs nest:

```text
ledgerful-{target}/ledgerful
```

CI and formula must agree on this. Windows zip layout is irrelevant to Homebrew.

## CI expectations

Workflow: `.github/workflows/ci.yml` (often `ubuntu-latest`).

Should:

1. Require root `ledgerful.rb`; reject `Formula/`  
2. Parse version + ≥3 sha256 pins  
3. Ensure urls contain `/v{version}/`  
4. Download each published `*.sha256` for the formula urls; match pins  
5. Linux tarball: verify archive hash; prove **archive** nesting (`ledgerful-*/ledgerful`); model brew **staging** (exactly one top-level `ledgerful-*` dir = buildpath); dual-glob Ruby resolve against that stage; run staged `--version`  
6. Formula source must contain `Pathname.glob(buildpath` and must **not** use `Dir["ledgerful-`  
7. Prefer real `brew install --formula ./ledgerful.rb` smoke (Linuxbrew job) when CI infra allows  
8. Require LICENSE, COMMERCIAL-EXCEPTION.md, README; quarantine guidance present  

Apple Silicon / Gatekeeper end-user install may remain a manual residual — record in `deferred.md` if not run; do not fake it.

## Local verification

```bash
# On macOS / Linuxbrew host:
brew install Ryan-AI-Studios/tap/ledgerful
ledgerful --version
# If Gatekeeper blocks:
xattr -d com.apple.quarantine "$(which ledgerful)"
```

Pin check without brew:

```bash
gh release download v0.1.8 --repo Ryan-AI-Studios/Ledgerful --pattern "*.sha256" --dir /tmp/lf-sha
# Compare first field of each sidecar to formula sha256 lines
```

## Safe change patterns

| Change | How |
|---|---|
| Version/sha256 | Engine bump automation preferred |
| install() path / caveats / CI | PR on homebrew-tap; keep root formula |
| Template change | Edit engine `packaging/homebrew/ledgerful.rb`, then ensure live tap matches |

## Do not

- Move formula under `Formula/` without engine push-path update  
- Convert to cask without an explicit track decision  
- Drop quarantine caveats while macOS artifacts remain un-notarized  
- Add a Linux aarch64 bottle/url unless engine actually publishes that asset  

## Related

- Engine docs: `C:\dev\ledgerful\docs\package-distribution.md`  
- Track: `C:\dev\coordinated\conductor\0051-PackageManagerDistribution\`  
