# winget — operator reference

## Important framing

winget is **not** an org-owned repo like scoop/homebrew. First package is submitted to **`microsoft/winget-pkgs`** (public, human-moderated). Subsequent versions can be automated via engine `winget-releaser` **after** the identifier exists.

## Bootstrap location (prep)

| | |
|---|---|
| Path | `C:\dev\coordinated\conductor\0051-PackageManagerDistribution\winget-bootstrap\` |
| PackageIdentifier | **`Ledgerful.Ledgerful`** (must match engine `release.yml` winget-releaser `identifier`) |
| Installer | Portable zip `ledgerful-x86_64-pc-windows-msvc.zip` (same as Scoop) |
| Hash source | Published `*.sha256` sidecar only |
| Bootstrap version | Latest **published** tag (e.g. `0.1.8`) — not a pre-release Cargo version without assets |

Layout:

```text
winget-bootstrap/
  README.md
  validate_manifests.py
  manifests/l/Ledgerful/Ledgerful/<version>/
    Ledgerful.Ledgerful.yaml
    Ledgerful.Ledgerful.installer.yaml
    Ledgerful.Ledgerful.locale.en-US.yaml
```

## Status discipline

| State | What agents may do |
|---|---|
| Prep | Edit YAMLs, run validator, document evidence |
| Submit | **Owner only** after explicit approval — public PR under human identity |
| Post-accept | Owner sets engine secret `WINGET_TOKEN`; later tags auto-PR via winget-releaser |

**Never** set `WINGET_TOKEN` on the engine before the package exists in winget-pkgs — the post-publish job fails.

**Never** run `wingetcreate submit` / open winget-pkgs PRs without owner approval.

## Constraints checklist

- Identifier: `Ledgerful.Ledgerful`  
- `InstallerType: zip` + portable nested installer pointing at root `ledgerful.exe`  
- SHA-256 from published sidecar (manifest often uppercase hex — match winget conventions but equal to sidecar)  
- Publisher: **Ledgerful, LLC**  
- License: PolyForm Noncommercial 1.0.0  
- Copy: **source-available** — ban “open source” / MIT claims in locale  

## Validate

```powershell
cd C:\dev\coordinated\conductor\0051-PackageManagerDistribution\winget-bootstrap
python .\validate_manifests.py

# If winget client available:
winget validate --manifest .\manifests\l\Ledgerful\Ledgerful\0.1.8
# Optional:
winget install --manifest .\manifests\l\Ledgerful\Ledgerful\0.1.8
ledgerful --version
```

Note: some `wingetcreate` versions have no standalone `validate` verb; do not “validate” by submitting.

## Owner submit steps (after approval)

1. Fork `microsoft/winget-pkgs`  
2. Copy `manifests/l/Ledgerful/Ledgerful/<ver>/` to the same path on a branch  
3. Open PR: `New package: Ledgerful.Ledgerful version <ver>`  
4. Wait for Microsoft validation + moderation  
5. After merge: set engine `WINGET_TOKEN` (PAT that can PR via fork)  
6. Later releases use SHA-pinned `vedantmgoyal9/winget-releaser` (see engine-wiring.md)

## Engine automation (after bootstrap)

- Job: `winget-release` in engine `release.yml`  
- Action pin: verify current SHA in `release.yml` (example historically `vedantmgoyal9/winget-releaser@4ffc7888…`)  
- Installer regex: portable Windows zip only  
- DoD for track: “submitted + validation-passing,” not “merged” (merge is external)

## Related

- Bootstrap README in-tree  
- Engine `docs/package-distribution.md` winget section  
- Track 0051 DoD-3  
