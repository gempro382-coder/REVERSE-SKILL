

### LLM4Decompile

### Decaf (2026)

### Constraint-Guided Multi-Agent (2026)

### REMEND (2026)

### Glaurung


```text
□ strings 提取 → LLM 语义分类（URL/密钥/路径/协议）
□ 导入表分析 → LLM 推断功能（加密=OpenSSL? 网络=libcurl?）
□ 反汇编片段 → LLM 识别模式（密码算法、反调试、虚拟机检测）
□ 错误消息 → LLM 推断上下文（"Invalid license" → 授权逻辑位置）
```


```bash
# LLM4Decompile
python llm4decompile.py --binary target.so --arch arm64 --output target.c

# 验证结果（重编译 + 对比）
gcc -O2 -o target_recompiled target.c -fPIC -shared
# → 验证输出行为等价性
```


```text
Agent 1 (语法): 检查生成的 C 代码是否可以 parse
  ↓ 失败 → 反馈错误信息给 LLM 重试
Agent 2 (编译): GCC 编译 → 检查 warnings/errors
  ↓ 失败 → 反馈编译错误给 LLM
Agent 3 (行为): LLM 生成输入 → 运行原始和重编译版本 → 对比输出
  ↓ 不一致 → 反馈差异给 LLM → 迭代修正
```


```text
□ 函数重命名: 输入反编译伪代码 → LLM 建议语义化名称
□ 类型恢复: 分析上下文 → LLM 推断结构体/类定义
□ 算法识别: 汇编片段 → LLM 识别密码算法（AES/TEA/RC4/自定义）
□ 协议逆向: 网络包序列 → LLM 推断协议格式
□ 注释生成: 反编译代码 → LLM 生成中文/英文注释
```


```text
问题: macOS 私有框架无文档，类型信息缺失
方案: LLM 分析使用模式 → 推断方法签名和参数类型
效果: ObjC 签名恢复 15% → 86% (vs 静态分析)
```


```
You are a reverse engineering expert. Analyze this decompiled function:

[伪代码]

1. What does this function do? (one sentence)
2. Suggest a meaningful function name.
3. What are the input parameters and their likely types?
4. What is the return value?
5. What external APIs/functions does it depend on?
6. Any security-relevant operations (crypto, auth, network, file I/O)?
```


```
Analyze this assembly/disassembly for cryptographic operations:

[汇编代码]

1. Is this a known cryptographic algorithm? (AES/DES/RC4/TEA/ChaCha20/custom?)
2. Identify the key schedule and round structure.
3. What is the key size?
4. Are there any hardcoded constants that identify the algorithm?
```


```
Given this network packet sequence, infer the protocol structure:

[hex dump]

1. Identify magic bytes and length fields.
2. Propose a struct definition for the packet header.
3. What field(s) appear to be checksums/CRCs?
4. Is this a known protocol or custom?
```


Source: Decaf (2026), REMEND (2026), Constraint-Guided Multi-Agent Decompilation (2026), LLM4Decompile, Glaurung
