

   ```bash
   file firmware.bin
   ```
   ```bash
   binwalk -e firmware.bin
   cd _firmware.bin.extracted/squashfs-root
   ```
   ```bash
   ```
   ```bash
   john --wordlist=rockyou.txt shadow
   ```


   ```bash
   sudo screen /dev/ttyUSB0 115200
   ```


```bash
# 1. 提取
unblob -k firmware.bin -o extracted/

# 2. 跑 firmwalker
git clone https://github.com/craigz28/firmwalker
./firmwalker.sh extracted/squashfs-root

# 3. 模拟启动（如果支持）
docker run -it --rm -v $(pwd):/firmware firmae:latest \
  /work/run.sh -d 1 /firmware/firmware.bin

# 4. 已模拟起 Web → 用 nuclei / nikto / curl 直接扫
```


```bash
for baud in 9600 19200 38400 57600 115200 460800 921600; do
    echo "--- $baud ---"
    timeout 3 sudo cat /dev/ttyUSB0 < <(stty -F /dev/ttyUSB0 $baud cs8 -cstopb -parenb)
done
```


```text
# U-Boot 阶段按键中断（一般是按住空格或 Ctrl+C）
=> setenv bootargs "console=ttyS0,115200 root=/dev/mtdblock2 rootfstype=squashfs init=/bin/sh"
=> saveenv
=> boot
# 启动后直接进 sh，无需密码
```


```text
阶段 1 — 软件
  · 厂商固件下载 + binwalk/unblob 提取
  · firmwalker 跑一遍
  · grep 默认凭据 / 私钥 / 后门字符串
  · QEMU 模拟启动跑 Web 漏扫

阶段 2 — 硬件
  · 拆机找 UART/JTAG 焊点
  · 万用表识别 GND/VCC/TX/RX
  · USB-TTL 接线，确认电平 3.3V

阶段 3 — 调试
  · screen/minicom 监听
  · U-Boot 阶段中断进交互
  · init=/bin/sh 单用户绕密码

阶段 4 — 利用
  · 拿到 root → 看 /etc/shadow 离线破
  · 看 Web 管理界面 CGI 二进制 → 找命令注入 / SSRF
  · 看 UPnP / mDNS / 蓝牙广播逻辑
```


```text
admin / admin
admin / password
root / root
root / 1234
support / support
ubnt / ubnt          # Ubiquiti
admin / 1234         # ZyXEL
```
