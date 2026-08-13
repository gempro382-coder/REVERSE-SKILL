

```markdown
### E-{nnn}
- title:
- observed_at:
- source_type: command | screenshot | file | log | memory | network | manual
- source_ref: {path or command id}
- content_hash: {sha256 of artifact if file, else n/a}
- artifact_path: {relative path under case root when content_hash is recorded, else n/a}
- repro_command: |
    {exact command}
- raw_excerpt: |
    {脱敏摘录}
- linked_workitem: WI-{nnn} | n/a
- supersedes: E-{nnn} | none
```


```powershell
powershell -File skills/scripts/append-evidence.ps1 -CaseRoot work/<case> `
  -Id E-001 -Title "..." -ReproCommand "..." -Severity info -Status observed
```

When the evidence is a case-local file, pass `-ArtifactPath` to record a SHA-256 fixity value and a relative artifact path. Review the complete case graph before handoff:

```bash
python3 skills/case-review/scripts/review_case.py work/<case> --verify-hashes --strict
```

The review is read-only and checks scope fields, Evidence records, work item and timeline references, structured Findings, Paths, and artifact hash matches.


```markdown
### F-{nnn}
- title:
- severity: critical | high | medium | low | info | n/a_re
- category: vuln | misconfig | design | reverse_algo | bypass | other
- status: candidate | validated | false_positive | accepted_risk
- evidence_ids: [E-001, E-002]
- location: {file:line | addr | url | class.method}
- impact:
- confidence: high | medium | low
- repro_steps:
  1.
  2.
- remediation: {or n/a for pure RE}
- optional_attack: {ATT&CK ID or empty}
```


```markdown
### P-{nnn}
- title:
- path_type: attack | callflow | solve
- start:
- goal:
- steps:
  1. action: — evidence: E-xxx — finding: F-xxx | none
  2. action: — evidence: E-xxx — finding: F-yyy | none
- residual_risks:
```
