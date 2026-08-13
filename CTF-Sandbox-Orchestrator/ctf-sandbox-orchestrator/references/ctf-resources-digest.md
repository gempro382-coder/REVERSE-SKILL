

---


|------|------|------|

---


|------|------|


- https://github.com/swisskyrepo/PayloadsAllTheThings
- https://book.hacktricks.wiki/

---


|------|------|


---


|------|------|
| one_gadget | libc one-shot |


```python
# ret2libc 模板
from pwn import *
elf = ELF('./vuln')
libc = ELF('./libc.so.6')
p = process('./vuln')
# leak libc base → calculate system/binsh → overwrite ret
```

---


|------|------|

- AES（ECB/CBC padding oracle/bit flipping）

---


|------|------|


---


|------|------|


---


|------|------|------|

---


|------|------|
| CTFTime Writeups | https://ctftime.org/writeups |
| 0xdf hacks stuff | https://0xdf.gitlab.io/ |
| LiveOverflow (YouTube) | https://www.youtube.com/c/LiveOverflow |
| John Hammond (YouTube) | https://www.youtube.com/c/JohnHammond010 |
| IppSec (HTB walkthrough) | https://www.youtube.com/c/ippsec |
