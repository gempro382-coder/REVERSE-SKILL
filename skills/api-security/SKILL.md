---
name: api-security
description: Use for authorized security assessment of REST, GraphQL, WebSocket, or SOAP APIs, including discovery, authentication, authorization, rate-limit, and CI/CD testing.
---


```text
主动发现：
□ Vespasian: 无头浏览器爬取 → 自动生成 OpenAPI 3.0 / GraphQL SDL 规范
□ Entropy --discover: 从 robots.txt + JS 文件提取端点
□ Kiterunner / ffuf: 爆破未文档化的端点路径
□ 检查常见路径: /swagger.json, /openapi.json, /graphql, /api-docs

GraphQL 内省（三级尝试）：
  1. 标准内省查询
  2. 精简查询（绕过 WAF 全量封禁）
  3. 仅查 __schema { types { name } }（最小探测）
```


```text
JWT 分析（jwt_tool / Burp）：
□ alg:none 攻击: 修改头部为 "alg":"none"，清空签名
□ 密钥混淆: RS256 公钥 → HS256 对称密钥
□ 弱 HMAC 密钥爆破: jwt_tool -C -d wordlist.txt
□ 过期/声明篡改: 修改 exp/iat/sub/role 声明
□ kid 注入: ../../etc/passwd → HMAC 签名绕过

OAuth 2.0：
□ redirect_uri 操控 → 授权码泄漏
□ CSRF via state 参数缺失
□ Token 在 Referer 头泄漏
□ PKCE 缺失检测

GraphQL 认证：
□ mutation 通过 GET 请求绕过认证（CSRF）
□ 批查询认证绕过
```


```text
BOLA（对象级授权绕过）：
□ 遍历数字 ID: /user/1 → /user/2 → /user/3
□ 遍历 UUID
□ 遍历用户名/邮箱
□ Burp Autorize: 双会话重放对比

BFLA（功能级授权绕过）：
□ 普通用户执行管理员 API
□ HTTP 方法切换: GET → PUT → PATCH → DELETE
□ API 版本降级: /v2/admin → /v1/admin
□ 批量操作注入: {"users": [1,2,3]} → {"users": [1,2,3,admin_id]}

工具: Burp Autorize, AuthMatrix, Entropy (malicious_insider persona)
```


```text
内省泄漏 → 信息暴露检测
别名过载 → 100+ 别名 DoS
批查询 → 10+ 同时查询 DoS
字段重复 → __typename × 500
指令过载 → 递归 @skip/@include
循环查询 → 深度嵌套内省递归
字段建议 → 错误消息信息泄漏
GraphiQL/Playground 暴露 → IDE 公开风险
GET 突变 → CSRF 风险
追踪/调试模式 → 元数据泄漏

工具: FireTail, Escape DAST, api.sh (Phases 1-3)
```


```text
□ HTTP 方法切换: GET→POST→PUT→DELETE→OPTIONS→PATCH
□ Content-Type 篡改: JSON→XML→multipart
□ NoSQL 注入: {"username": {"$gt": ""}}
□ SSRF via URL 参数: webhook URL/头像 URL/导入 URL
□ XXE in XML 端点
□ 参数污染: /api?role=user&role=admin
□ 批量赋值: 向请求体添加 is_admin: true
```


```text
□ Entropy compare: diff v1 vs v2 API → 状态码变化/字段删除/延迟回归
□ 多角色工作流测试: admin/user/readonly 权限矩阵
□ 优惠券/积分/价格操控
□ 竞态条件: 并发请求测试 TOCTOU
```


```text
□ 端点发现
□ 消息注入（注入 payload、原型污染）
□ 超大消息处理
□ 类型混淆
□ 跨站点 WebSocket 劫持（CSWH）
```


```text
□ 限速绕过 via 头部: X-Forwarded-For, X-Real-IP
□ 路径变体: /api/ → /api → /Api/ → /API/
□ Slowloris 低带宽耗尽
□ GraphQL 批查询深度嵌套 DoS
□ IP 轮换测试（ProxyCat 代理池）
```


```text
□ 响应过度暴露: 对比 API 返回 vs UI 展示
□ 分页枚举: ?page=1&limit=10000
□ 错误消息信息泄漏: 堆栈跟踪/内部路径/SQL 错误
□ GraphQL 嵌套遍历访问越权数据
□ OpenAPI 规范暴露敏感端点
```


```text
□ Entropy --ci --watch: spec 变更时自动重跑
□ Escape DAST: 按严重度阈值自动阻断构建
□ 发现持久化为回归测试
□ StackHawk（开发者优先、ZAP 内核）
```
