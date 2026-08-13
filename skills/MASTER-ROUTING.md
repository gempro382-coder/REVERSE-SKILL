

```text
1. 先路由后动手
2. 输出 PRIMARY 路径 + 一句话依据
3. case-init / scope.md（ops/scope-contract）— auth 未 granted 禁止对目标 ACT
4. 指定 lead + specialist 角色（ops/role-map）
5. 立即打开 PRIMARY 的 SKILL.md → ACTION REQUIRED
6. 工具路径只认 tool-index；缺则 bootstrap（仅 manifest 能力）
7. 过程追加 timeline / workitems；结论走 Evidence→Finding→Path
8. 未命中 → 读 routing.md 全表或提议新 skill
```

```powershell
powershell -File skills\scripts\master-route.ps1 -Hint "<用户任务>"
# 默认写出当前项目的 work/master-route-<ts>/route-scope.md；从其他目录调用时显式指定项目根
powershell -File skills\scripts\master-route.ps1 -Hint "<用户任务>" -ProjectRoot "C:\path\to\analysis-project"
powershell -File skills\scripts\case-init.ps1 -Hint "<用户任务>" -CaseName "my-case"
# case 默认写入当前项目的 work/<case>/；-PackageRoot 保持兼容，-ProjectRoot 优先级更高
powershell -File skills\scripts\case-init.ps1 -Hint "<用户任务>" -CaseName "my-case" -ProjectRoot "C:\path\to\analysis-project"
# 一次成型可 ACT（授权 + 目标 + 网络档）：
powershell -File skills\scripts\case-init.ps1 -Hint "<任务>" -CaseName "my-case" -AuthGranted -TargetUrl "https://target/" -NetworkProfile authorized_target_only
# 冒烟：verify + 脚本解析 + 路由矩阵（含中文 Hint）
powershell -File skills\scripts\smoke.ps1
# ACT 前轻量 scope 门禁（未就绪 exit 2；-Force 仅警告）
powershell -File skills\scripts\case-guard.ps1 -CaseRoot work\my-case
# Evidence 追加
powershell -File skills\scripts\append-evidence.ps1 -CaseRoot work\my-case -Id E-001 -Title "..." -ReproCommand "..."
python3 skills/case-review/scripts/review_case.py work/<case> --verify-hashes --strict
```


|------|------|
| `reverse-engineering/references/re-agent-workflow.md` | RE：triage→static→dynamic→synthesis |


|----|------|---------|
| **R1** | APK / smali / jadx / apktool | `apk-reverse/` |
| **R2** | IPA / iOS / Objection / MobSF / mobile | `mobile-reverse/` |
| **R5** | .NET / dnSpy / de4dot / ConfuserEx | `dotnet-reverse/` |
| **R7** | radare2 / r2 | `radare2/` |
| **R24** | Windows / AD / Kerberos / AD CS | `windows-ad/` |
| **R31** | macOS / Mach-O | `macos-reverse/` |


|------|------|


```text
RULES.md → MASTER-ROUTING.md → PRIMARY SKILL.md
  → (可选) routing.md 三轴 / field-journal
  → tool-index.md → bootstrap → ACT
```
