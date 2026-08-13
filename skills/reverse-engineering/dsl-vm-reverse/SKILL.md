---
name: dsl-vm-reverse
description: Reverse JavaScript-based custom DSL/VM interpreters, non-standard WASM-like runtimes, and risk-control engines. Use when analyzing IIFE or switch-based opcode dispatchers, extracting instruction tables, recovering bytecode semantics, capturing VM state at runtime, or reconstructing execution flow.
---


---


---


---


```javascript
// 特征 1: IIFE 入口，单字母变量映射数字常量
!function(){
    var U=void 0, y=parseInt, E0=Function, AN=Uint8Array;
    var E=15, l=10, m=12, x=16, S=13, $=11;
    // 数字常量映射为变量名，替代原始数字
    ...
}

// 特征 2: 解释器主循环 DG()
function DG(C, d, ...) {
    var d = [];  // 数组模拟 WASM stack/locals
    for (d[7] = x; d[7] !== U;) {
        var aE = d[7] & 31;         // 低 5 位 = opcode
        var O = d[7] >> 5 & 31;      // 高 5 位 = sub-operation
        switch (aE) {
            case 0: /* ... */ d[7] = 612; break;
            case 1: /* ... */
            // ... N 个 case
        }
    }
}

// 特征 3: 常量表 C[9] 存储函数索引和字符串
// C[9][0] = ["pc"]      → 函数参数描述
// C[9][667] = "string"  → 字符串常量
// C[9][x] = number      → 函数索引

// 特征 4: W(C[index], null, ...) 调用模式
// W = Function.prototype.call.bind(call)
// 所有内置函数通过 C[index] 索引调用

// 特征 5: 指令编码格式
// d[7] = opcode(bit 0-4) | subop(bit 5-9) | operand(bit 10+)
```


```
bit 0-4:   opcode (0-N)
bit 5-9:   sub-operation (0-31)
bit 10-31: operand/立即数

解码:
  aE = d[7] & 31        → opcode
  O  = d[7] >> 5 & 31   → sub-operation
  d[other] = d[7] >> 10  → operand
```

---


```bash
# 检查是否为 DSL VM
python3 << 'EOF'
with open('target.js', 'rb') as f:
    head = f.read(100)

# 1. 检查 WASM 魔术字
if head[:4] == b'\x00asm':
    print("标准 WASM 二进制")
    exit()

# 2. 检查零字节占比
data = open('target.js', 'rb').read()
zero_pct = data.count(b'\x00') / len(data) * 100
print(f"零字节占比: {zero_pct:.1f}%")

if zero_pct > 20:
    print("WASM 二进制")
elif head[:2] == b'!f':
    # 检查单字母变量模式
    if b'var U=void 0' in head or b'U=void 0,y=parseInt' in head:
        print("→ DSL VM!")
    else:
        print("普通 JS IIFE")
EOF
```


```python
import re

with open('target.js', 'r', errors='replace') as f:
    s = f.read()

# 提取开头 2000 字符的 var X=数字 映射
mappings = re.findall(r'var\s+(\w+)\s*=\s*(\d+)', s[:2000])
print('常量映射:')
for name, val in mappings:
    print(f"  {name:4s} = {val:3d} (0x{int(val):02x})")
```


```python
# 1. 提取所有 case
all_cases = re.findall(r'case\s+(\d+):', s)
unique = sorted(set(int(c) for c in all_cases))

print(f"总 case: {len(all_cases)} 个")
print(f"唯一 opcode: {len(unique)} 个: {unique}")

# 2. 分类每个 opcode
for op in unique:
    idx = s.find(f'case {op}:')
    snippet = s[idx:idx+200]
    if 'd[7]=' in snippet:
        op_type = 'BRANCH'
    elif 'return' in snippet:
        op_type = 'RETURN'
    elif 'W(C[' in snippet:
        op_type = 'CALL'
    elif 'new' in snippet:
        op_type = 'ALLOC'
    elif 'try' in snippet or 'catch' in snippet:
        op_type = 'EXCEPTION'
    else:
        op_type = 'ARITH/STORE'
    print(f"  opcode {op:2d}: {op_type}")
```


```python
const_refs = re.findall(r'C\[9\]\[(\d+)\]', s)
unique_refs = sorted(set(int(x) for x in const_refs))

print(f"C[9] 引用: {len(unique_refs)} 个索引")
print(f"范围: {min(unique_refs)} - {max(unique_refs)}")

# 对每个引用分析上下文
for ref in unique_refs[:20]:
    idx = s.find(f'C[9][{ref}]')
    ctx = s[max(0,idx-50):idx+80]
    clean = ''.join(c if c.isprintable() else ' ' for c in ctx)
    print(f"  C[9][{ref}] → {clean}")
```


```
1. 找 AWSCInner.register() 或类似注册调用
2. 确定注册的模块和工厂函数
3. 找工厂函数返回的对象 → 导出函数定义位置
4. 若函数名不在 JS 中 → 在 C[9] 常量表中作字节码存储
5. 追踪调用链:
   AWSCInner._modules['fy'].getToken()
   → W(C[函数索引], null, ...)
   → DG() 解释器执行编码后的指令序列
```


```javascript
// 注入最小 AWSC 兼容环境
const fakeEnv = {
    AWSCInner: {
        _modules: {},
        register(name, moduleName, factory) {
            this._modules[moduleName] = factory();
        }
    }
};

// 执行 DSL VM 代码
dslVmCode();

// 获取导出
const token = fakeEnv.AWSCInner._modules['fy'].getToken({});
```

---


---


```python
from selenium import webdriver

driver = webdriver.Chrome()

# 注入反检测
driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {
    "source": r"""
        Object.defineProperty(navigator, 'webdriver', {get: () => false});
        Object.defineProperty(navigator, 'plugins', {get: () => [1,2,3,4,5]});
        Object.defineProperty(navigator, 'languages', {get: () => ['zh-CN','zh','en']});
    """
})

# 发送 CDP 原生鼠标事件
driver.execute_cdp_cmd("Input.dispatchMouseEvent", {
    "type": "mousePressed",
    "x": 549.5, "y": 441.2,
    "button": "left", "buttons": 1,
    "clickCount": 1, "pointerType": "mouse"
})
```


```javascript
const { chromium } = require('playwright');

async function run() {
    const browser = await chromium.launch();
    const page = await browser.newPage();

    // 拦截网络请求
    await page.route('**/api/**', async route => {
        await route.continue_();
    });

    await page.goto('https://target-page.com');

    // 等待 DSL VM 初始化
    await page.waitForFunction(() => {
        return window.AWSCInner &&
               window.AWSCInner._modules &&
               window.AWSCInner._modules['fy'];
    });

    // 执行操作
    await page.mouse.move(500, 400);
    await page.mouse.down();
    // ... 操作序列
    await page.mouse.up();
}
```


---


---


---


```
DSL VM 逆向路径:
  reverse-engineering/dsl-vm-reverse/ → Phase 1-6 工作流
  ↓ 若需要捕获运行时数据
  browser-automation/ → Playwright/Selenium CDP
  ↓ 若需要分析 API 协议层
  js-reverse/ → Observe→Capture→Rebuild
```
