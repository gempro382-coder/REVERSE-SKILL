---
name: browser-automation
---


---


```bash
# 1. 打开页面
agent-browser open <url>

# 2. 获取可交互元素（返回 @e1, @e2... 引用）
agent-browser snapshot -i

# 3. 用引用操作元素
agent-browser click @e1
agent-browser fill @e2 "text"

# 4. 完成后关闭
agent-browser close
```


```bash
# 导航
agent-browser open <url>
agent-browser close

# 页面快照
agent-browser snapshot        # 完整无障碍树
agent-browser snapshot -i     # 仅可交互元素（推荐）

# 交互操作
agent-browser click @e1
agent-browser fill @e2 "text"
agent-browser type @e2 "text"
agent-browser press Enter
agent-browser scroll down 500

# 获取信息
agent-browser get text @e1
agent-browser get title
agent-browser get url

# 等待
agent-browser wait @e1
agent-browser wait 2000
agent-browser wait --load networkidle
```


---


```bash
# 1. Clone 项目
git clone https://github.com/zhexulong/openreverse.git
cd openreverse

# 2. 安装依赖
npm install

# 3. 接入 Agent 宿主（Claude Code / Codex / Zed）
npm run init:agents -- --target=all /path/to/project

# 4. 安装 CUA runtime（如果需要视觉驱动模式）
npm run install:cua-runtime
npm run doctor:cua-runtime

# 5. 安装网络观察依赖（如果需要抓包）
npm run install:mitmproxy
npm run doctor:network
```


```text
场景：自动化操作 IDA Pro 进行批量分析

1. 用 OpenReverse CUA 模式打开 IDA Pro
2. 自动加载目标二进制
3. 等待分析完成
4. 通过 UI 操作导出函数列表
5. 同时用 network lane 观察 IDA 的网络行为（如 Lumina 请求）
```

```text
场景：自动化操作 x64dbg 调试

1. 用 OpenReverse UIA 模式启动 x64dbg
2. 加载目标程序
3. 设置断点
4. 运行并观察寄存器/内存变化
5. 截图保存证据
```

---


```markdown
⚠️ **需要 OpenReverse 进行桌面应用自动化**

**安装步骤**：
1. `git clone https://github.com/zhexulong/openreverse.git`
2. `cd openreverse && npm install`
3. `npm run init:agents -- --target=all <你的项目路径>`
4. 如需视觉模式：`npm run install:cua-runtime`
5. 如需网络观察：`npm run install:mitmproxy`

**验证**：`npm run doctor:cua-runtime` 和 `npm run doctor:network`
```

---
