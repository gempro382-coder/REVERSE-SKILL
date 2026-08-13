

```text
1. Identify  → 确认是 .NET 托管程序（不是 native）
2. Detect    → DIE / de4dot --detect 识别混淆器
3. Deobf     → de4dot 脱混淆（保留原样本）
4. Static    → dnSpyEx 浏览 C# 视图定位，IL 视图看关键逻辑
5. Dynamic   → dnSpyEx 调试器在关键方法下断，看运行时明文
6. Patch     → IL 编辑器修改，Save Module
```


```text
改判断（if (check) → 永远 true）：
  原: call bool Foo::Check()
      brfalse.s SKIP
  改: ldc.i4.1            ; push true
      brfalse.s SKIP      ; 现在永远不跳，SKIP 不执行
  或更直接：
      ldc.i4.1
      ret                 ; 方法直接返回 true

改判断（if (check) → 永远 false）：
  ldc.i4.0
  ret

删整段校验：
  全部 nop，或改成 ret + 正确返回值

改字符串常量：
  C# 编辑器改字符串通常 OK（ldstr 直接换 token），但若字符串在资源/加密里则要改解密逻辑

改数字常量：
  ldarg / ldc 指令直接改操作数
```


```text
async/await 的 MoveNext 结构：
  switch(this.<>1__state) {
    case 0: ... await 前的逻辑; this.<>1__state = 1; await MoveNext;
    case 1: ... await 后的逻辑;
  }

要 patch async 逻辑：改 MoveNext 里的 state 转移或具体 case 里的判断。
C# 编辑器改 async 几乎必失败 → 必须用 IL。
```


```csharp
// dnlib 脚本：扫描所有字符串解密器调用，运行时还原后写回
// 用法：dotnet script decrypt.csproj target.exe 0x06000012
using System;
using System.Reflection;
using dnlib.DotNet;
using dnlib.DotNet.Writer;
using dnlib.DotNet.Emit;

var module = ModuleDefMD.Load(args[0]);
var decryptorToken = uint.Parse(args[1], System.Globalization.NumberStyles.HexNumber);

// 找到解密方法，用反射调用它（需把 assembly 加载进 AppDomain）
// 遍历所有方法，把 call Decryptor(token) 替换成 ldstr "解密结果"
foreach (var type in module.GetTypes())
    foreach (var method in type.Methods)
    {
        if (!method.HasBody) continue;
        var instrs = method.Body.Instructions;
        for (int i = 0; i < instrs.Count; i++)
        {
            // 识别 call 解密器模式，调用解密器拿明文，替换为 ldstr
            // （此处省略反射调用解密器的样板，思路：加载原 assembly →
            //   MethodInfo.Invoke 拿明文 → instrs[i] = OpCodes.Ldstr + operand=明文）
        }
    }

var opts = new ModuleWriterOptions(module);
module.Write("target-decrypted.exe", opts);
```


```text
try { throw new CustomException(0x42); }
catch (CustomException e) {
    switch(e.Code) {
        case 0x42: 真实逻辑A; break;
        case 0x43: 真实逻辑B; break;
    }
}
```


```text
1. 先看 <module>.cctor（Module .cctor）—— 解密/反调试初始化
2. 再看 Program.Main / Startup
3. anti-tamper 在 .cctor 里 → 先 patch .cctor 再脱壳
```


```text
定位流程：
1. strings 看有无明文 URL/IP（混淆后通常没有）
2. 找 byte[] 字段 + 解密方法（AES/XOR）
3. 动态断在解密方法的返回点，dump 解密后的明文
4. 常见：AES-256-CBC with Key==IV（Codegate 2013 模式，见 reverse-engineering/tools.md .NET 段）
```
