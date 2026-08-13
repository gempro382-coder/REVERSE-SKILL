---
name: pwn-chain
---


```text
Step 1: 确认漏洞类型 + 保护机制
   ├─ checksec ./vuln（NX / Canary / PIE / RELRO / Fortify）
   ├─ file ./vuln  + readelf -d ./vuln
   ├─ 漏洞分类：栈溢出 / 格式化字符串 / 堆 (UAF/DF/OF) / 整数 / 竞态 / 内核
   └─ → 决定走哪个 references/

Step 2: 选择利用策略
   ├─ NX 关 + 无 ASLR → 直接 shellcode
   ├─ NX 开 + 给 libc → ret2libc / one_gadget
   ├─ NX 开 + 不给 libc → leak 后 libc-database 反查
   ├─ 堆 → 按 glibc 版本对应技术 (tcache/fastbin/unsorted/large)
   └─ 内核 → commit_creds / modprobe_path / core_pattern

Step 3: 准备 libc + gadget
   ├─ libc-database：./find puts 0x6f0
   ├─ ROPgadget --binary ./libc.so.6 --only "pop|ret"
   ├─ one_gadget ./libc.so.6
   └─ 计算 base：leak_addr - libc.sym['puts']

Step 4: 写 pwntools 模板（本地 process）
   ├─ context.binary = ELF('./vuln')
   ├─ p = process('./vuln')  /  p = gdb.debug('./vuln','b *main+xx')
   ├─ payload = cyclic(N) + p64(ret) + ...
   └─ p.interactive()

Step 5: 本地通
   ├─ 反复 attach + 看寄存器 + 调 offset
   ├─ 用 pwndbg/GEF 的 vmmap / heap / bins / telescope
   └─ 跑通后切 remote()

Step 6: 远程稳定化
   ├─ libc 偏移：用 leak 反查 libc-database，不要拍脑袋
   ├─ 栈对齐：16-byte 不对齐 → movaps 崩 → 加一个 ret gadget
   ├─ 远程网络延迟 → recvuntil 精确锚字符串，禁用模糊 sleep
   ├─ 远程缓冲：sendlineafter 比 sendline 更稳
   ├─ 堆喷成功率：放大 spray 数量 + 留 padding chunk 防合并
   └─ 多次跑：写 while True 验证成功率 ≥ 95%
```


```text
已有：./vuln（64-bit ELF, NX, PIE, canary）+ ./libc.so.6 + nc host port
漏洞：read(buf, 0x200) 但 buf 只有 0x40 字节 → 栈溢出
保护：canary 拦住，PIE 让 .text 随机化

策略：
1. 先 leak canary（栈/格式化字符串/部分读）
2. 再 leak 一个 libc 函数地址（puts@got）
3. 用 libc.address = leaked - libc.sym['puts'] 算 libc base
4. one_gadget ./libc.so.6 选一个约束能满足的 magic gadget
5. payload = padding + canary + saved_rbp + (pop_rdi + bin_sh + system) 或直接 one_gadget
6. 加一个 ret gadget 修栈对齐（关键！）
```


```text
已有：vmlinux + bzImage + initramfs.cpio.gz + 自定义 vuln.ko
漏洞：ioctl(0x1337, ptr) 里 copy_from_user 长度可控 → kernel heap overflow (kmalloc-64 slab)
保护：SMEP, SMAP, KASLR, KPTI

策略：
1. 改 init 脚本拿到 root shell（CTF）或先 leak KASLR base 再继续（真实）
2. 通过 /proc/kallsyms（可能限权）或未初始化堆喷 leak 内核基址
3. 在 kmalloc-64 slab 里喷 tty_struct / msg_msg / pipe_buffer
4. 覆盖 vtable 指针指向用户态 → 不行（SMEP），改走 stack pivot + 内核 ROP
5. ROP 链：prepare_kernel_cred(0) → commit_creds → swapgs+iretq → 用户态 execve("/bin/sh")
6. 或更省事：覆盖 modprobe_path 为 "/tmp/x"，写一个 /tmp/x，然后触发 modprobe
```


```bash
# 一键检查 + 安装核心工具
for t in pwntools ropgadget ropper; do
  pip show $t >/dev/null 2>&1 || pip install $t
done

command -v one_gadget >/dev/null || gem install one_gadget

[ -d ~/tools/libc-database ] || git clone https://github.com/niklasb/libc-database ~/tools/libc-database
[ -d ~/tools/libc-database/db ] || (cd ~/tools/libc-database && ./get ubuntu debian)

[ -d ~/tools/pwndbg ] || (git clone https://github.com/pwndbg/pwndbg ~/tools/pwndbg && cd ~/tools/pwndbg && ./setup.sh)
```
