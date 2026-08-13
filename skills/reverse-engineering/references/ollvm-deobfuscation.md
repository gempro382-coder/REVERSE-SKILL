

---


|---------|---------|------|------|


---


|------|----------|----------------------|---------|


---


```
Original:             OLLVM flattened:
  block_A               entry -> dispatcher
  block_B                 ↓
  block_C              state_machine:
                         switch(state):
                           0 → block_A
                           1 → block_B
                           2 → block_C
```


```c
// 经典不透明谓词：x(x+1) 必为偶数，编译器证明不了
if (x * (x + 1) % 2 == 0) {
    // 真实逻辑
} else {
    // 不可达垃圾代码
}
```


```
a + b  →  (a ^ b) + 2*(a & b)
a ^ b  →  (a | b) - (a & b)
a - b  →  a + (~b) + 1
```


|---------|---------|------------|

---


```text
1. 下载 obpo_plugin.py 和 obpoplugin 目录
2. 复制到 IDA plugins 路径
3. 重启 IDA，打开目标二进制
4. 在 CFG 中定位分发块（dispatcher），通常长这样：
   [截图参考仓库 assets/dispatchblock.png]
5. 右键 → OBPO → Mark and process function
6. 处理完成后刷新反编译器
7. 可根据反编译变化继续标记新的分发块（迭代处理嵌套 fla）
```


|------|------|

|------------|------|------|

```text
1. clone d810-ng
2. 安装依赖（含 Z3）
3. 复制到 IDA plugins 目录
4. IDA 中按 Ctrl-Shift-D 加载插件
5. 在 GUI 中勾选要应用的规则集
6. 对目标函数应用
```


```bash
git clone https://github.com/cdong1012/ollvm-unflattener.git
cd ollvm-unflattener
pip install -r requirements.txt   # miasm, graphviz, keystone-engine

# 基本用法
python unflattener -i <input.bin> -o <output.bin> -t <function_addr> -a
# -a: 自动跟随调用做多层处理
```


