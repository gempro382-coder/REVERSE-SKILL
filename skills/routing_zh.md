

|---------|---------|---------|
| macOS / iOS | `reverse-engineering/platforms.md` — Mach-O/ObjC/Swift | — |


|--------|---------|
| "Patch Tuesday/MSRC/Microsoft Update Catalog" | `patch-diff-exploit/references/patch-tuesday-workflow.md` |
| "pwntools/GEF/pwndbg/one_gadget/libc-database" | `pwn-chain/SKILL.md` |
| "binwalk/unblob/SquashFS/UBI/JFFS2" | `firmware-pentest/references/extraction-methodology.md` |
| "direct syscall/indirect syscall/Hell's Gate/SysWhispers" | `edr-bypass-re/references/unhook-techniques.md` |
| "ETW patch/AMSI patch/telemetry blinding" | `edr-bypass-re/references/telemetry-blinding.md` |

|------|---------|
| radare2 (r2/rabin2/rasm2) | `radare2/` — CLI + recon.ps1 |
| jadx / apktool | `apk-reverse/` — decode.ps1 / manifest-summary.ps1 |
| Frida | `reverse-engineering/tools-dynamic.md` |
| angr / Qiling / Unicorn | `reverse-engineering/tools-dynamic.md` |
| BinDiff / Diaphora | `reverse-engineering/tools-advanced.md` |


---


```
APK 逆向路径：
  apk-reverse/scripts/decode.ps1 → Java 层分析
  ↓ 如果核心在 .so
  ida-reverse/ 或 radare2/ → so 分析
  ↓ 如果需动态验证
  apk-reverse/scripts/frida-run.ps1 → Frida Hook

前端 JS 逆向路径：
  js-reverse/Observe → 定位目标请求
  ↓ 需要更强的浏览器/CDP/Hook/Network 面
  jshookmcp → 做页面运行时采样、断点、拦截、SourceMap/AST 辅助
  ↓ 确认入口函数后
  js-reverse/Rebuild → Node 本地复现
  ↓ 需要补环境
  js-reverse/references/env-patching.md

二进制逆向路径：
  radare2/scripts/recon.ps1 → 快速侦察
  ↓ 深度分析
  ida-reverse/ → IDA 反编译
  ↓ 动态验证
  reverse-engineering/tools-dynamic.md → Frida/GDB

CTF 竞赛路径（通过 CTF-Sandbox-Orchestrator）：
  ctf-sandbox-orchestrator/SKILL.md → 建立沙盒模型
  ↓ 按主导证据面路由
  competition-web-runtime/ 或 competition-reverse-pwn/ 或 competition-identity-windows/
  ↓ 走不通时回总控
  ctf-sandbox-orchestrator → 重新路由

Cookie HMAC 密钥复用 → 后台认证绕过：
  competition-web-runtime/references/cookie-hmac-key-reuse-auth-bypass.md
  ↓ 适用场景
  URL 含 access token、签名 Cookie、后台 admin_session 共用同一密钥

固件渗透路径：
  firmware-pentest/references/extraction-methodology.md → 提取文件系统
  ↓ 拿到二进制
  firmware-pentest/references/emba-automated-analysis.md → EMBA 自动审计找已知 CVE
  ↓ 已知 CVE 不够 / 想找 0-day
  firmware-pentest/references/emulation-and-fuzz.md → Firmadyne 仿真 + AFL++ fuzz
  ↓ 找到 crash
  pwn-chain/references/stack-pwn.md 或 heap-pwn.md → 写 exploit
  ↓ 打实机
  attack-chain/SKILL.md → 整合进攻击链

N-day 武器化路径：
  patch-diff-exploit/references/patch-tuesday-workflow.md → 取补丁前后二进制
  ↓ 对齐符号
  patch-diff-exploit/references/diff-tools-comparison.md → BinDiff/ghidriff/Diaphora 选型
  ↓ 定位变更
  patch-diff-exploit/references/root-cause-and-poc.md → LLM 辅助根因 + 写 PoC
  ↓ 武器化
  pwn-chain/SKILL.md（构造稳定 exploit）+ pentest-tools/references/msf-protocol.md（Metasploit 模块化）

红队投递路径：
  attack-chain/SKILL.md → 选阶段
  ↓ 需要绕 EDR
  edr-bypass-re/references/hook-survey.md → 识别目标 EDR 的 hook
  ↓ 选绕过技术
  edr-bypass-re/references/unhook-techniques.md → 直接 syscall / Hell's Gate
  edr-bypass-re/references/telemetry-blinding.md → ETW patch / AMSI patch
  ↓ 本地验证
  pe-sieve / API Monitor → 确认 unhook 干净
  ↓ 投递
  回到 attack-chain 后渗透阶段
```
