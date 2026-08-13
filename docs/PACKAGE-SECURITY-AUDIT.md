

- `Invoke-Expression` / `IEX` / `FromBase64String` / `DownloadString`


```
skills/scripts/*.ps1|*.sh + lib/ToolDiscovery.ps1
skills/apk-reverse/scripts/*
skills/radare2/scripts/*
skills/ida-reverse/scripts/*
skills/browser-automation/scripts/*
skills/diagram-generator/scripts/*.py
skills/case-review/scripts/*.py
kali/scripts/*
burp-mcp-full/mcp-bridge.js (+ Java 扩展源)
```


```powershell
# 可执行面快速体检（示例）
rg -n "Invoke-Expression|FromBase64String|DownloadString|rm -rf /|DROP DATABASE" skills/scripts skills/*/scripts kali/scripts burp-mcp-full -g "*.ps1" -g "*.sh" -g "*.py" -g "*.js"
```


'@
