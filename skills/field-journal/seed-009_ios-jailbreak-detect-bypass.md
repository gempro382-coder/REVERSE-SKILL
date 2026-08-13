

```javascript
// 拦截 NSFileManager fileExistsAtPath 检测越狱目录
var NSFileManager = ObjC.classes.NSFileManager;
Interceptor.attach(NSFileManager['- fileExistsAtPath:'].implementation, {
    onEnter: function (args) {
        var path = ObjC.Object(args[2]).toString();
        var jbPaths = [
            '/Applications/Cydia.app',
            '/Library/MobileSubstrate/MobileSubstrate.dylib',
            '/bin/bash', '/usr/sbin/sshd',
            '/etc/apt', '/private/var/lib/apt/'
        ];
        if (jbPaths.indexOf(path) !== -1) {
            this.shouldFake = true;
            console.log('[+] Hide JB path: ' + path);
        }
    },
    onLeave: function (retval) {
        if (this.shouldFake) retval.replace(0);
    }
});

// 拦截 fork() —— 越狱机能 fork，非越狱机返回 -1
var fork = Module.findExportByName(null, 'fork');
Interceptor.replace(fork, new NativeCallback(function () {
    return -1;
}, 'int', []));
```


```bash
frida-ios-dump -l com.target.app
# 输出 Payload/TargetApp.app + 已脱壳 Mach-O
```


```text
1. 越狱环境准备（Dopamine 16.x / palera1n 旧版）
2. frida-ios-dump 脱壳
3. otool / class-dump 看类层级
4. objection 起 console
5. ios jailbreak disable
6. ios sslpinning disable
7. mitmproxy 抓包（系统证书 + 信任设置双开）
8. 关键逻辑找完后用 IDA / Hopper 静态深挖
```


- frida-server-ios: 16.x
