
>
>


- asset_types: [web, frontend_js, public_openapi, screenshot, local_source]


- lead_role: lead
- specialists: [cre, doc]


|------|-------------|----------------|----------|


- path_type: callflow


|------|------|---------|------|


```bash
# 静态调用面与状态字段
rg -n 'customerToken|productType|upiExpiresAt|customerResubmitCount|sessionStorage' {formatted_chunk}

# 公开 API 结构
jq '{openapi,info,paths:(.paths|keys),security:.components.securitySchemes}' openapi.json

# 本地质量门
npm run lint
npm test
npx tsc --noEmit
npm run build
BASE_URL=http://127.0.0.1:{port} npm run smoke
```


- OS: macOS


---
