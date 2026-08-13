

|------|------|---------|------|


```bash
# 确认文件类型
file LinYuDriverLoader4.9.sh
# ELF 64-bit LSB executable, ARM aarch64

# 查看程序头
readelf -l binary 2>/dev/null | head -20

# 提取压缩数据
dd if=binary bs=1 skip=$((0xa6a24)) count=1981 of=compressed.bin

# 计算文件偏移
# vaddr 0x3d66bc → file_offset = 0x3d66bc - 0x330000 = 0xa66bc
```

```python
# LZSS 解压器核心（简化）
def decompress(data):
    shift_reg = 0x80000000
    # ... 位流读取 + 字面量/匹配分支
```


```text
入口点 → 少量初始化 → 调用解压函数 → mmap(RW) → 解压到 mmap 区域 → mprotect(RX) → 跳转
```

```text
lsl w4, w4, #1    # 左移（提取最高位到 carry）
cbz w4, refill    # 如果移空了，从输入加载新的 32 位
```


---
