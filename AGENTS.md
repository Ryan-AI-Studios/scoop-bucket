# Ledgerful package-distribution agents

This repository is a **package-manager distribution surface**, not the Ledgerful engine.

| This repo | Role |
|---|---|
| `Ryan-AI-Studios/scoop-bucket` | Scoop bucket — root `ledgerful.json` only |

Related surfaces (read with the same skills):

| Path / repo | Role |
|---|---|
| `C:\dev\homebrew-tap` / `Ryan-AI-Studios/homebrew-tap` | Homebrew tap — root `ledgerful.rb` only |
| `C:\dev\coordinated\conductor\0051-PackageManagerDistribution\winget-bootstrap\` | Winget first-package prep (not submitted until owner approval) |
| `C:\dev\ledgerful` | Engine: templates, release CI, bump scripts, binaries |

## Cold start (mandatory)

1. Read `.agents/skills/onboarding/SKILL.md`
2. Read `.agents/skills/package-distribution/SKILL.md`
3. Read the channel reference for your task under `.agents/skills/package-distribution/references/`
4. Read track **0051** in `C:\dev\coordinated\conductor\` when doing track work

## Non-negotiables

- **Hashes come only from published `*.sha256` sidecars** — never recompute from archive bytes for manifest pins.
- **Root paths are load-bearing** — scoop: `ledgerful.json`; homebrew: `ledgerful.rb`. Engine release push expects those exact dests.
- **No engine runtime / crypto / network-posture changes** from packaging work.
- **Never push to `main` without PR + green CI.**
- **Do not set `WINGET_TOKEN`** on the engine until `Ledgerful.Ledgerful` exists in winget-pkgs.
- **Do not open winget-pkgs PRs** without explicit owner approval (outward-facing).

## Skills map

| Skill | When |
|---|---|
| `onboarding` | Session start, any packaging task |
| `package-distribution` | Scoop / Homebrew / winget / bump wiring |
| `implement` | Conductor track slices on these repos |
| `codex-review` | Cross-model gate after internal review is clean |
