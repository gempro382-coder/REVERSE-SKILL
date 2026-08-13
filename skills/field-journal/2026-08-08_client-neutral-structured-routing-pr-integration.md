

- auth_basis: repository_owner_authorized
- network_profile: authorized_upstream_only
- asset_types: [source_repository, pull_request_refs, local_tests]


- lead_role: lead
- specialists: [cae, doc]


- path_type: callflow


```text
git merge --no-ff --no-commit <pr-ref>
powershell -File skills/scripts/test-routing.ps1
powershell -File skills/scripts/verify-routing-coherence.ps1
bash skills/scripts/master-route.sh --hint "case review evidence graph"
```


---
