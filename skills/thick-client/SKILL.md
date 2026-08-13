---
name: thick-client
description: Use for authorized security testing of desktop thick clients including local storage, update channels, IPC, traffic, and client-side trust boundaries.
---

# Thick Client Security Testing


```text
□ 进程树、子进程、驱动/服务
□ 监听端口与出站域名
□ 本地敏感路径：%APPDATA%、Keychain、注册表
```


```text
□ 明文配置、硬编码密钥、调试开关
□ DLL 劫持/搜索顺序（Windows）
□ 数据库文件（SQLite）权限与加密
□ IPC：谁可连接？是否鉴权？
```


```text
□ 系统代理 / 应用自定义 TLS
□ 证书钉扎 → 联合 mobile/js 方法学或 Frida
□ API 越权：客户端隐藏的管理接口
```


```text
□ .NET → dotnet-reverse；原生 → ida/ghidra；Electron → asar + js-reverse
```


- `references/thick-client-checklist.md`
- `../dotnet-reverse/` `../ida-reverse/` `../js-reverse/` `../api-security/`
