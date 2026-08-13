---
name: radare2
description: |
  Use this skill whenever the user wants to analyze binaries with radare2/r2 from the command line, including reverse engineering, disassembly, function analysis, strings/import inspection, patching, binary diffing, hex inspection, or r2 scripting. Also use it when the user mentions PE/ELF/Mach-O/DEX/WASM files together with CLI analysis, `rabin2`, `rasm2`, `radiff2`, `r2pipe`, or asks for radare2 command help on Windows/Linux/macOS.
---

# radare2


```powershell
r2 -v
rabin2 -v
```


- `radare2.exe`
- `rabin2.exe`
- `rasm2.exe`
- `radiff2.exe`
- `rahash2.exe`
- `rax2.exe`
- `r2pm.exe`


### `scripts/recon.ps1`


```powershell
powershell -File "<skill-root>\radare2\scripts\recon.ps1" -TargetPath "C:\path\to\sample.exe"
```


```powershell
powershell -File "<skill-root>\radare2\scripts\recon.ps1" -TargetPath "C:\path\to\sample.exe" -RunAnalysis
```

### `references/cheatsheet.md`


```text
ERROR: Cannot find ...\share\format\dll\*.sdb
```


```powershell
powershell -File "<skill-root>\radare2\scripts\recon.ps1" -TargetPath "sample.exe"
```


```powershell
rabin2 -I sample.exe
rabin2 -z sample.exe
rabin2 -i sample.exe
rabin2 -E sample.exe
```


```powershell
r2 sample.exe
```


```text
aaa          # 常规自动分析
afl          # 列出函数
iz           # 列出字符串
iS           # 列节区
is           # 列符号
s entry0     # 跳到入口点
pdf          # 反汇编当前函数
VV           # 进入可视化模式（如果终端适合）
q            # 退出
```


```text
afl~main
afl~sym.
iz~http
iz~error
axt <addr>
```


```text
px 64        # 当前地址起 64 字节十六进制
pd 20        # 反汇编 20 条指令
psz          # 读取当前地址字符串
pxa          # 更友好的十六进制视图
```


```powershell
r2 -w sample.exe
```


```text
s 0x401000
wa nop
wa jmp 0x401050
wq
```


```powershell
r2 -A -q -c "afl;iz;ii;q" sample.exe
```


### `rabin2`


```powershell
rabin2 -I sample.exe   # 基本信息
rabin2 -S sample.exe   # 节区
rabin2 -s sample.exe   # 符号
rabin2 -i sample.exe   # 导入
rabin2 -E sample.exe   # 导出
rabin2 -z sample.exe   # 字符串
rabin2 -zz sample.exe  # 更详细字符串
```

### `rasm2`


```powershell
rasm2 -d "9090"
rasm2 -a x86 -b 64 "xor eax, eax"
```

### `radiff2`


```powershell
radiff2 old.exe new.exe
radiff2 -C old.exe new.exe
```

### `rahash2`


```powershell
rahash2 -a md5 sample.exe
rahash2 -a sha256 sample.exe
```

### `rax2`


```powershell
rax2 0x401000
rax2 4198400
rax2 -s hello
```


---


---


|------|-----------|---------|------|
