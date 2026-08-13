

---


```bash
pip install ciphey
# 自动检测并解密
ciphey -t "密文"
# 从文件读取
ciphey -f encrypted.txt
```


```text
在线版：https://gchq.github.io/CyberChef/
离线版：下载 GitHub Release 的 HTML 文件直接打开

常用 Recipe：
- From Base64 → 解 Base64
- XOR → 异或解密（可暴力尝试 key）
- AES Decrypt → AES 解密
- Magic → 自动检测编码类型
```

---


```bash
# 识别哈希类型
hashid '5f4dcc3b5aa765d61d8327deb882cf99'
# 输出: [+] MD5

# haiti（更准确）
haiti '5f4dcc3b5aa765d61d8327deb882cf99'

# Hashcat 破解
hashcat -m 0 hash.txt rockyou.txt  # MD5
hashcat -m 1000 hash.txt rockyou.txt  # NTLM
```

---


```bash
# RsaCtfTool 自动攻击
python RsaCtfTool.py --publickey pub.pem --private
python RsaCtfTool.py --publickey pub.pem --uncipherfile cipher.txt

# 支持的攻击：
# Wiener、Boneh-Durfee、Fermat、Pollard p-1、Williams p+1
# Common modulus、Small q、Hastads、Noveltyprimes 等
```

---


```bash
# 猜测 XOR key 长度
xortool encrypted_file
# 用猜测的 key 长度解密
xortool -l 4 -c 00 encrypted_file

# 已知明文攻击（知道部分明文）
xortool-xor -f encrypted -s "known_plaintext"
```

---


---


---


---


---


```text
拿到一段未知数据：

1. 看长度和字符集
   - 只有 hex 字符 → 可能是 hex 编码或哈希
   - 末尾有 = → Base64
   - 三段点分 → JWT
   - 32/40/64 字符 hex → 哈希（MD5/SHA1/SHA256）

2. 用 Ciphey 自动尝试
   ciphey -t "数据"

3. 如果 Ciphey 失败 → 用 CyberChef Magic 模式

4. 如果是哈希 → hashID 识别类型 → Hashcat/John 破解

5. 如果是 RSA → RsaCtfTool 自动攻击

6. 如果是 XOR → xortool 分析 key

7. 如果是传统 ZIP 加密 → 优先使用 `bkcrack` 已知明文攻击，不要先做无证据的密码暴力

8. 如果是自定义加密 → IDA/Ghidra 逆向算法 → 手写解密脚本
```

---
