

---


```text
□ android:debuggable="true" → 可调试（生产环境不应出现）
□ android:allowBackup="true" → 数据可备份提取
□ android:exported="true" 的组件 → 暴露的 Activity/Service/Receiver/Provider
□ 自定义权限 protectionLevel → 是否为 normal（应为 signature）
□ intent-filter 中的 scheme → 自定义 deeplink 是否可被劫持
□ android:usesCleartextTraffic="true" → 允许明文 HTTP
□ minSdkVersion 过低 → 可能缺少安全特性
```


```text
□ 硬编码密钥/Token（搜索 "key"、"secret"、"password"、"api_key"）
□ 不安全的随机数（java.util.Random 而非 SecureRandom）
□ 不安全的加密（ECB 模式、DES、MD5 用于密码）
□ WebView 配置（setJavaScriptEnabled + addJavascriptInterface = RCE 风险）
□ SQL 注入（rawQuery 拼接用户输入）
□ 路径遍历（ContentProvider 的 openFile 未校验路径）
□ 日志泄露（Log.d/Log.i 输出敏感信息）
□ 剪贴板泄露（ClipboardManager 存储敏感数据）
□ 隐式 Intent 泄露（sendBroadcast 未指定包名）
```


```text
□ 过时的 OkHttp/Retrofit 版本（已知漏洞）
□ 过时的 WebView 内核
□ 含已知漏洞的 SDK（检查 CVE）
□ 广告 SDK 数据收集范围
□ 推送 SDK 配置（是否泄露 token）
```

---


```bash
# 追踪所有加密操作
frida-trace -U -f com.target.app -j '*Cipher*!*'

# 追踪所有 HTTP 请求
frida-trace -U -f com.target.app -j '*OkHttp*!*'

# 追踪 SharedPreferences 读写
frida-trace -U -f com.target.app -j '*SharedPreferences*!*'

# 追踪所有 native 函数调用
frida-trace -U -f com.target.app -i 'Java_*'
```


```bash
# 连接
objection -g com.target.app explore

# 常用命令
android hooking list activities
android hooking list services
android sslpinning disable
android root disable
android clipboard monitor
env                              # 查看应用目录
sqlite connect <db_path>         # 连接数据库
```

---


```text
方法 1: 系统代理 + Burp/mitmproxy
- 设置 WiFi 代理 → Burp 监听地址
- 安装 CA 证书到设备
- Android 7+ 需要 network_security_config 或 Frida 绕过

方法 2: VPN 模式（推荐）
- 使用 HttpCanary / Packet Capture
- 不需要 root，不需要配置代理
- 但无法解密 SSL Pinning 的流量

方法 3: Frida + r2frida
- 直接在进程内拦截网络调用
- 不受代理/VPN 限制
```


```text
□ 是否使用 HTTPS（所有 API 调用）
□ 是否有 SSL Pinning（证书绑定）
□ 证书验证是否正确（不接受自签名）
□ 是否有证书透明度（CT）检查
□ API 密钥是否在请求中明文传输
□ Token 是否有过期机制
□ 是否有请求签名防篡改
□ 是否有重放攻击防护（nonce/timestamp）
□ WebSocket 是否加密
□ 是否有敏感数据在 URL 参数中（会被日志记录）
```

---


---


```bash
# 越权测试
curl -H "Authorization: Bearer USER_A_TOKEN" \
     "https://api.target.com/users/USER_B_ID/profile"

# Token 重放
# 1. 正常登录获取 token
# 2. 登出
# 3. 用旧 token 请求 → 应该返回 401

# 短信验证码爆破
for code in $(seq 0000 9999); do
    curl -X POST "https://api.target.com/verify" \
         -d "phone=13800138000&code=$code"
done
```

---


---


```text
1. [5min] 解包 + Manifest 审计
   apktool d app.apk
   检查 debuggable/allowBackup/exported/cleartext

2. [10min] 代码快速审计
   jadx -d out app.apk
   搜索: password, key, secret, token, http://

3. [5min] 网络测试
   配置代理 → 操作 APP → 检查是否有明文/弱加密

4. [5min] 存储检查
   adb shell → 检查 shared_prefs 和 databases

5. [5min] 动态验证
   Frida hook 关键函数 → 确认发现
```
