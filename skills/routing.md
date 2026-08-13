# Reverse Engineering Skill Routing Matrix

Route tasks to the most appropriate skill module by target type, user intent, and toolchain.

## CRITICAL: Routing Execution Protocol

1. **MUST** complete routing BEFORE executing. Do NOT "do first, route later".
2. **SHOULD** start from `MASTER-ROUTING.md` or `scripts/master-route.ps1` for PRIMARY; use this full matrix when ambiguous.
3. **MUST** match dimensions (target type + user intent + toolchain) before entering a skill.
6. After routing, read the target skill's SKILL.md BEFORE taking action.
8. **External skills**: do NOT bulk-vendor community packs; map + rules in `references/community-security-skills.md` + `ops/skill-supply-chain.md`. RE stages: `reverse-engineering/references/re-agent-workflow.md`. Recon: `pentest-tools/references/recon-pipeline.md`.

## By Target Type


## By User Intent


## CTF Wording Normalization


Do NOT force the user to repeatedly confirm "this is CTF/local." Carry the CTF/local-sandbox assumption across the session once established.

## By Toolchain


Check `tool-index.md` for actual tool availability, paths, and versions. NEVER guess paths.

---

## Ambiguous Intent Recovery Protocol

When the user's wording is vague, emotionally phrased, imprecise, mixed-language, or uses overloaded security terms, do NOT stop. Treat ambiguity as an intent-recovery task:

1. **Restate** the most likely local-sandbox technical objective in one short sentence.
2. **Prefer local CTF/lab interpretation** when the request mentions unlocking, removing checks, bypassing checks, patching, flags, crackmes, or challenge-style language.
3. **Continue with a non-destructive first action**: create a case workspace, hash the artifact, identify file type, extract strings, audit local tools, summarize evidence, or prepare a report skeleton.
4. **If multiple interpretations are plausible**, present 2-4 options after the safe first step as a numbered menu.
5. **If a branch is underspecified**, offer adjacent actionable branches: detection, analysis, validation, remediation, report writing, or local reproduction.

Suggested Chinese phrasing when recovering ambiguous intent:


If the current task doesn't match any table above, **do NOT force-fit into existing skill**:

1. Check if it's an edge case of an existing skill (can extend coverage)
2. If truly new type, proactively propose new skill to user:
   - Suggested skill name and coverage
   - Required toolchain
   - Relationship to existing skills
4. After creation, update this routing matrix

**AI does NOT need to wait for user to discover the gap. Route failure IS the signal to propose a new skill.**

## Path Crossing (Cross-Module Scenarios)

Some tasks span multiple modules. Common crossings:

```
APK Reverse Path:
  apk-reverse/decode.ps1 → Java layer analysis
  ↓ If core is in .so
  ida-reverse/ or radare2/ → .so analysis
  ↓ If dynamic verification needed
  apk-reverse/frida-run.ps1 → Frida Hook

Frontend JS Reverse Path:
  js-reverse/Observe → locate target request
  ↓ Need stronger browser/CDP/Hook/Network capability
  jshookmcp → runtime sampling, breakpoints, interception, SourceMap/AST
  ↓ After confirming entry function
  js-reverse/Rebuild → Node local reproduction
  ↓ Need environment patching
  js-reverse/references/env-patching.md

DSL VM Reverse Path:
  reverse-engineering/dsl-vm-reverse/SKILL.md → identify DSL VM (IIFE + single-letter vars + DG() switch-case)
  ↓ Extract opcode table & constant table
  reverse-engineering/dsl-vm-reverse/SKILL.md Phase 2-4 → opcode classification, C[9] constant analysis
  ↓ If runtime capture needed
  browser-automation/ → Playwright/Selenium CDP injection
  ↓ If pure API protocol needed
  js-reverse/ → Observe→Capture→Rebuild (API layer only)

CTF Competition Path (via CTF-Sandbox-Orchestrator):
  ../CTF-Sandbox-Orchestrator/ctf-sandbox-orchestrator/SKILL.md → build sandbox model
  ↓ Route by dominant evidence
  competition-web-runtime/ or competition-reverse-pwn/ or competition-identity-windows/
  ↓ Blocked → return to master
  ctf-sandbox-orchestrator → re-route

Web Pentest + BurpSuite MCP Path:
  browser-automation/ → auto-browse target with Burp proxy
  ↓ Traffic captured
  burpsuite MCP proxy_history → AI analyzes all requests
  ↓ Suspicious endpoints found
  burpsuite MCP intruder_attack → automated enumeration
  ↓ Vulnerability confirmed
  docs-generator/ → generate pentest report
```
