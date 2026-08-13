

---


|--------|-------------|---------|


---


- macOS、iOS、Mach-O、ObjC、Swift、Frida iOS

---


---


---


```
1. 识别任务属于安全/逆向类 → 触发本路由规则
2. 检测本包实际安装路径（从本文件位置推导）
3. 首次使用 → 将规则写入当前客户端的全局配置
4. 如果 tool-index 不存在或过期 → 先执行 refresh-tool-index.sh
5. 读取 SKILL.md → routing.md → 确定进入哪个子 skill
6. 如果路由未命中 → 联网搜索 → 提议新增 skill
7. 检查 field-journal/_index.md → 是否有同类经验可复用
8. 读取 tool-index.md → 确认本机工具状态
9. 如果缺工具 → 调用 bootstrap-reverse.sh 自动补齐
10. 如果自动补齐失败 → 输出结构化引导，等用户确认后继续
11. 进入对应 skill 的工作流 → 执行任务
12. 任务完成 → 执行"完成 Checklist"
13. 输出最终结果
```

---


```bash
bash "<本包根目录>/kali/scripts/bootstrap-reverse.sh" <capability1> [capability2] ... [--start-services]
```


```bash
# 一键配齐 Kali 原生 MCP（推荐首次使用时执行）
bash kali/scripts/bootstrap-reverse.sh mcp-kali-server metasploitmcp hexstrike-ai

# 安装 2026.1 全部新工具
bash kali/scripts/bootstrap-reverse.sh adaptixc2 atomic-operator sstimap xsstrike wpprobe fluxion gef

# AD/内网渗透工具链
bash kali/scripts/bootstrap-reverse.sh coercer evil-winrm-py netexec responder bloodhound certipy

# 逆向分析工具链
bash kali/scripts/bootstrap-reverse.sh jadx frida gef ghidra-mcp

# Web 渗透工具链
bash kali/scripts/bootstrap-reverse.sh sstimap xsstrike wpprobe nuclei
```


```bash
bash "<本包根目录>/kali/scripts/refresh-tool-index.sh"
```

---


|------|------|------|------|---------|


|------|------|------|---------|
| jshookmcp | — | JS Hook/CDP/Network/AST | `npx -y @jshookmcp/jshook@0.3.4`（stdio） |


```bash
bash kali/scripts/bootstrap-reverse.sh mcp-kali-server metasploitmcp hexstrike-ai pentestswarm
```

---


|------|-------------|

---


---


---


```text
□ 1. 生成正式报告（docs-generator skill）
     - 使用对应模板（逆向报告/渗透报告/CTF writeup/签名报告）
     - 必须包含：目标概述、完整步骤、关键证据、复现命令
     - 输出到用户项目目录（不是 skill 包内）

□ 2. 生成图表（diagram-generator skill）
     - 至少 1 张流程图嵌入报告
     - 类型选择：渗透→攻击路径图 / 逆向→调用关系图 / JS→时序图 / CTF→解题流程

□ 3. 回写 field-journal（已脱敏）
     - 按 field-journal/_template.md 格式
     - 必须包含：踩坑记录、可复用模式、工具链发现、环境信息
     - 脱敏检查：无真实域名/IP/Token/用户名

□ 4. 沉淀搜索到的知识（如果本次任务中联网搜索过）
     - 将搜索到的有价值内容写入对应 skill 的 references/
     - 标注来源 URL 和日期
     - 如果发现了新工具 → 更新 bootstrap-manifest.json
     - 如果发现了新场景 → 更新 routing.md + RULES-kali.md 关键词

□ 5. 询问社区贡献
     - "是否将本次经验贡献到社区主仓库？数据已脱敏，只提交 field-journal 文件。"
     - 用户同意 → 按 CONTRIBUTE-BACK.md 流程创建 PR
     - 用户拒绝 → 跳过

□ 6. 更新系统索引
     - 更新 field-journal/_index.md（新增条目）
     - 检查是否需要更新：routing.md / bootstrap-manifest / tool-index
     - 如果发现新工具或新场景 → 执行对应更新
```


---


---


|------|---------|-------------|


```text
1. 搜索获取信息
2. 验证信息可靠性（优先官方文档 > GitHub > 博客 > 论坛）
3. 提取可操作的内容（命令/脚本/配置/步骤）
4. 写入本包对应位置：
   - 通用方法论 → 对应 skill 的 references/*.md
   - 特定工具用法 → 对应 skill 的 references/ 或 SKILL.md
   - 踩坑经验 → field-journal/
   - 新工具发现 → kali/scripts/bootstrap-manifest.json + tool-discovery.sh
   - 新场景发现 → routing.md + RULES-kali.md 关键词
5. 标注来源（URL + 日期），便于后续验证时效性
6. 如果信息量足够大（新领域），提议新增独立 skill
```


---
