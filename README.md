# Scoop bucket for Ledgerful

Official [Scoop](https://scoop.sh) bucket for the [Ledgerful](https://github.com/Ryan-AI-Studios/Ledgerful) CLI.

| | |
|---|---|
| **Engine repo** | [Ryan-AI-Studios/Ledgerful](https://github.com/Ryan-AI-Studios/Ledgerful) |
| **Install page** | [ledgerful.dev/install](https://ledgerful.dev/install) |
| **License** | PolyForm Noncommercial 1.0.0 + small-entity commercial exception (see below) |

This repository is a Scoop **bucket** (app manifest only). Release automation in the engine repo rewrites `ledgerful.json` version + 64bit hash from the published `*.sha256` asset on each tag.

## Install

```powershell
scoop bucket add ledgerful https://github.com/Ryan-AI-Studios/scoop-bucket
scoop install ledgerful
```

Then:

```powershell
ledgerful --version
```

Supported platform (prebuilt release archive):

| Platform | Archive |
|---|---|
| Windows x86_64 | `ledgerful-x86_64-pc-windows-msvc.zip` (portable) |

The Windows artifact is a portable `.zip` with `ledgerful.exe` at the archive root (no 7-Zip prerequisite for Scoop extraction of standard ZIP). Hashes are pinned to the published release checksum sidecar (never recomputed locally by the bump scripts).

## Update

```powershell
scoop update ledgerful
```

## Uninstall

```powershell
scoop uninstall ledgerful
# optional: remove the bucket
scoop bucket rm ledgerful
```

## Layout (load-bearing)

```text
ledgerful.json            # Scoop manifest at repo root (NOT bucket/ledgerful.json)
LICENSE                   # PolyForm Noncommercial 1.0.0
COMMERCIAL-EXCEPTION.md   # small-entity exception
```

**Root path is intentional.** Engine release job `bump-manifests` pushes to `ledgerful.json` at the bucket root. Do not move the manifest under `bucket/` without updating that push path in the same change.

## License

Source for Ledgerful is **PolyForm Noncommercial 1.0.0** with a small-entity commercial exception. See:

- [`LICENSE`](./LICENSE)
- [`COMMERCIAL-EXCEPTION.md`](./COMMERCIAL-EXCEPTION.md)

## Maintenance

- Manifest template lives in-engine at `packaging/scoop/ledgerful.json`.
- On each engine release, `scripts/bump-manifests.*` rewrites version + hash from published `*.sha256` files only.
- When secret `MANIFEST_PUSH_TOKEN` is configured, release CI commits the bumped `ledgerful.json` here automatically.
- Manual seed / review PRs may still land on this repo for structure changes (README, CI, license).

## Related distribution channels

- Homebrew: [Ryan-AI-Studios/homebrew-tap](https://github.com/Ryan-AI-Studios/homebrew-tap)
- winget: `Ledgerful.Ledgerful` (external review on `microsoft/winget-pkgs`)
- `cargo binstall --git https://github.com/Ryan-AI-Studios/Ledgerful`
- One-line installers: see [installation docs](https://github.com/Ryan-AI-Studios/Ledgerful/blob/main/docs/installation.md)
