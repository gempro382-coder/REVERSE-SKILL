---
name: threat-hunting
description: Use for blue-team threat hunting, detection engineering with Sigma/YARA, SIEM query design, and incident detection validation.
---

# Threat Hunting & Detection Engineering


```text
例：攻击者用 living-off-the-land 做横向
→ 数据源：Sysmon 1/3/10、Windows Security 4624/4648
→ 成功标准：发现异常父进程或罕见账户日志源
```


```text
□ 基线：正常管理员行为时段与主机
□ 异常：新服务、编码 PowerShell、异常出站
□ 关联：同账号多主机短时登录
```


```yaml
# Sigma 骨架见 malware-analysis；本 skill 强调：
# - 误报面
# - 数据源字段映射
# - 响应 playbook 链接
```


```text
□ 原子测试（Atomic Red Team）仅在授权实验室
□ 回放历史日志验证召回
```


- `references/hunting-loop.md`
- `../malware-analysis/references/yara-sigma-rules.md`
- `../digital-forensics/`
