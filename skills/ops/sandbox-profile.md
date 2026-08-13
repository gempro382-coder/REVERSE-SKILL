

```powershell
powershell -File skills\scripts\bootstrap-reverse.ps1 -Capability @('jadx','nmap','yara') -StartServices
powershell -File skills\scripts\refresh-tool-index.ps1
```


```text
最小：nmap + nuclei + sqlmap 容器或 pentestMCP 类镜像
移动：jadx + apktool + frida 宿主机
逆向：宿主机 IDA/r2 + tool-index
```
