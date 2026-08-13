

   ```bash
   xxd dump.bin | head -20
   ```


|------|------|---------|------|


```python
from scapy.all import *

class MyMsg(Packet):
    name = "MyProto"
    fields_desc = [
        StrFixedLenField("magic", b"\xab\xcd", 2),
        ByteField("version", 1),
        ByteField("type", 0),
        LenField("length", None, fmt="H"),     # H = uint16 BE
        XIntField("seq", 0),
        StrLenField("payload", "", length_from=lambda p: p.length - 8),
        XShortField("crc", 0),
    ]

# 解析 PCAP
pkts = rdpcap('dump.pcap')
for p in pkts:
    if TCP in p and p[TCP].dport == 9527 and p.payload:
        msg = MyMsg(bytes(p[TCP].payload))
        msg.show()
```


```yaml
# myproto.ksy
meta:
  id: myproto
  endian: be
seq:
  - id: magic
    contents: [0xab, 0xcd]
  - id: version
    type: u1
  - id: type
    type: u1
  - id: length
    type: u2
  - id: seq_no
    type: u4
  - id: payload
    size: length - 8
  - id: crc
    type: u2
```


```bash
binwalk -E dump.bin             # 熵图
ent dump.bin                    # 数值
```


```text
1. 看节奏（I/O 图 + Conversations 找出会话边界）
2. 找帧界（magic / length / 终止符）
3. 拆字段（固定头、长度、payload、校验）
4. 验加密（熵 + 找 nonce + 二进制反查 send 函数）
```


- Kali / Ubuntu，Wireshark 4.x, Python 3.10+, scapy 2.5
