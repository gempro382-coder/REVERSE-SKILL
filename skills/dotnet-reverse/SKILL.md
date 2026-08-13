---
name: dotnet-reverse
license: MIT
allowed-tools: Bash Read Write Edit Glob Grep Task WebFetch WebSearch
metadata:
  user-invocable: "false"
---


|------|------|------|


```powershell
# Windows
file target.exe                       # "PE32 executable ... for MS Windows" 不够
# 关键：看有没有 CLR
powershell -c "[System.Reflection.AssemblyName]::GetAssemblyName('target.exe')"
# 或
dnSpyEx 直接拖进去 —— 能打开就是托管

# 通用
strings target.exe | grep -iE "mscoree|_CorExeMain|mscorlib|System\\."
```


```powershell
# DIE 快速识别
diec target.exe                        # Detect It Easy CLI
# 或拖进 dnSpyEx，看是否大量乱码类名 / 控制流变形
```


|--------|------|------------|


```powershell
# de4dot 默认自动识别大多数壳
de4dot target.exe -o target-clean.exe

# 指定类型（自动识别失败时）
de4dot --type cfze target.exe          # ConfuserEx
de4dot --type sa target.exe            # SmartAssembly

# 多层混淆 / de4dot 报 unknown
de4dot --detect target.exe             # 看它识别成什么
# 可能要先 patch anti-tamper 再 de4dot（见 references/obfuscators.md）
```


```text
定位字符串 → 反向引用 → 找到使用它的方法 → IL 视图看判断逻辑
```


```text
dnSpyEx → 右键方法 → Edit Method (C#) 或 Edit IL
  - 改判断：ldc.i4.0 → ldc.i4.1（false→true）
  - 改常量：直接编辑字符串/数字
  - 删除校验：nop 掉整段
File → Save Module → 替换原文件
```


- IL2CPP / NativeAOT（native）→ `reverse-engineering/`
