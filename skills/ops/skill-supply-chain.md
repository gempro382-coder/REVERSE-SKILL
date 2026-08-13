

|--------|------|----------|


```text
□ 来源：官方 org / 已审计列表（如 ToB curated）/ 用户自有
□ 阅读全部 SKILL.md + scripts/* + package 依赖
□ 无神秘外连、无读取 ~/.ssh / 浏览器库 的默认步骤
□ 与本包路由冲突时：以本包 MASTER-ROUTING + scope 为准
□ 不复制进 monorepo 除非走 CONTRIBUTING 与脱敏
□ 更新 skills/references/community-security-skills.md 记录来源日期
```


|------|------|------|


```powershell
# 列出将引入的脚本扩展名
Get-ChildItem -Recurse -Include *.ps1,*.sh,*.py,*.js | Select-Object FullName
# 粗搜危险模式（人工复核，非完备）
# 在外部目录执行：Select-String -Pattern 'Invoke-WebRequest|curl .\||wget .\||~/.ssh|exfil'
```
