# CLAUDE.md

This repository is a **task skill router** for authorized reverse engineering, mobile/security analysis, and pentest workflows.

## On Any Task

`RULES.md` is the single source of truth for behavior chain and authorization.

Routing order:

1. `skills/MASTER-ROUTING.md` or `skills/scripts/master-route.ps1 -Hint "..."`
3. `skills/routing.md` when ambiguous; roles in `skills/ops/role-map.md`
4. Open PRIMARY `SKILL.md` and execute ACTION REQUIRED
6. `skills/tool-index.md` for real tool paths (never guess)


## First-Run Setup

`skills/tool-index.md` is not in fresh clones. Generate it:

```bash
# Windows
powershell -NoProfile -ExecutionPolicy Bypass -File skills/scripts/refresh-tool-index.ps1

# macOS / Linux
bash skills/scripts/refresh-tool-index.sh

# Kali
bash kali/scripts/refresh-tool-index.sh
```

Read `README_AI.md` for full bootstrap sequence.

## Coherence check

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File skills/scripts/verify-routing-coherence.ps1
```
