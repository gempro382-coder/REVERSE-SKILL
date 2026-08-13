

|------|------|---------|------|


```javascript
// Node.js 复现
const crypto = require('crypto');

function generateSign(params, timestamp, secretKey) {
    // 1. 参数按 key 字母序排序
    const sorted = Object.keys(params).sort().map(k => `${k}=${params[k]}`).join('&');
    // 2. 拼接时间戳
    const message = sorted + '&timestamp=' + timestamp;
    // 3. HMAC-SHA256
    return crypto.createHmac('sha256', secretKey).update(message).digest('hex');
}

const params = { user_id: '123', action: 'query' };
const timestamp = Math.floor(Date.now() / 1000);
const secretKey = 'hardcoded_key_from_webpack';
console.log(generateSign(params, timestamp, secretKey));
```


```text
1. 抓包找到带签名的请求
2. 用 initiator/调用栈定位签名函数
3. 分析签名逻辑（参数排序 + 拼接 + 加密）
4. 找密钥来源（硬编码/接口返回/时间派生）
5. Node.js 复现
6. 对比验证
```

```text
- HmacSHA256(sorted_params, key) → 最常见
- MD5(params + salt + timestamp) → 较老的系统
- AES(JSON.stringify(params), key) → 加密而非签名
- RSA sign → 少见，通常是金融类
```


- OS: Windows


---
