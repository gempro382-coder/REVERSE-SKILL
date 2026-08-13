

- Hook `TrustManagerImpl.verifyChain()`
- Hook `TrustManagerImpl.checkTrustedRecursive()`


- Hook `Debug.isDebuggerConnected()`


```bash
# 前置条件
pip install frida-tools
adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell su -c /data/local/tmp/frida-server &

# 注入目标 APP
frida -U -f com.example.app -l FridaBypassKit.js
```
