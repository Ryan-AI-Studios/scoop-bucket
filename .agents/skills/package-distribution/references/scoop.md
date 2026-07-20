# Scoop bucket — operator reference

## Repo

| | |
|---|---|
| Local | `C:\dev\scoop-bucket` |
| Remote | https://github.com/Ryan-AI-Studios/scoop-bucket |
| Manifest path | **`ledgerful.json` at repo root** |
| Engine template | `C:\dev\ledgerful\packaging\scoop\ledgerful.json` |
| Engine push dest | `ledgerful.json` (root) via `release.yml` `push_one` |

## Install (users)

```powershell
scoop bucket add ledgerful https://github.com/Ryan-AI-Studios/scoop-bucket
scoop install ledgerful
ledgerful --version
```

Update / uninstall:

```powershell
scoop update ledgerful
scoop uninstall ledgerful
scoop bucket rm ledgerful   # optional
```

## Manifest contract

Required shape (conceptually):

```json
{
  "version": "X.Y.Z",
  "architecture": {
    "64bit": {
      "url": "https://github.com/Ryan-AI-Studios/Ledgerful/releases/download/vX.Y.Z/ledgerful-x86_64-pc-windows-msvc.zip",
      "hash": "<64 lowercase hex from *.sha256>"
    }
  },
  "bin": "ledgerful.exe",
  "checkver": { "github": "https://github.com/Ryan-AI-Studios/Ledgerful" },
  "autoupdate": {
    "architecture": {
      "64bit": {
        "url": "https://github.com/Ryan-AI-Studios/Ledgerful/releases/download/v$version/ledgerful-x86_64-pc-windows-msvc.zip",
        "hash": { "url": "$url.sha256" }
      }
    }
  }
}
```

### Layout requirements

- Archive must be **`.zip`** (not `.tar.gz`) so Scoop does not require 7-Zip.
- `ledgerful.exe` must sit at the **zip root** (alongside LICENSE/README/web/ is fine).
- `hash` is raw 64-hex SHA-256 (no `sha256:` prefix).

## CI expectations

Workflow: `.github/workflows/ci.yml` on `windows-latest` (lightweight; usually minutes).

Should:

1. Require root `ledgerful.json`; reject `bucket/` tree  
2. Parse version, url, hash, bin  
3. Require `.zip` + expected basename  
4. Fail-closed if `checkver` / `autoupdate` missing  
5. Download published `*.sha256` for tag `v{version}` and compare pin  
6. Download zip, `Get-FileHash`, extract, run root `ledgerful.exe --version`  
7. Require LICENSE, COMMERCIAL-EXCEPTION.md, README install docs  

**PowerShell gotcha:** in double-quoted strings write `"${version}: $url"` not `"$version: $url"` (drive-scope parse error on windows-latest).

Checkout action: SHA-pin (same posture as engine / homebrew-tap). Permissions: `contents: read`.

## Local verification (dev)

```powershell
# Template identity (after intentional sync)
# Compare-Object / Get-FileHash vs C:\dev\ledgerful\packaging\scoop\ledgerful.json

# Sidecar pin
gh release download v0.1.8 --repo Ryan-AI-Studios/Ledgerful `
  --pattern ledgerful-x86_64-pc-windows-msvc.zip.sha256 --dir $env:TEMP\scoop-pin

# Path install smoke (pre-merge)
scoop install C:\dev\scoop-bucket\ledgerful.json
ledgerful --version

# Post-merge GitHub smoke
scoop bucket add ledgerful https://github.com/Ryan-AI-Studios/scoop-bucket
scoop install ledgerful
```

Note: `scoop bucket add` with a bare local path may fail; use Git URL or direct manifest path install for local smoke.

## Safe change patterns

| Change | How |
|---|---|
| New release version/hash | Prefer engine release bump automation; do not hand-edit unless automation blocked |
| README / CI / license structure | PR on scoop-bucket; keep root path documented |
| Schema fields | Keep autoupdate hash as `$url.sha256`; match published sidecar naming |

## Do not

- Create `bucket/ledgerful.json` as the only copy  
- Point url at a `.tar.gz`  
- Set `bin` to a nested path unless the zip layout actually nests  
- Recompute hash from a local build of the engine  

## Related

- Engine docs: `C:\dev\ledgerful\docs\package-distribution.md`  
- Track: `C:\dev\coordinated\conductor\0051-PackageManagerDistribution\`  
