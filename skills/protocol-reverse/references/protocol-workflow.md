

```python
import struct
def parse_frame(buf: bytes):
    magic, length, msg_type = struct.unpack_from(">IHI", buf, 0)
    body = buf[10:10+length]
    return {"magic": magic, "type": msg_type, "body": body}
```


```bash
tshark -r cap.pcap -Y "tcp.port==4433" -T fields -e tcp.payload | head
```
