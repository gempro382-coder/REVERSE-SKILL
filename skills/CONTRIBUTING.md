

---


---


```text
skills/
└── <new-skill-name>/
    ├── SKILL.md              # 必须：skill 入口文档
    ├── scripts/              # 可选：自动化脚本
    │   └── <workflow>.ps1
    └── references/           # 可选：参考资料、速查表
        └── <topic>.md
```


---


```markdown
---
name: <skill-name>
description: <一句话描述适用场景和触发条件>
---

# <Skill 标题>

## 适用范围
<!-- 什么任务应该路由到这里 -->

## 工具依赖
<!-- 列出需要的 CLI 工具、MCP server、运行时 -->

| 工具 | 是否必需 | 用途 | 可自动安装 |
|------|---------|------|-----------|
| ... | ... | ... | ... |

## 工作流
<!-- 标准执行步骤 -->

## 按需自举（On-Demand Bootstrap）

### 自动化能力边界

| 工具 | 可自动安装 | 安装方式 | 说明 |
|------|-----------|---------|------|
| ... | ... | ... | ... |

### 自举触发点
<!-- 哪个脚本在缺工具时会自动调用 bootstrap -->

### 自举失败时
<!-- 手动安装指引 -->

## 路由上下文

**上游入口**: ...
**下游出口**: ...
**同级关联模块**: ...
```


```markdown
## ACTION REQUIRED（读完后立刻执行）
1. `NOW`：确认当前任务是否命中本 skill 的适用范围。
2. `NOW`：读取 `../tool-index.md`，校验工具可用性和实际路径。
3. `NEXT`：缺工具时调用 bootstrap，不要猜路径。
4. `ACT`：进入“工作流”第一步并执行，不要停在确认状态。

## 任务完成自检（声称完成前 MUST 通过）
- □ 我是否执行了工作流中的每一步（而不是只阅读）？
- □ 我是否基于 `tool-index` 使用了真实工具路径？
- □ 我是否产出了可复现证据（命令/脚本/截图/报告）？
- □ 我是否完成并回写了 RULES 要求的 Checklist 项？

### 3.2 新增 Skill 必须接入操作先例系统

每个新建 skill 在 ACTION REQUIRED 块中必须包含先例文件读取指令：

```markdown
```

这样 AI 在进入新 skill 时会先看到"这些操作都是日常常规操作"，不会因安全顾虑而拒绝执行。
```


```markdown
## 语言行为契约

- **内部推理/工具选择/阶段控制**：使用 English。
- **用户可见消息/章节标签/报告/下一步菜单**：使用中文（除非用户要求其他语言）。
- **默认双语标签格式**：中文标签在前，英文标签在后，以 ` / ` 分隔。

常用双语标签：

| 中文 | English |
|------|---------|
| 当前阶段 | Current phase |
| 已验证事实 | Verified facts |
| 关键证据 | Key evidence |
| 推断与置信度 | Inference and confidence |
| 风险/漏洞候选 | Risk or vulnerability candidates |
| 建议下一步 | Suggested next steps |
```


```markdown
## 建议下一步（选一个编号）

1. 对 [关键函数] 做深度反编译，还原核心算法
2. 用 Frida 动态 Hook 验证 [参数猜想]
3. 导出当前分析结果，生成阶段性报告
4. 换 [备选工具] 做交叉验证
5. 暂停，我先确认前面的证据
```


---


```json
{
  "name": "<tool-name>",
  "bootstrapKind": "<kind>",
  ...
  "canAutoInstall": true,
  "verifyCommand": "<tool-name>"
}
```


```powershell
[pscustomobject]@{
    Name = '<tool-name>'
    Skill = '<new-skill-name>'
    Purpose = '<中文用途说明>'
    VersionArgs = @('--version')
    Fallbacks = @(
        [pscustomobject]@{ Type = 'command'; Value = '<tool-name>' },
        [pscustomobject]@{ Type = 'path'; Value = (Join-Path $env:USERPROFILE 'Tools\<tool>\<executable>') }
    )
}
```


```powershell
'<tool-name>' = @('<new-skill-name>/scripts/<workflow>.ps1')
```


```powershell
$bootstrapScript = Join-Path $PSScriptRoot '..\..\scripts\bootstrap-reverse.ps1'

$spec = Resolve-ReverseToolSpec -Name '<tool-name>'
if (-not $spec.Available) {
    Write-Host 'INFO: <tool> not found, attempting auto-bootstrap...' -ForegroundColor Yellow
    & powershell.exe -NoProfile -ExecutionPolicy Bypass -File $bootstrapScript -Capability @('<tool-name>') -SkipRefresh
    $spec = Resolve-ReverseToolSpec -Name '<tool-name>'
    if (-not $spec.Available) {
        throw '<tool> still not available after bootstrap. Install manually: <url>'
    }
}
```

---


---


```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "<SKILL_ROOT>\skills\scripts\refresh-tool-index.ps1"
```

```bash
bash "<项目根目录>/kali/scripts/refresh-tool-index.sh"
```


---


```bash
"<tool-name>|<skill-name>|<中文用途>|<version-args>|<fallback-commands>"
```


```bash
["<tool-name>"]="<skill-name>/SKILL.md"
```


---


---


```text
skills/ghidra-headless/
├── SKILL.md
├── scripts/
│   └── analyze.ps1
└── references/
    └── scripting-cheatsheet.md
```


