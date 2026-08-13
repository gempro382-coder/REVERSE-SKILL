

---


---


- https://github.com/swisskyrepo/PayloadsAllTheThings
- https://book.hacktricks.wiki/

---


---


```python
# ret2libc 模板
from pwn import *
elf = ELF('./vuln')
libc = ELF('./libc.so.6')
p = process('./vuln')
# leak libc base → calculate system/binsh → overwrite ret
```

---


---


---


---


---
