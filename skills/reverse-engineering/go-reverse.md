

---


```bash
# 字符串特征
strings binary | grep -E "runtime\.|go\.buildid|GOROOT"

# rabin2 侦察
rabin2 -z binary | grep -i "runtime"

# 文件大小异常大（静态链接 runtime）
# 典型 Hello World: C ~20KB, Go ~2MB
```


---


|------|------|------|


|------|------|------|


|------|------|------|


|------|------|------|

---


### pclntab (PC Line Table)


```text
定位方法：
1. 搜索 magic bytes: 0xFFFFFFF0 (Go 1.16+) 或 0xFFFFFFFB (Go 1.18+)
2. 用 GoReSym 自动定位
3. 用 go_parser IDA 插件自动解析
```

### moduledata


```text
C 字符串:   "hello\0"
Go 字符串:  struct { ptr *byte; len int } → ptr 指向 "hello"（无 \0）
```


---


```text
1. GoReSym -t -d -p binary > symbols.json
   → 导出所有函数名、类型、源文件路径
2. 加载到 IDA/Ghidra
3. 导入 GoReSym 的符号信息
4. 过滤掉 runtime.* 和标准库函数，聚焦用户代码
5. 从 main.main 开始分析
```


```text
1. GoReSym -t -d -p binary > symbols.json
   → 即使 strip 了，pclntab 通常还在
2. 如果 GoReSym 失败 → 用 redress
   redress -src binary    # 恢复源文件路径
   redress -pkg binary    # 恢复包结构
   redress -type binary   # 恢复类型信息
3. 加载到 IDA + go_parser 插件
4. 运行 go_parser 自动恢复
5. 从恢复的 main.main 开始
```


```text
Garble 会：
- 随机化函数名（main.main → main.a3f2b1c）
- 加密字符串
- 移除文件路径信息
- 混淆包名

对抗方法：
1. GoResolver（CFG 签名匹配）
   → 通过控制流图相似度恢复标准库函数名
2. GoStringUngarbler（字符串解密）
   → 自动识别 Garble 的字符串加密模式并解密
3. 动态分析（Frida/dlv）
   → Hook runtime 函数观察实际行为
4. 对比分析
   → 编译同版本 Go 的 Hello World，用 binary-diff 对比 runtime 部分
```


```text
1. 识别 CGo 边界（_cgo_* 函数）
2. Go 部分用 go_parser 恢复
3. C 部分用常规 IDA 分析
4. 关注 _cgo_topofstack、crosscall2 等桥接函数
```

---


```bash
# GoReSym：导出符号
GoReSym -t -d -p binary > symbols.json
GoReSym -t -d -p binary -o ida_script.py  # 生成 IDA 脚本

# redress：分析 stripped 二进制
redress -src binary          # 源文件路径
redress -pkg binary          # 包结构
redress -type binary         # 类型信息
redress -interface binary    # 接口信息
redress -filepath binary     # 完整文件路径

# GoResolver：去混淆 Garble
GoResolver -binary binary -output resolved.json

# GoStringUngarbler：解密 Garble 字符串
GoStringUngarbler -i binary -o deobfuscated_binary

# 快速判断 Go 版本
strings binary | grep "go1\."
GoReSym -p binary | grep "Version"
```

---


```text
1. 加载二进制（选择正确的架构）
2. 等待自动分析完成
3. 运行 go_parser 插件：
   - File → Script File → go_parser.py
   - 或 Edit → Plugins → Go Parser
4. 插件会自动：
   - 解析 pclntab
   - 恢复函数名
   - 标记 Go 字符串
   - 解析类型信息
5. 过滤视图：
   - 隐藏 runtime.* 函数
   - 聚焦 main.* 和第三方包
6. 从 main.main 开始逆向
```

---


|------|------|------|

---


|------|--------|
