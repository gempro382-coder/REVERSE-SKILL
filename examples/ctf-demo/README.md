

```text
1. 用户任务："分析这个 CTF pwn 题，栈溢出 gets"
2. 路由：master-route.ps1 -Hint "CTF pwn 栈溢出" → PRIMARY R17 (pwn-chain)
3. 授权：case-init.ps1 -Hint ... -CaseName ctf-demo -AuthGranted → scope.md
4. 执行：时间线追加 + 证据 E-001/E-002 + workitems 更新
5. 产出：报告（docs-generator）+ field-journal 脱敏沉淀
```


|------|------|


```powershell
# 初始化真实 case（授权目标）
powershell -NoProfile -ExecutionPolicy Bypass -File skills/scripts/case-init.ps1 `
  -Hint "你的任务" -CaseName my-case -AuthGranted -TargetUrl "https://target/" `
  -NetworkProfile authorized_target_only

# 追加证据
powershell -File skills/scripts/append-evidence.ps1 -CaseRoot work\my-case `
  -Id E-001 -Title "..." -ReproCommand "..."
```
