

---


```text
偏移  大小  字段              说明
0x00  4    e_ident[EI_MAG]   Magic: 7f 45 4c 46 ("\x7fELF")
0x04  1    e_ident[EI_CLASS] 1=32bit, 2=64bit
0x05  1    e_ident[EI_DATA]  1=LE, 2=BE
0x10  2    e_type            2=EXEC, 3=DYN(PIE/SO), 4=CORE
0x12  2    e_machine         0x03=x86, 0x3E=x86_64, 0xB7=AArch64, 0x28=ARM
0x18  8    e_entry           入口点虚拟地址
0x20  8    e_phoff           程序头表偏移
0x28  8    e_shoff           节头表偏移（strip 后可能为 0）
0x38  2    e_phnum           程序头数量
0x3C  2    e_shnum           节头数量
```


```text
类型值  名称       说明
0x01   PT_LOAD    可加载段（代码/数据）
0x02   PT_DYNAMIC 动态链接信息
0x03   PT_INTERP  解释器路径（/lib/ld-linux.so）
0x04   PT_NOTE    辅助信息
0x06   PT_PHDR    程序头表自身
0x6474e550 PT_GNU_EH_FRAME  异常处理
0x6474e551 PT_GNU_STACK     栈可执行标记
0x6474e552 PT_GNU_RELRO     只读重定位
```


---


```text
特征：
1. 入口点附近有 mmap(PROT_READ|PROT_WRITE|PROT_EXEC) 调用
2. 紧接着有 memcpy 或循环拷贝
3. 然后 mprotect 改权限
4. 最后 br/jmp 到新映射的地址

分析策略：
1. 找到 mmap 调用 → 记录返回的地址
2. 在 mprotect(PROT_EXEC) 后下断点
3. dump 解压后的内存区域
4. 作为新的二进制分析
```

---


```text
函数序言：
  stp x29, x30, [sp, #-N]!    # 保存 FP 和 LR
  mov x29, sp                  # 设置帧指针

函数尾声：
  ldp x29, x30, [sp], #N      # 恢复 FP 和 LR
  ret                          # 返回（br x30）

系统调用：
  mov x8, #NR                  # syscall 号
  svc #0                       # 触发 syscall

条件分支：
  cmp x0, #0
  b.eq label                   # 等于跳转
  b.ne label                   # 不等于跳转
  cbz x0, label                # x0 == 0 跳转
  cbnz x0, label               # x0 != 0 跳转

地址加载：
  adrp x0, page                # 加载页地址高位
  add x0, x0, #offset          # 加低 12 位偏移
  ldr x0, [x1, #offset]        # 从内存加载
```


---


```text
1. 入口点附近有循环 + 位操作（移位、AND、OR）
2. 有"滑动窗口"回拷（从输出缓冲区往回读）→ LZ 系列
3. 有频率表/霍夫曼树构建 → Deflate/Huffman
4. 有固定大小块处理 → 块压缩（LZ4/Snappy）
5. 有算术编码特征（区间缩小）→ LZMA/ANS
```

---


```text
流程：
1. mmap(NULL, size, PROT_READ|PROT_WRITE, MAP_ANON|MAP_PRIVATE, -1, 0)
2. 将 shellcode/payload 写入映射区域
3. mprotect(addr, size, PROT_READ|PROT_EXEC)  # 改为可执行
4. 跳转到映射地址执行

特征：
- mmap 返回值被保存
- 紧接着有 memcpy 或循环写入
- 然后 mprotect 改权限
- 最后 br/blr 到该地址
```


```text
流程：
1. ptrace(PTRACE_ATTACH, target_pid)
2. waitpid(target_pid)
3. ptrace(PTRACE_GETREGS, target_pid, &regs)
4. 修改 regs.pc 指向注入代码
5. ptrace(PTRACE_SETREGS, target_pid, &regs)
6. ptrace(PTRACE_CONT, target_pid)

特征：
- 打开 /proc/<pid>/mem 或使用 ptrace
- 读取/修改目标进程寄存器
- 写入 shellcode 到目标进程空间
```


```text
流程：
1. open("/proc/self/mem", O_RDWR)
2. lseek(fd, target_addr, SEEK_SET)
3. write(fd, new_code, size)

用途：
- 绕过 W^X 保护（mmap 的页不能同时 W+X）
- 修改自身代码段（.text 通常是只读的）
- 运行时 patch 指令
```

---


```text
1. 快速侦察（5 分钟）
   - file / rabin2 -I → 架构、类型、保护
   - strings | grep -i "error\|fail\|http\|/proc\|/dev" → 关键字符串
   - rabin2 -i → 导入函数（如果有）
   - rabin2 -E → 导出函数

2. 结构分析（10 分钟）
   - readelf -l → 程序头（LOAD 段布局）
   - 入口点附近代码 → 是否有解压/解密
   - 找 .init_array → 构造函数（可能有反调试）

3. 定位关键逻辑
   - 从字符串交叉引用入手
   - 从系统调用（mmap/ptrace/open）入手
   - 从网络函数（connect/send/recv）入手

4. 分而治之
   - 如果是自解压 → 先解压，分析 payload
   - 如果是多模块 → 按功能分块分析
   - 用 binary-diff 对比不同版本
```

---


```bash
# 基本信息
file binary
readelf -h binary          # ELF 头
readelf -l binary          # 程序头
readelf -S binary          # 节头（如果有）
rabin2 -I binary           # 综合信息

# 字符串
strings -a binary | less
rabin2 -z binary           # 数据段字符串
rabin2 -zz binary          # 全文件字符串

# 反汇编
r2 -A binary               # radare2 分析
objdump -d binary          # GNU 反汇编
aarch64-linux-gnu-objdump -d binary  # ARM64 交叉反汇编

# 动态分析
strace -f ./binary         # 系统调用跟踪
ltrace -f ./binary         # 库函数跟踪
qemu-aarch64 -strace ./binary  # ARM64 模拟执行

# 内存 dump
gdb -p <pid> -ex "dump memory out.bin 0xADDR 0xADDR+SIZE" -ex quit

# 修复损坏的 ELF
# 手动修改 e_phnum 或 patch 损坏的 PHDR
python -c "
import struct
with open('binary', 'r+b') as f:
    f.seek(0x38)  # e_phnum offset (64-bit)
    f.write(struct.pack('<H', 2))  # 修改为正确的 PHDR 数量
"
```
