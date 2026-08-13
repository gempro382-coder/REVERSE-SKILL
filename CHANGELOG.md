# Changelog

All notable changes to **reverse-skill** are documented in this file.

Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Versioning follows [Semantic Versioning](https://semver.org/).

## [Unreleased]


### Added


### Fixed


### Security

- Core scripts do not write client-global instruction files; client-specific integration remains outside the routing core.

### Fixed

- Linux/macOS bootstrap: register PentestSwarm MCP with a verified executable path after Go install or when already installed

### Security

- Added `docs/PACKAGE-SECURITY-AUDIT.md`: static audit of package executables (no backdoor / no auto DB wipe found)
- Pin supply-chain floating tags: jshook `@0.3.4`, pentestswarm `v0.1.0`
- Bootstrap integrity: GitHub zip/jar downloads verify `assetSha256` (manifest) or GitHub API `digest`; mismatch deletes file and fails
- Pin jadx `v1.5.6` and apktool `v3.0.2` with published SHA256
- Remove shell evaluation from Kali user-home resolution and pass Frida hosts through argument arrays
- Write the Burp MCP token atomically with owner-only permissions on POSIX filesystems
- Reconnect the Burp MCP bridge when Burp starts after the bridge and parse one stdio message per line
- Enable authentication for bootstrapped Anything Analyzer MCP servers and register the bearer token with supported clients
- Stop each stale IDA MCP process individually before starting a replacement
- Add Bash case initialization, authorization guard, and a structured router that reads the same `routing.json` as PowerShell
- Verify Bash routing parity in CI without introducing a client-specific plugin manifest
- Enforce immutable Kali/Windows bootstrap sources for Frida, IDA MCP, Agent Browser, ProxyCat, Nuclei, and pwntools
- Reject path-like Bash case names and scope authorization checks to the contract's auth/network/signoff sections
- Align Bash network defaults with PowerShell: authorized URLs use `authorized_target_only`, while offline readiness requires an explicit local sample
- Pin GitHub Actions checkout to the immutable v4.2.2 commit and keep the CI case count synchronized
- Create a functional Kali `proxycat` wrapper after installing the pinned source checkout
- Scope PowerShell authorization fields to their contract sections and reject unsupported network modes in both guards
- Reject unsupported network profiles during case initialization so invalid scopes are never emitted as ready
- Generate `skills/INDEX.md` from tracked skills only, excluding ignored local modules so clean-clone CI stays reproducible
- Fail routing coherence when a configured skill is missing or only exists as an untracked local file


### Added

- `case-review/`: read-only Evidence Graph Review with scope, timeline, work item, Finding, Path, and optional SHA-256 fixity checks
- Wired into `MASTER-ROUTING.md`, `master-route.ps1`, routing tables, domain map, role-map, coherence tests

### Removed

- `game-reverse/` (not a product focus; Unity/IL2CPP remains via `reverse-engineering` + seed-014)


First **formal** public release of the reverse-skill skill-router pack.

### Added

#### Ops / combat contract layer (`skills/ops/`)


#### PRIMARY routing & case tooling (`skills/scripts/`)


#### Core skills & docs

- Full skill matrix: APK / IDA / radare2 / JS / .NET / mobile / malware / pwn / firmware / EDR / pentest / API / LLM / supply-chain / crypto / binary-diff / patch-diff / attack-chain / docs & diagrams
- `MASTER-ROUTING.md` + `routing.md` / `routing_zh.md` three-axis matrix
- Bootstrap + tool-index pipeline (`bootstrap-reverse.ps1` / `.sh`, `refresh-tool-index`)
- `field-journal` precedent library + completion checklist
- Multi-platform paths: Windows primary, Linux / macOS / Kali docs and scripts
- CTF-Sandbox-Orchestrator competition sub-skills
- Burp MCP extension package (`burp-mcp-full/`)

#### Quality / localization

- UTF-8 integrity for Chinese docs (`RULES_zh`, `routing_zh`, related guides)
- Client-side lab playbook and recon pipeline references for authorized testing friction reduction

### Notes

- `skills/tool-index.md` / `tool-index.json` are **machine-local** and intentionally gitignored; generate via `refresh-tool-index` after clone.
- This tag freezes the skill-router product surface at commit `9fc280b` plus this release metadata.

### Links

- Tag: `v1.0.0`
- Repository: https://github.com/zhaoxuya520/reverse-skill

[Unreleased]: https://github.com/zhaoxuya520/reverse-skill/compare/v1.0.1...HEAD
[1.0.1]: https://github.com/zhaoxuya520/reverse-skill/compare/v1.0.0...v1.0.1
[1.0.0]: https://github.com/zhaoxuya520/reverse-skill/releases/tag/v1.0.0
