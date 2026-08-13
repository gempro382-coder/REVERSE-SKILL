

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File skills/scripts/refresh-tool-index.ps1
```


```powershell
# 路由回归（162 用例，修改 routing.json 后必跑）
powershell -NoProfile -ExecutionPolicy Bypass -File skills/scripts/test-routing.ps1

# 结构一致性 + 供应链 pin gate
powershell -NoProfile -ExecutionPolicy Bypass -File skills/scripts/verify-routing-coherence.ps1

# 冒烟（verify + 脚本解析 + 快速路由）
powershell -NoProfile -ExecutionPolicy Bypass -File skills/scripts/smoke.ps1
```
