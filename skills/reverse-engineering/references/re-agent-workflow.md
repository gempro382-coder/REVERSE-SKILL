

```text
□ scope.md：offline 样本路径 或 授权设备/靶机
□ tool-index：file/strings/r2/ida/frida 等实际路径
□ 角色：cre（ops/role-map）
```


```text
□ file / DIE / 熵 / 壳特征
□ strings / rabin2 -z 捡漏
□ 架构/链接/是否 .NET/Go/Rust/加壳
□ 产出：E-triage + 假设清单（勿过早下结论）
```

## 2. Static

|------|------|
| jadx / dnSpy | Android / .NET |

```text
□ 定位关键函数（加密/校验/网络/授权）
□ 记录地址/符号 → Evidence
□ 一条路不通 → 换工具（IDA↔r2↔Ghidra）
```


## 3. Dynamic

```text
□ Frida / gdb / emulator：验证静态假设
□ 反调试/反 Frida → reverse-engineering/anti-analysis
□ Android：root 检测 / SSL pinning 绕过脚本按需生成，**须在授权设备**
□ 崩溃日志驱动下一轮 hook（自适应循环）
```

## 4. Synthesis

```text
□ Finding：算法/校验逻辑/可利用点
□ Path：callflow 或 solve 步骤挂 E-*
□ 报告 docs-generator + 可选图
□ field-journal 脱敏
```
