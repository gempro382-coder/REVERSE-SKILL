

|------|------|---------|------|


```bash
# 目录发现
ffuf -u https://target.example.com/api/v1/FUZZ -w /path/to/SecLists/Discovery/Web-Content/api/api-endpoints.txt -rate 10

# IDOR 测试
# 用账号 A 的 token 访问账号 B 的资源
curl -H "Authorization: Bearer <token_A>" https://target.example.com/api/v1/users/USER_B_ID

# 未授权测试
curl https://target.example.com/api/v1/users/USER_B_ID
# 如果返回 200 + 数据 → 未授权访问
```


```text
1. 正常请求（带 token）→ 记录正常响应
2. 去掉 token → 看是否仍返回数据（未授权）
3. 换另一个用户的 token → 看是否能访问（越权）
```

```text
1. 注册两个账号 A 和 B
2. 获取 A 的资源 ID 和 B 的资源 ID
3. 用 A 的 token 请求 B 的资源 ID
4. 如果返回 B 的数据 → IDOR 确认
```


---
