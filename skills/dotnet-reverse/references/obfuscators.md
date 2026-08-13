

|--------|-------------|---------|---------|---------|


```powershell
# 自动识别（多数情况够用）
de4dot target.exe -o target-clean.exe

# 显式指定 type（自动识别失败）
de4dot --type cfze target.exe -o target-clean.exe

# 先探测壳类型
de4dot --detect target.exe

# 批量
de4dot *.exe

# 只解字符串，不动控制流（最小干预）
de4dot --strtyp delegate --strtok METHOD_TOKEN target.exe
```


---


```powershell
# 1. 标准脱壳
de4dot target.exe -o target-clean.exe

# 2. 如果 de4dot 报 "unknown" 或脱壳后打不开 → 新版/私改 ConfuserEx
#    先确认 anti-tamper：
dnSpyEx 打开 → 找 Module .cctor 或 Main 里的完整性校验
```


```text
方法 A — dnSpyEx 直接 patch 校验函数：
  1. 找 anti-tamper 校验方法（通常在 <module> 的静态构造里调用）
  2. IL 编辑：把校验方法体改成 ret（直接返回）
  3. 保存 → 再喂给 de4dot

方法 B — 运行时 dump：
  1. 用 MegaDumper / ExtremeDumper 跑起来 dump 内存中的 assembly
  2. dump 出来的已经解密，再用 de4dot 清理残留
```


---

## SmartAssembly

```powershell
de4dot --type sa target.exe -o target-clean.exe
```


---

## .NET Reactor（necrobit）


```text
当 de4dot 失败时：
1. 让程序跑起来（dotnet target.exe 或直接双击）
2. MegaDumper / ExtremeDumper dump 进程内存 → 导出解密后的 assembly
3. 用 de4dot 清理 dump 产物的残留混淆
4. 如果 metadata 损坏，用 dnlib 重建（见 common-workflow.md）
```

---


```text
1. dnSpyEx 找到解密方法（通常签名固定：static string Decrypt(int) 或 Decrypt(string, int)）
   - 特征：被大量调用、参数是数字常量、返回 string
2. 记下方法 token（如 0x06000012）
3. de4dot 指定解密器：
   de4dot --strtyp delegate --strtok 0x06000012 target.exe -o target-clean.exe
```


|------|------|------|
