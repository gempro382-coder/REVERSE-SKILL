

```powershell
$edrSigs = @{
    'CSAgent'           = 'CrowdStrike Falcon'
    'SentinelAgent'     = 'SentinelOne'
    'elastic-endpoint'  = 'Elastic Defend'
    'ekrn'              = 'ESET'
    'MsMpEng'           = 'Microsoft Defender'
    'SophosFileScanner' = 'Sophos Intercept X'
    'avp'               = 'Kaspersky'
    'TmListen'          = 'Trend Micro Apex One'
    'cb'                = 'Carbon Black'
}

Get-Process | ForEach-Object {
    foreach ($k in $edrSigs.Keys) {
        if ($_.ProcessName -match $k) {
            "[+] $($edrSigs[$k]) detected: $($_.ProcessName) (PID $($_.Id))"
        }
    }
}

Get-ChildItem 'C:\Windows\System32\drivers\*.sys' |
    Where-Object { $_.Name -match 'CSAgent|Sentinel|elastic|eam|WdFilter|Sophos|klif|tmcomm|Parity' } |
    Select-Object Name, VersionInfo
```


```powershell
# 简单：把磁盘 ntdll 和当前进程的 ntdll 反汇编 diff
# 1. 拿磁盘 ntdll
copy C:\Windows\System32\ntdll.dll C:\temp\ntdll_clean.dll

# 2. 在 windbg 中 attach 任意进程，导出当前 ntdll 的 .text 段
# .writemem c:\temp\ntdll_live.bin ntdll!.text L?<size>

# 3. 用 IDA / radare2 反汇编 NtAllocateVirtualMemory，正常应该是：
#    mov r10, rcx
#    mov eax, <SSN>
#    test byte ptr [...]
#    jne ...
#    syscall
#    ret
# 如果第一条变成 jmp <某地址>，那就是 hook
```


```text
0: kd> dx -r1 nt!PspCreateProcessNotifyRoutine
0: kd> dx -r1 nt!PspCreateThreadNotifyRoutine
0: kd> dx -r1 nt!PspLoadImageNotifyRoutine

0: kd> !object \Callback
0: kd> !object \Callback\ProcessObject
```


```text
1. 找一个已被 EDR 注入用户态组件的进程（任意已存活进程）
2. windbg attach (-pn target.exe)
3. lm m ntdll  → 拿到模块基址
4. .writemem c:\temp\ntdll_live.bin ntdll+0x0 L?<image size>
5. 把 C:\Windows\System32\ntdll.dll 复制为 c:\temp\ntdll_disk.dll
6. 在 IDA 里加载两个文件，跳到 NtAllocateVirtualMemory：
     - disk：标准 prologue
     - live：第一条 jmp <0x7FFE000000xx>
7. 跟着 jmp 目标地址 → 那就是 EDR 的 trampoline，dump 出来
8. 进 trampoline 看它最终落到哪个 DLL，确认 EDR 模块名
```


```powershell
# pseudo workflow，详见 references 提到的脚本
$disk = Get-Content C:\Windows\System32\ntdll.dll -Encoding Byte
$live = # 通过 OpenProcess + ReadProcessMemory 拿
# 对比 .text 段每个 export 的前 16 字节
```


```powershell
# 基本扫描
pe-sieve64.exe /pid 1234

# 推荐组合（含 shellcode 与 hook 检测）
pe-sieve64.exe /pid 1234 /shellc 3 /modules 3 /imp 3 /data 3 /dir hooks_dump

# 关键参数：
#   /shellc N    shellcode 扫描等级 (0-3)
#   /modules N   模块完整性检查 (0-3)
#   /imp N       IAT hook 检查
#   /data N      数据段扫描
#   /dir <path>  dump 输出目录
```


```text
modified_modules.tag 示例：
71f10000;ntdll.dll
71f1a3b0;hook;jmp_far
71f1c020;hook;jmp_near
```


```text
1. 启动 API Monitor v2（管理员）
2. API Filter 勾选：
     - NT Native API → Memory Management
     - NT Native API → Process and Thread
     - Windows Defender / AMSI（如果可见）
3. Monitor New Process → 选择 implant 测试样本
4. 观察：
     - NtAllocateVirtualMemory 调用顺序
     - 是否被 EDR DLL 中转
5. 在 Modules tab 看哪些 EDR DLL 被 LoadLibrary 注入
```