> [amimo/ollvm-breaker](https://github.com/amimo/ollvm-breaker) · 441⭐


### 3.5 deollvm — ARM64 Unicorn

> [GeT1t/deollvm](https://github.com/GeT1t/deollvm) · 34⭐ · 2026-04


> [Mrack/DeObfBR](https://github.com/Mrack/DeObfBR) · 96⭐ · 2026-06-25


```python
import angr

proj = angr.Project("target.so", auto_load_libs=False)
cfg = proj.analyses.CFGFast()
func = proj.kb.functions[0x12345]

# 内置 Deobfuscator
deob = proj.analyses.Deobfuscator(func=func)
deob.normalize()
```


---


```
目标二进制
  ↓
1. 识别 OLLVM 变种（看 1.2 节线索）
  ├── 原始 OLLVM / Hikari / O-MVLL  → 标准 fla/bcf/sub
  ├── Pluto / Polaris                → 注意 Trap Angr，避开 angr
  ├── Goron / Arkari                 → 先试数据段只读，再处理 BR
  ├── Tigress                        → d810-ng Tigress unflattener
  ├── Hodur (PlugX)                  → d810-ng HodurUnflattener
  └── amice (含 VM)                  → 不是单纯 fla，需 VM handler 还原
  ↓
2. 选择工具（看第 0 节决策表）
  ├── 有 IDA + 可联网 + 非敏感样本 → obpo-plugin
  ├── 有 IDA + 本地              → d810-ng
  ├── 有 Binary Ninja            → ollvm-breaker
  ├── 无 GUI + x86/x64           → ollvm-unflattener (Miasm)
  ├── 无 GUI + ARM64             → deollvm (Unicorn) / angr
  └── 纯符号执行 / CTF           → angr
  ↓
3. 分层去混淆（顺序很重要）
  a) 先去除不透明谓词 (bcf)   → d810-ng opaque predicate removal
  b) 再去除控制流平坦化 (fla) → unflattener
  c) 最后简化 MBA (sub)       → d810-ng MBA simplifier / SiMBA
  ↓
4. 验证
  ├── 函数体积显著减小？
  ├── CFG 从星形/放射状变为链形/树形？
  └── Frida hook 关键函数验证逻辑正确？
```


```bash
adb pull /data/app/~~/lib/arm64/libnative.so
# 或从 APK 直接解压：unzip target.apk -d out/ ; find out -name "*.so"
```

```bash
readelf -a libnative.so | grep -E "Size|text"   # .text 异常大但函数少 → 大概率 OLLVM
# IDA 打开看函数特征：
#   巨大 switch → fla
#   不可达分支 → bcf
#   复杂算术 → sub/MBA
#   间接跳转 BR x8 → Goron/Arkari，试数据段只读
#   while(1) + jnz state → Hodur，用 d810-ng HodurUnflattener
```

```
a) bcf: d810-ng opaque predicate removal  (或 obpo 自动处理)
b) fla: d810-ng Unflattener / obpo-plugin / deollvm(ARM64)
c) sub: d810-ng MBA simplifier
```

```javascript
// Trace OLLVM 状态变量，辅助 deflat 确定状态变量地址
const target = Module.findBaseAddress("libnative.so");
console.log("[+] libnative.so @", target);

// 在分发器入口下 hook，观察 state 变化序列
Interceptor.attach(target.add(0x1234), {  // dispatcher offset
    onEnter(args) {
        // 读取状态变量（需根据反编译确定寄存器/栈位置）
        console.log("[state]", this.context.x8);  // 假设 state 在 x8
    }
});
```


```python
#!/usr/bin/env python3
"""CTF OLLVM quick deflat with angr"""
import angr

proj = angr.Project("challenge", auto_load_libs=False)
cfg = proj.analyses.CFGFast()

# 找最大的几个函数（最可能是被混淆的）
funcs = sorted(cfg.functions.values(), key=lambda f: f.size, reverse=True)[:5]
for func in funcs:
    print(f"[*] {func.name} @ {hex(func.addr)} size={hex(func.size)}")
    try:
        deob = proj.analyses.Deobfuscator(func=func)
        deob.normalize()
        print(f"    [+] deobfuscated")
    except Exception as e:
        print(f"    [-] failed: {e}")
        # angr 失败 → 怀疑 Trap Angr → 换 d810-ng / Unicorn
```

---


```python
# 这些等式是 OLLVM sub pass 生成表达式的化简目标
"(a | b) + (a & b)"        # → a + b
"(a | b) - (a & b)"        # → a ^ b
"(a ^ b) + 2*(a & b)"      # → a + b
"(a | b) & ~(a & b)"       # → a ^ b
"~(~a & ~b)"               # → a | b (De Morgan)
```


|------|------|------|

```python
# SiMBA 示例
from simba import simplify_mba
exprs = ["(a | b) + (a & b)", "(a ^ b) + 2*(a & b)"]
for e in exprs:
    print(f"{e}  →  {simplify_mba(e)}")
```

---


```bash
#!/bin/bash
# OLLVM deobfuscation pipeline (2026 community tools)
# 适用标准 OLLVM / Hikari / O-MVLL 加固的 ELF/.so

BINARY=$1

echo "[*] Stage 0: 基本分析与变种识别"
file $BINARY
readelf -h $BINARY 2>/dev/null | head -5
echo "    → 在 IDA 中确认变种（参考第 1 节）"

echo "[*] Stage 1: d810-ng 本地反混淆（首选）"
echo "    IDA → Ctrl-Shift-D 加载 d810-ng"
echo "    勾选: MBA + Opaque predicate + Unflattener"
echo "    Apply to target functions"
echo "    保存 IDB"

echo "[*] Stage 2: obpo-plugin（如 d810-ng 效果不足且可联网）"
echo "    IDA → 右键 dispatcher → OBPO → Mark and process"
echo "    ⚠️ 敏感样本勿用（二进制上传云服务）"

echo "[*] Stage 3: 无 IDA 备选（x86/x64）"
echo "    python unflattener -i $BINARY -o deobf.bin -t <func_addr> -a"

echo "[*] Stage 4: ARM64 .so 无 IDA 备选"
echo "    deollvm (Unicorn) 或 angr Deobfuscator"

echo "[+] Done. 在 IDA 中重新分析验证。"
```

---


|------|------|---------|

---


|------|------|------|---------|---------|------|------|

---


- [HikariObfuscator/Hikari](https://github.com/HikariObfuscator/Hikari) — Hikari
- [amimo/goron](https://github.com/amimo/goron) — goron
- [bluesadi/Pluto](https://github.com/bluesadi/Pluto) — Pluto
- [open-obfuscator/o-mvll](https://github.com/open-obfuscator/o-mvll) — O-MVLL

- [amimo/ollvm-breaker](https://github.com/amimo/ollvm-breaker) — Binary Ninja
- [GeT1t/deollvm](https://github.com/GeT1t/deollvm) — ARM64 Unicorn
