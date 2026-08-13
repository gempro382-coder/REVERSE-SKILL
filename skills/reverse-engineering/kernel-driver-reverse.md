

---


|------|------|---------|


```text
1. 找 DriverEntry（入口点）
   - IDA 自动识别，或搜索 IoCreateDevice / IoCreateSymbolicLink

2. 找设备名和符号链接
   - IoCreateDevice → DeviceName（如 \Device\MyDriver）
   - IoCreateSymbolicLink → SymLink（如 \DosDevices\MyDriver）

3. 找 Dispatch 例程
   - DriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL] = DispatchIoctl
   - 这是用户态通过 DeviceIoControl 调用的入口

4. 分析 IOCTL 处理
   - switch(IoControlCode) 分发不同功能
   - IOCTL 编码：CTL_CODE(DeviceType, Function, Method, Access)
   - Method: METHOD_BUFFERED / METHOD_IN_DIRECT / METHOD_OUT_DIRECT / METHOD_NEITHER

5. 找漏洞
   - 用户可控缓冲区未验证长度 → 溢出
   - METHOD_NEITHER 直接使用用户指针 → 任意读写
   - 未检查 IOCTL 权限 → 非特权用户可调用
```


```python
# 解析 IOCTL code
def decode_ioctl(code):
    device_type = (code >> 16) & 0xFFFF
    access = (code >> 14) & 0x3
    function = (code >> 2) & 0xFFF
    method = code & 0x3
    
    methods = {0: "BUFFERED", 1: "IN_DIRECT", 2: "OUT_DIRECT", 3: "NEITHER"}
    access_types = {0: "ANY", 1: "READ", 2: "WRITE", 3: "READ|WRITE"}
    
    return f"DevType=0x{device_type:X} Func=0x{function:X} Method={methods[method]} Access={access_types[access]}"

# 示例
decode_ioctl(0x80002034)
# DevType=0x8000 Func=0x80D Method=BUFFERED Access=ANY
```


|------|------|------|


---


```text
关键函数：
- init_module / module_init → 模块加载时执行
- cleanup_module / module_exit → 模块卸载时执行

关键结构：
- struct file_operations → 字符设备的 open/read/write/ioctl
- struct net_device_ops → 网络设备操作
- struct block_device_operations → 块设备操作
```


```text
1. 确认是内核模块
   file module.ko → "ELF 64-bit ... relocatable"（注意是 relocatable 不是 executable）

2. 找 init/exit 函数
   readelf -s module.ko | grep -E "init_module|cleanup_module"
   或在 .modinfo section 找模块信息

3. 找 file_operations 结构
   搜索 register_chrdev / cdev_add / misc_register
   → 找到 fops 结构体 → 定位 ioctl/read/write 处理函数

4. 分析 ioctl 处理
   unlocked_ioctl / compat_ioctl 函数
   → switch(cmd) 分发

5. 找 Rootkit 行为
   - 修改 sys_call_table → syscall hook
   - 修改 /proc 文件系统 → 隐藏进程/文件
   - 注册 netfilter hook → 隐藏网络连接
   - 修改 VFS 层 → 隐藏文件
```


|------|------|---------|


|------|------|

---


|---------|-----------|


|---------|-----------|


```text
1. 找 vtable
   - 搜索连续的函数指针数组（在 .rodata 或 .rdata 段）
   - 构造函数中 `mov [rcx], offset vtable` 写入 vtable 指针

2. 确定类层次
   - vtable 前 -8 偏移处通常是 RTTI 指针（如果未 strip）
   - 多个 vtable 共享前几个条目 → 继承关系

3. 标注虚函数
   - vtable[0] 通常是析构函数（或 deleting destructor）
   - 后续按偏移标注：vtable[1] = func1, vtable[2] = func2...

4. IDA 中操作
   - 在 vtable 地址创建 struct（每个字段是函数指针）
   - 对 `call [rax+offset]` 添加注释标明调用的虚函数
```


```text
方法 1：从访问模式推断
  mov eax, [rdi+0x00]  → field_0: int/ptr (4/8 bytes)
  mov ecx, [rdi+0x08]  → field_8: int/ptr
  movss xmm0, [rdi+0x10] → field_10: float

方法 2：从 sizeof 推断
  call malloc(0x30) → 结构体大小 0x30 (48 bytes)
  
方法 3：从构造函数推断
  构造函数会初始化所有字段 → 字段类型和偏移一目了然

方法 4：用 IDA 的 "Create struct" 功能
  选中访问模式 → Edit → Struct → Create struct from selection
```

---


|--------|---------|
| GCC | `__stack_chk_fail`、`-fstack-protector`、`.note.GNU-stack` |


|---------|------|

---


### Windows

```text
调试器：WinDbg Preview
连接方式：网络调试（推荐）或串口

被调试机设置：
bcdedit /debug on
bcdedit /dbgsettings net hostip:192.168.x.x port:50000

调试机连接：
WinDbg → File → Attach to Kernel → Net → Port:50000 Key:xxx

常用命令：
!analyze -v          # 自动分析崩溃
lm                   # 列出已加载模块
!drvobj \Driver\xxx  # 查看驱动对象
dt nt!_DRIVER_OBJECT # 显示结构体
bp module!function   # 下断点
```

### Linux

```text
调试器：GDB + QEMU 或 kgdb

QEMU 内核调试：
qemu-system-x86_64 -kernel bzImage -s -S ...
gdb vmlinux -ex "target remote :1234"

常用命令：
info threads         # 内核线程
lx-symbols           # 加载内核符号（需要 scripts/gdb/）
p init_task          # 查看 init 进程
lx-dmesg             # 内核日志
```

---


|------|------|------|
