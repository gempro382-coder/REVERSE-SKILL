

   ```bash
   adb shell "ps -A | grep com.target.app"
   frida-ps -U | grep target
   ```


|------|------|---------|------|


```javascript
Java.perform(function () {
    // 1. OkHttp 3/4 内置 CertificatePinner
    try {
        var CertificatePinner = Java.use('okhttp3.CertificatePinner');
        CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function (host, peers) {
            console.log('[+] OkHttp CertificatePinner.check bypassed: ' + host);
            return;
        };
    } catch (e) {}

    // 2. 自定义 X509TrustManager.checkServerTrusted
    try {
        var TrustManagerImpl = Java.use('com.android.org.conscrypt.TrustManagerImpl');
        TrustManagerImpl.verifyChain.implementation = function (untrusted, holdHost, host, clientAuth, ocspData, tlsSctData) {
            console.log('[+] TrustManagerImpl.verifyChain bypassed: ' + host);
            return untrusted;
        };
    } catch (e) {}

    // 3. HostnameVerifier 全过
    var HostnameVerifier = Java.use('javax.net.ssl.HostnameVerifier');
    // 用 objection 自带模板补完...
});
```


```bash
objection --gadget com.target.app explore -s "android sslpinning disable"
```


```text
1. 抓包 → 看是哪类错误（CertPin / Hostname / TrustManager）
2. jadx 搜关键类（CertificatePinner / X509TrustManager / HostnameVerifier）
3. 优先 objection 一键 → 不行再 frida-multiple-unpinning → 再不行手写
4. 若有反 Frida 检测 → 切 frida-gadget 或 zygisk
5. Flutter 应用单独处理（hook libflutter.so 的 ssl_verify_peer_cert）
```


- Kali / Windows + adb + frida-tools 16.x
