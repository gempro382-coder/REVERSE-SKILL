

---


```text
1. 检测包根目录（含 skills/ 与 kali/ 的仓库根）
2. 读取 kali/RULES-kali.md → 全局注入与工具扫描
3. bash kali/scripts/refresh-tool-index.sh
4. 与主包共用作战链：
   - skills/MASTER-ROUTING.md（或 pwsh skills/scripts/master-route.ps1）
   - skills/scripts/case-init.ps1 → work/<case>/scope.md
   - auth.status=granted + network_profile 后才对目标 ACT
   - skills/ops/（证据链 / 角色 / 时间线 / IDENTITY）
5. 向用户报告配置结果
```


---


```text
项目根目录/
├── skills/                    # 共享：SKILL、routing、MASTER-ROUTING、ops、scripts、field-journal
├── CTF-Sandbox-Orchestrator/  # 共享：40+ CTF 子技能
├── kali/                      # ← 你在这里
│   ├── scripts/
│   │   ├── bootstrap-reverse.sh
│   │   ├── refresh-tool-index.sh
│   │   ├── bootstrap-manifest.json
│   │   └── lib/
│   │       └── tool-discovery.sh
│   ├── RULES-kali.md
│   └── README-kali.md
├── RULES.md                   # Windows 版规则
└── Readme.md                  # Windows 版说明
```


- Windows：`skills/scripts/bootstrap-reverse.ps1`
- Kali：`kali/scripts/bootstrap-reverse.sh`


- `docs-generator/`、`diagram-generator/`


---


|------|----------|------|


|------|------|------|


|------|------|------|


|------|------|------|------|


---


```bash
# 全新 Kali 2026.1 系统一键配置（需要 root）
sudo bash kali/scripts/quick-setup.sh

# 跳过系统更新（网络慢时）
sudo bash kali/scripts/quick-setup.sh --skip-update

# 最小安装（不装 AD/内网工具）
sudo bash kali/scripts/quick-setup.sh --minimal
```


```bash
# 1. 进入项目根目录
cd /path/to/cybersecurity-skills-router

# 2. 给脚本加执行权限
chmod +x kali/scripts/*.sh kali/scripts/lib/*.sh

# 3. 刷新工具索引（检测本机工具状态）
bash kali/scripts/refresh-tool-index.sh

# 4. 查看结果
cat skills/tool-index.md
```


```bash
# 安装 Kali 官方 MCP 三件套
bash kali/scripts/bootstrap-reverse.sh mcp-kali-server metasploitmcp hexstrike-ai

# 安装后 MCP 配置自动写入 ~/.claude/mcp.json
# 如果用 Kiro，手动复制到 ~/.kiro/settings/mcp.json
```


```bash
# 全部新工具一键安装
bash kali/scripts/bootstrap-reverse.sh adaptixc2 atomic-operator sstimap xsstrike wpprobe fluxion gef

# AD/内网渗透套件
bash kali/scripts/bootstrap-reverse.sh coercer evil-winrm-py netexec responder bloodhound certipy
```


```bash
# 安装单个工具
bash kali/scripts/bootstrap-reverse.sh jadx

# 安装多个工具
bash kali/scripts/bootstrap-reverse.sh jadx apktool frida jshookmcp

# 安装并启动服务
bash kali/scripts/bootstrap-reverse.sh idapro --start-services
```


---


|------|----------|
| Android SDK | `~/Android/Sdk/` |
| Node.js | `/usr/bin/node`（apt/nvm） |

---


|------|-----------|---------|

---


```bash
# ─── 基础命令 ───
java -version
python3 --version
pip3 --version
node -v
npx -v

# ─── 逆向工具 ───
jadx --version
apktool --version
adb version
frida --version
r2 -v
gdb --version          # GEF 自动加载

# ─── 渗透工具（Kali 预装） ───
nmap --version
sqlmap --version
hashcat --version
hydra -h | head -1
msfconsole --version
gobuster version
ffuf -V
nuclei -version

# ─── Kali 2026.1 新工具 ───
sstimap -h 2>&1 | head -3
xsstrike -h 2>&1 | head -3
wpprobe --help 2>&1 | head -3
coercer -h 2>&1 | head -3
evil-winrm-py -h 2>&1 | head -3

# ─── AD/内网工具 ───
netexec --help 2>&1 | head -3
responder -h 2>&1 | head -3
certipy --version 2>&1 | head -1

# ─── Kali 原生 MCP ───
which kali-server-mcp && echo "mcp-kali-server OK"
which metasploitmcp && echo "metasploitmcp OK"
which hexstrike-ai && echo "hexstrike-ai OK"

# ─── 刷新工具索引 ───
bash kali/scripts/refresh-tool-index.sh

# ─── 检查 MCP 服务（如果已配置） ───
nc -z 127.0.0.1 5000 && echo "mcp-kali-server OK" || echo "mcp-kali-server offline"
nc -z 127.0.0.1 8085 && echo "metasploitmcp OK" || echo "metasploitmcp offline"
nc -z 127.0.0.1 13337 && echo "IDA MCP OK" || echo "IDA MCP offline"
nc -z 127.0.0.1 23816 && echo "anything-analyzer OK" || echo "anything-analyzer offline"
```

---


```bash
# 用官方源安装最新版
bash kali/scripts/bootstrap-reverse.sh r2
# Kali 版默认优先 apt 安装/补齐 radare2；如需最新版可按平台文档改用 GitHub/source
```
