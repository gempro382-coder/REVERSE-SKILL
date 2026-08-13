---
name: binary-diff
description: |
---


|------|--------|


|------|--------------|------|--------|


```text
旧版函数（有符号）          新版同一函数（无符号）
    ↓                              ↓
导出反汇编 + 伪代码          导出反汇编 + 伪代码
    ↓                              ↓
    └──────── LLM 结构化比对 ────────┘
                    ↓
         输出 YAML（符号映射表）
                    ↓
         程序化解析 → 批量应用到新版 IDB
```


```text
I have disassembly outputs and procedure code of the same function.

This is the function for reference:

**Disassembly for Reference**
```c
{disasm_for_reference}
```

**Procedure code for Reference**
```c
{procedure_for_reference}
```

This is the function you need to reverse-engineering:

**Disassembly to reverse-engineering**
```c
{disasm_code}
```

**Procedure code to reverse-engineering**
```c
{procedure}
```

What you need to do is to collect all references to "{symbol_name_list}" in the function you need to reverse-engineering and output those references as YAML.

Example:
```yaml
found_vcall: # This is for indirect call to virtual function or virtual function pointer fetching.
  - insn_va: '0x180777700' # Always be the instruction with displacement offset
    insn_disasm: call [rax+68h] # Always be the instruction with displacement offset
    vfunc_offset: '0x68'
    func_name: ILoopMode_OnLoopActivate
  - insn_va: '0x180777778' # Always be the instruction with displacement offset
    insn_disasm: mov rax, [rax+80h] # Always be the instruction with displacement offset
    vfunc_offset: '0x80'
    func_name: INetworkMessages_GetNetworkGroupCount

found_call: # This is for direct call to non-virtual regular function.
  - insn_va: '0x180888800'
    insn_disasm: call sub_180999900
    func_name: CLoopMode_RegisterEventMapInternal
  - insn_va: '0x180888880'
    insn_disasm: call sub_180555500
    func_name: CLoopMode_SetSystemState

found_funcptr: # This is for non-virtual regular function pointer.
  - insn_va: '0x180666600' # Must load/reference the function pointer target address
    insn_disasm: lea rdx, sub_15BC910 # Must load/reference the function pointer target address
    funcptr_name: CLoopMode_OnClientPollNetworking

found_gv: # This is for reference to global variable.
  - insn_va: '0x180444400'
    insn_disasm: mov rcx, cs:qword_180666600 # Must load/reference the global variable
    gv_name: g_pNetworkMessages
  - insn_va: '0x180333300'
    insn_disasm: lea rax, unk_180222200 # Must load/reference the global variable
    gv_name: s_EventManager

found_struct_offset: # This is for reference to struct offset. NOTE THAT virtual function pointer should not be here! virtual function pointer should ALWAYS be in found_vcall !
  - insn_va: '0x1801BA12A' # Always be the instruction with displacement offset
    insn_disasm: mov rcx, [r14+58h] # Always be the instruction with displacement offset
    offset: '0x58'
    size: 8
    struct_name: CResourceService
    member_name: m_pEntitySystem
```

If nothing found, output an empty YAML. DO NOT output anything other than the desired YAML. DO NOT collect unrelated symbols.
```


|------|------|------|


```text
Step 1: 准备数据
  - 旧版二进制加载到 IDA（有 PDB/符号）
  - 新版二进制加载到 IDA（无符号）
  - 找到两个版本中相同的锚点函数（导出函数、字符串引用等）

Step 2: 批量导出
  - 从旧版导出：锚点函数的反汇编 + 伪代码（含符号名）
  - 从新版导出：同一锚点函数的反汇编 + 伪代码（无符号名）

Step 3: LLM 比对
  - 用 prompt 模板填充数据
  - 调用 LLM API（推荐：deepseek 量大便宜，超大函数切 gpt）
  - 解析返回的 YAML

Step 4: 应用结果
  - 将 YAML 中的符号映射批量应用到新版 IDB
  - 用 idapro_rename 或 IDAPython 脚本批量重命名

Step 5: 迭代
  - 第一轮迁移的函数成为新的锚点
  - 进入这些函数，继续对比内部调用
  - 重复直到覆盖所有目标函数
```


|---------|--------|------|


|------|------|---------|


```text
found_call → idapro_rename(addr=call_target, name=func_name)
found_vcall → idapro_set_comments(addr=insn_va, comment="vcall: {func_name} @ +{offset}")
found_funcptr → idapro_rename(addr=funcptr_target, name=funcptr_name)
found_gv → idapro_rename(addr=gv_addr, name=gv_name)
found_struct_offset → idapro_set_comments(addr=insn_va, comment="{struct_name}.{member_name}")
```


```text
已有：ntoskrnl.exe 10.0.26100.2000 + 完整 PDB
目标：ntoskrnl.exe 10.0.26100.2605（PDB 被下架）
需求：定位 PspSetCreateProcessNotifyRoutine 的新地址

步骤：
1. 两个版本都加载到 IDA
2. 找到导出函数 PsSetCreateProcessNotifyRoutine（两个版本都有）
3. 旧版中它调用了 PspSetCreateProcessNotifyRoutine（有符号）
4. 新版中它调用了 sub_140822108（无符号）
5. LLM 一眼看出：sub_140822108 = PspSetCreateProcessNotifyRoutine
6. 批量应用
```


```text
已有：target.exe v1.0 的完整逆向结果（200+ 函数已命名）
目标：target.exe v1.1（所有符号丢失）
需求：批量迁移 200 个函数名

步骤：
1. 从旧版导出所有已命名函数的反汇编+伪代码
2. 在新版中通过导出函数/字符串找到对应锚点
3. 批量调用 LLM 比对
4. 解析 YAML，批量 rename
5. 迭代深入
```


|------|---------|------|------|


---


|------|------|-----------|


---
