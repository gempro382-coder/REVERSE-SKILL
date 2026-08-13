---
name: digital-forensics
description: Use for authorized digital forensics including memory dumps, disk timelines, PCAP investigation, artifact triage, and IR evidence preservation.
---

# Digital Forensics & IR Artifacts


```text
□ 计算 SHA256；记录时区与采集命令
□ 工作在副本上；原始只读
□ chain of custody 备注写入 timeline
```


```bash
vol -f mem.dmp windows.info
vol -f mem.dmp windows.pslist
vol -f mem.dmp windows.netscan
vol -f mem.dmp windows.cmdline
```


```text
□ 事件日志：Security / PowerShell / Sysmon
□ 持久化：Run 键、服务、计划任务、WMI
□ 执行痕迹：Amcache、Prefetch、BAM
```


```text
□ tshark 统计会话与 DNS
□ 导出可疑流 → protocol-reverse 或 malware C2 分析
```


- `references/forensics-triage.md`
- `../malware-analysis/` `../threat-hunting/` `../protocol-reverse/`
