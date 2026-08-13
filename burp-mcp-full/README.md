# BurpSuite MCP Full Control Extension


**Windows**:
```cmd
cd burp-mcp-full
build.bat
```

**Linux / Kali / macOS**:
```bash
cd burp-mcp-full
chmod +x build.sh
./build.sh
```


```
Burp Suite → Extensions → Add → Java → 选择 build/libs/burp-mcp-full.jar
```

```
[MCP] Server started on http://127.0.0.1:9876
```


```json
{
  "mcpServers": {
    "burpsuite": {
      "command": "node",
      "args": ["<本目录路径>/mcp-bridge.js"]
    }
  }
}
```


|------|------|
| Scope / Sitemap | `sitemap`, `target_info`, `get_scope`, `add_to_scope`, `remove_from_scope`, `add_issue` |
| Collaborator | `collaborator_generate`, `collaborator_poll` |


|------|------|


|------|------|


|------|------|


```json
POST http://127.0.0.1:9876
{"tool": "proxy_history", "params": {"limit": 10, "url_filter": "personalblog"}}
```

```json
POST http://127.0.0.1:9876
{"tool": "send_request", "params": {"method": "GET", "url": "https://example.com/api/test"}}
```

```json
POST http://127.0.0.1:9876
{
  "tool": "intruder_attack",
  "params": {
    "url_template": "https://target.com/api/verify?code=@@",
    "method": "POST",
    "from": 0,
    "to": 999999,
    "pad_digits": 6,
    "success_length_not": 176,
    "headers": {"User-Agent": "Mozilla/5.0"}
  }
}
```

```json
POST http://127.0.0.1:9876
{"tool": "intercept_toggle", "params": {"enable": false}}
```


|------|------|


```bash
cd burp-mcp-full
gradle jar      # 需本机已装 Gradle 8.7+
# 输出: build/libs/burp-mcp-full.jar
```