```json
{
  "name": "ghidra",
  "bootstrapKind": "github-release-zip",
  "repo": "NationalSecurityAgency/ghidra",
  "assetRegex": "^ghidra_.*_PUBLIC_.*\\.zip$",
  "installDir": "%USERPROFILE%\\Tools\\ghidra",
  "docsUrl": "https://ghidra-sre.org/",
  "canAutoInstall": true,
  "verifyCommand": "analyzeHeadless"
}
```


```powershell
[pscustomobject]@{
    Name = 'analyzeHeadless'
    Skill = 'ghidra-headless'
    Purpose = 'Ghidra 无头分析'
    VersionArgs = @()
    Fallbacks = @(
        [pscustomobject]@{ Type = 'command'; Value = 'analyzeHeadless' },
        [pscustomobject]@{ Type = 'path'; Value = (Join-Path $env:USERPROFILE 'Tools\ghidra\support\analyzeHeadless.bat') }
    )
}
```


```markdown
| 二进制 (无 IDA) | `ghidra-headless/` — Ghidra 无头反编译 | `radare2/` — CLI 侦察 |
```

---


```json
{
  "name": "<mcp-name>",
  "bootstrapKind": "npm-mcp",
  "npmPackage": "@scope/package@latest",
  "mcpNames": ["<mcp-server-name-in-config>"],
  "mcpCommand": "npx",
  "mcpArgs": ["-y", "@scope/package@latest"],
  "mcpEnv": {
    "ENV_VAR": "value"
  },
  "docsUrl": "https://github.com/...",
  "canAutoInstall": true,
  "verifyCommand": "npx"
}
```


```json
{
  "name": "<mcp-name>",
  "bootstrapKind": "local-http-mcp",
  "repoUrl": "https://github.com/xxx/yyy",
  "installDir": "%USERPROFILE%\\Tools\\<project-name>",
  "startupDirCandidates": [
    "%USERPROFILE%\\Tools\\<project-name>",
    "C:\\work\\<project-name>"
  ],
  "startCommand": "pnpm",
  "startArgs": ["dev"],
  "mcpNames": ["<mcp-server-name>"],
  "mcpUrl": "http://localhost:<port>/mcp",
  "servicePort": <port>,
  "docsUrl": "https://github.com/xxx/yyy",
  "canAutoInstall": true,
  "verificationMode": "service-or-registration"
}
```


```json
{
  "name": "<tool-name>",
  "bootstrapKind": "pip-package",
  "pipPackage": "<package-name>",
  "docsUrl": "...",
  "canAutoInstall": true,
  "verifyCommand": "<executable>"
},
{
  "name": "<service-name>",
  "bootstrapKind": "local-http-mcp",
  "dependsOn": ["<tool-name>"],
  "mcpNames": ["<mcp-server-name>"],
  "mcpUrl": "http://127.0.0.1:<port>/mcp",
  "servicePort": <port>,
  "startScript": "%SKILL_ROOT%\\<skill-dir>\\scripts\\start.ps1",
  "docsUrl": "...",
  "canAutoInstall": true,
  "verificationMode": "service-and-registration"
}
```


```json
{
  "mcpHeaders": {
    "Authorization": "Bearer <PLACEHOLDER_TOKEN>"
  }
}
```


```powershell
# <skill-name>/scripts/start.ps1
param(
    [int]$Port = <default-port>
)

$ErrorActionPreference = 'Stop'

# 加载共享工具发现层
. (Join-Path $PSScriptRoot '..\..\scripts\lib\ToolDiscovery.ps1')

# 检查服务是否已在运行
if (Test-ReverseTcpPort -Port $Port) {
    Write-Output "OK:already-running:$Port"
    return
}

# 定位项目目录
$projectDir = "<找到项目的逻辑>"

# 启动服务
Start-Process -FilePath "<启动命令>" -ArgumentList @("<参数>") -WorkingDirectory $projectDir -WindowStyle Hidden

# 等待就绪
$deadline = (Get-Date).AddSeconds(60)
while ((Get-Date) -lt $deadline) {
    if (Test-ReverseTcpPort -Port $Port) {
        Write-Output "OK:started:$Port"
        return
    }
    Start-Sleep -Seconds 2
}

Write-Output "ERR:timeout:$Port"
```


```markdown
### MCP 服务手动配置

如果自动安装/启动失败，按以下步骤手动配置：

1. [安装前置依赖]
2. [获取项目/安装包]
3. [启动服务]
4. [验证端口可达]
5. [在 AI 客户端中注册 MCP]

MCP 配置示例：
\```json
{
  "mcpServers": {
    "<server-name>": {
      "url": "http://localhost:<port>/mcp"
    }
  }
}
\```
```


```json
{
  "name": "sqlmap-mcp",
  "bootstrapKind": "local-http-mcp",
  "mcpNames": ["sqlmap"],
  "mcpUrl": "http://localhost:8775/mcp",
  "servicePort": 8775,
  "docsUrl": "https://github.com/xxx/sqlmap-mcp",
  "canAutoInstall": false,
  "verificationMode": "service-or-registration",
  "manualInstallHint": "需要 Docker：docker run -d -p 8775:8775 xxx/sqlmap-mcp"
}
```


```markdown
## 按需自举

| 能力 | 可自动安装 | 方式 | 说明 |
|------|-----------|------|------|
| sqlmap-mcp | ✗（需 Docker） | docker run | AI 会自动注册 MCP URL，但需要用户手动启动容器 |

### 手动启动
\```powershell
docker run -d -p 8775:8775 xxx/sqlmap-mcp
\```
```


---
