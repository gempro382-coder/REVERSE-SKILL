# reverse-skill v1.0.0


AI-powered skill router for authorized reverse engineering, penetration testing, and security research.

## Highlights


## Install (quick)

```bash
git clone https://github.com/zhaoxuya520/reverse-skill.git
cd reverse-skill
# Windows
powershell -NoProfile -ExecutionPolicy Bypass -File skills/scripts/refresh-tool-index.ps1
# Linux / macOS
bash skills/scripts/refresh-tool-index.sh
```

Point your AI client at `RULES.md` / `README_AI.md`. See [README.md](../README.md).

## Verify

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File skills/scripts/smoke.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File skills/scripts/verify-routing-coherence.ps1
```

## Breaking / intentional product boundary

- This pack is a **skill router + bootstrap + journal**, not a full host security platform.
- Local tool paths live only in generated `tool-index` (not shipped in the tag content as absolute machine paths).

## Full changelog

See [CHANGELOG.md](../CHANGELOG.md#100--2026-07-18).

## License
