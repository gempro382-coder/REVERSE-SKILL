

|------|------|-----------|


```text
1. dnSpyEx 打开（通常没混淆，少数团队会加 ConfuserEx）
2. 看 Program.Main 或入口命令分发（Rubeus 是 switch(command) 结构）
3. 找目标命令的实现类/方法
4. 看 P/Invoke 段（Interop.* 命名空间）—— native API 调用在这里
5. 提取内嵌资源（有些工具嵌配置/模板）
6. 如需改特征（EDR 规避）：改命令字符串、API 调用、字符串常量
```


```text
入口: Rubeus.CommandLineParser → 解析 args
分派: switch(command) → "kerberoast" → 执行 Ask.TGS(...)
P/Invoke: Rubeus.Interop.Lsa* / Native.cs → native Kerberos API
关键: LsaCallAuthenticationPackage (KERB_RETRIEVE_TKT_REQUEST)
```


```powershell
# dnSpyEx 里看 Resources（资源树）
# 或命令行
powershell -c "[System.Reflection.Assembly]::LoadFile('target.exe').GetManifestResourceNames()"
# 找到资源后 dnSpyEx 右键 → 提取 / Save
```


---


```powershell
# 方式 A：Chocolatey
choco install dnspy ilspy de4dot detect-it-easy

# 方式 B：手动下载 release（推荐，版本可控）
# dnSpyEx:    https://github.com/dnSpyEx/dnSpy/releases
# de4dot:     https://github.com/de4dot/de4dot/releases
# ILSpy:      https://github.com/icsharpcode/ILSpy/releases
# DIE:        https://github.com/horsicq/Detect-It-Easy/releases
# dnlib:      dotnet add package dnlib  (NuGet)
```


```bash
# ILSpy CLI 反编译
dotnet tool install -g ilspycmd
ilspycmd target.exe -p -o outdir/         # 反编译到目录

# de4dot 跨平台（需 mono 或 dotnet）
# 从 release 下载 de4dot 产物的 .dll，用 dotnet 跑
dotnet de4dot.dll target.exe -o target-clean.exe

# dnlib（脚本化，需 dotnet SDK）
dotnet new console -o dnclean && cd dnclean
dotnet add package dnlib

# DIE CLI (diec)
# Linux: 从 https://github.com/horsicq/Detect-It-Easy 装
diec target.exe
```


```bash
# Linux
sudo apt install dotnet-runtime-8.0        # 或 6.0/7.0 看目标
# macOS
brew install --cask dotnet-sdk
```


---


|------|------|------|


```json
{
  "mcpServers": {
    "dnspy": {
      "command": "dotnet",
      "args": ["path/to/dnspy-mcp.dll"]
    }
  }
}
```


---
