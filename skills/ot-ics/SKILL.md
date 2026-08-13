---
name: ot-ics
description: Use for authorized OT/ICS security assessment covering Purdue model zoning, PLC/SCADA exposure, industrial protocol discovery, and safe passive-first evaluation.
---

# OT / ICS Security


```text
MUST NOT 在未明确允许时：
- 对 PLC 写线圈/寄存器
- 全网高速率扫描生产 OT
- 中断安全仪表系统（SIS）相关路径
优先：只读识别、流量镜像、离线固件/配置分析
```


```text
□ Purdue L0–L5 草图：现场设备 → 控制 → 监督 → 站点 DMZ → 企业
□ 资产清单：PLC/RTU/HMI/工程师站/历史库/Jump host
□ 协议与端口基线（仅授权网段）
```


```text
□ SPAN/镜像 PCAP → protocol-reverse / Wireshark 工控解析器
□ 配置与工程文件离线审计（TIA/RSLogix 导出等）
□ 默认口令与明文协议（Modbus 无认证）记录为 Finding，不写盘改值
```


```text
□ 低速识别，维护窗口
□ 只读功能码优先
□ 每步 Evidence；异常立即停止并通报
```


```text
□ 控制器固件版本 → CVE 映射（不盲刷固件）
□ 联合 firmware-pentest 做离线镜像分析
```


|------|------|------|


- `references/ot-safe-assessment.md`
- `../firmware-pentest/` `../protocol-reverse/` `../network` via pentest-tools


- [ ] Checklist / journal？
