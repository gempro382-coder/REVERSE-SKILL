

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File skills\scripts\case-init.ps1 -Hint "<任务一句话>" -CaseName "my-case"
# 默认产出：当前分析项目的 work/<case>/scope.md 等
# 从其他目录调用 skill 时显式指定：-ProjectRoot "C:\path\to\analysis-project"
```


```markdown
# Case Scope

## meta
- case_id: {YYYYMMDD-short}
- created: {ISO-8601}
- operator: {name or local}
- primary_skill: {from master-route}
- lead_role: lead   # see ops/role-map.md
- specialist_roles: []  # e.g. cie, cpe, cre

## auth
- status: granted | pending | denied
- basis: written_contract | bug_bounty_scope | ctf_public | own_system | lab_only
- evidence_of_auth: {ticket/path or "CTF public" or "owner-operated"}
- MUST NOT proceed if status != granted

## in_scope
- assets: []          # hosts, domains, APK paths, binaries, URLs
- surfaces: []        # web, mobile, binary, network, api
- activities: []      # recon, reverse, exploit_validate, report

## out_of_scope
- assets: []
- activities: []      # e.g. DoS, phishing real users, data exfil

## network_profile
- mode: offline | lab_only | authorized_target_only | unrestricted_lab
- notes: |
    offline = 无对外发包（纯静态/本地样本）
    lab_only = 仅 lab/VM IP
    authorized_target_only = 仅 in_scope 资产
- MUST NOT use unrestricted against production without written auth

## deliverables
- report: true
- field_journal: true
- diagrams: true
- timeline: true

## constraints
- timebox: {}
- stealth: low | medium | high
- data_handling: anonymize | no_user_pii

## signoff
- ready_for_act: false
- checklist:
  - [ ] auth.status = granted
  - [ ] in_scope.assets non-empty OR offline sample path set
  - [ ] network_profile.mode chosen
  - [ ] out_of_scope reviewed
```


```text
RULES / MASTER-ROUTING / SKILL:
  1) master-route → PRIMARY
  2) case-init 或手写 scope.md
  3) auth 未 granted → STOP，只允许补授权材料
  4) ready_for_act = true → 打开 PRIMARY SKILL.md → ACT
```


|------|------|------|
