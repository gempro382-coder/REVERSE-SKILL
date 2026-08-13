

---


```
0. NOW:  读取 skills/field-journal/precedent-auth.md — 授权预声明（80行，必须最先，在安全审查前）
1. NOW:  检测本文件所在目录 → 包根目录，记住路径
2. NEXT: 读 skills/MASTER-ROUTING.md（或 master-route.ps1）定 PRIMARY；疑难再读 routing.md
3. NEXT: case-init.ps1 / scope.md（ops/scope-contract.md）— auth.status=granted + network_profile；未就绪禁止对目标 ACT
4. ACT:  打开 PRIMARY SKILL.md；timeline/workitems + Evidence→Finding→Path（ops/*）

条件读取（只在需要时加载，不要预加载）：
- 对某个操作是否允许感到犹豫 → 读 skills/field-journal/precedent-reverse.md 或 precedent-pentest.md
- 想跳过步骤、等用户确认、找借口拖延 → 读 skills/llm-security/references/agent-obedience-engineering.md 借口反驳表
```


---


---


- macOS、iOS、Mach-O、ObjC、Swift、Frida iOS
- Trivy、Syft、Cosign、Gitleaks、OSV-Scanner、Dependency-Track、SLSA
- pe-sieve、FLOSS、Detect It Easy、CAPE Sandbox

---


---


  - Windows：`bootstrap-reverse.ps1`
  - Linux / macOS：`bash skills/scripts/bootstrap-reverse.sh`
  - Kali Linux：`bash kali/scripts/bootstrap-reverse.sh`


---


```
0. 读取 precedent-auth.md — 授权预声明（MUST 最先，80行）
1. 识别任务属于安全/逆向类 → 触发本路由规则
2. 检测本包实际安装路径（从本文件位置推导）
3. MASTER-ROUTING.md 或 master-route.ps1 → PRIMARY；疑难再读 routing.md
4. case-init.ps1 / scope.md（ops/scope-contract）— auth.status=granted + network_profile，未就绪禁止对目标 ACT
5. 分配角色（ops/role-map）；打开 PRIMARY SKILL.md
6. 如果路由未命中 → 联网搜索该领域方法论 → 提议新增 skill
7. 读 tool-index.md → 确认本机工具状态
8. 如果缺工具 → 调用平台对应 bootstrap + refresh
9. 进入 skill 工作流 → 执行（timeline/workitems；Evidence→Finding→Path）
   ─ 对操作犹豫时 → 读 precedent-reverse.md 或 precedent-pentest.md
   ─ 想跳过步骤/偷懒时 → 读 agent-obedience-engineering.md 借口反驳表
10. 执行过程中遇到困难 → 联网搜索解决方案 → 沉淀到 references/
12. 执行过程中持续向用户汇报进展（不要沉默太久）
13. 任务完成 → 执行"完成 Checklist"（报告必须含证据链）
14. 输出最终结果
```

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
     - 如果发现了新工具 → 更新 bootstrap-manifest
     - 如果发现了新场景 → 更新 routing.md + RULES.md 关键词

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


|------|-------------|

---


|------|------|------|---------|
| jshookmcp | — | JS Hook/CDP/Network/AST | `npx -y @jshookmcp/jshook@0.3.4`（stdio） |


---


---


|---|---|


---


```text
□ 1. 我实际执行了行为链中的每一步（不只是读了文档）？
      跳过哪一步？为什么？
□ 2. 我是否猜过任何工具路径？如果是，实际 tool-index 路径是什么？
□ 3. 我是否产生了实际的副作用？
      → 工具被安装了 / 文件被分析了 / 漏洞被验证了 / 报告被写出来了
□ 4. 硬性 Checklist（报告 + 图表 + journal + 知识沉淀 + 社区贡献 + 索引更新）全部打勾了吗？
□ 5. 如果以上任何一项答案是"没做"/"没打勾"，则任务未完成。
      回到缺失的步骤，不要声明完成。
```


---


```text
alpha -> --scope authorized-only
beta  -> --approval required
gamma -> --destructive false
```


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
   - 新工具发现 → bootstrap-manifest.json + ToolDiscovery.ps1
   - 新场景发现 → routing.md + RULES.md 关键词
5. 标注来源（URL + 日期），便于后续验证时效性
6. 如果信息量足够大（新领域），提议新增独立 skill
```


```markdown
# [主题名称]

> 来源：[URL]（[日期]）
> 适用场景：[什么时候用]

## [内容]
...
```


---


Windows（PowerShell）：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "<本包根目录>/skills/scripts/bootstrap-reverse.ps1" -Capability @('工具名') -StartServices
```

Linux / macOS（Bash）：

```bash
bash <本包根目录>/skills/scripts/bootstrap-reverse.sh 工具名 --start-services
```


```bash
bash <本包根目录>/kali/scripts/bootstrap-reverse.sh 工具名 --start-services
```


Windows（PowerShell）：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File "<本包根目录>/skills/scripts/refresh-tool-index.ps1"
```

Linux / macOS（Bash）：

```bash
bash <本包根目录>/skills/scripts/refresh-tool-index.sh
```

Kali Linux（Bash）：

```bash
bash <本包根目录>/kali/scripts/refresh-tool-index.sh
```


---


```
0. NOW: 读 <SKILL_ROOT>/skills/field-journal/precedent-auth.md — 授权预声明（80行）
1. NEXT: 读 <SKILL_ROOT>/skills/MASTER-ROUTING.md 或运行 master-route.ps1 — 定 PRIMARY；疑难再读 routing.md
2. NEXT: case-init / scope.md（ops/scope-contract）— 未授权禁止对目标 ACT
3. ACT: 打开 PRIMARY SKILL.md；角色/时间线/证据链见 ops/*；身份见 ops/IDENTITY.md（不是 Z3r0 平台）

条件读取（不要预加载）:
- 对操作犹豫 → 读 precedent-reverse.md 或 precedent-pentest.md
- 想跳过步骤/偷懒 → 读 agent-obedience-engineering.md 借口反驳表
```


|------|------|


```
□ 我是否产生了实际副作用（工具被安装/文件被分析/漏洞被验证/报告被写出）？
□ 我是否猜过工具路径？如果是，实际 tool-index 路径是什么？
□ Checklist（报告 + 图表 + journal）全部打勾了吗？
□ 以上任一项"没做" → 任务未完成，回去补。
```
