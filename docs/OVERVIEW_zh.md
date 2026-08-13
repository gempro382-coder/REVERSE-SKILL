

```text
用户任务
  ↓
RULES.md
  ↓
Skill Router
  ↓
目标场景 Skill
  ↓
工具 / MCP / 脚本
  ↓
报告 + field journal
```


|---|---|


|---|---|---|


```bash
bash skills/scripts/bootstrap-reverse.sh --list
```


```bash
bash skills/scripts/refresh-tool-index.sh
```


- Claude Code
- Codex CLI
- Cursor
- Cline
- Windsurf
- Kiro


|---|---|


```text
帮我分析这个 APK 的签名校验逻辑。
```


```text
.
├── README.md                    # 主入口（英文）
├── README_zh.md                 # 主入口（中文）
├── README_AI.md                 # AI Agent bootstrap 入口（英文）
├── RULES.md                     # 全局路由与执行规则
├── docs/OVERVIEW.md              # 详细概览（英文）
├── docs/OVERVIEW_zh.md           # 详细概览（中文）
├── docs/ARCHITECTURE.md          # 架构说明
├── docs/PLATFORMS.md             # 平台支持总览
├── skills/                      # 主 Skill 目录
│   ├── SKILL.md                 # 总控入口
│   ├── routing.md               # 路由矩阵
│   ├── field-journal/           # 经验沉淀
│   ├── apk-reverse/
│   ├── js-reverse/
│   ├── reverse-engineering/
│   ├── ida-reverse/
│   ├── radare2/
│   └── ...
├── CTF-Sandbox-Orchestrator/    # CTF 场景子技能库
├── burp-mcp-full/               # BurpSuite MCP 控制模块
└── kali/                        # Kali 环境辅助脚本
```


### AI Agent


## License

MIT License. See [LICENSE](../LICENSE).
