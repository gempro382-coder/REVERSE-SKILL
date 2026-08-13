

---


```text
1. 从 APK 中提取 .so 文件
   unzip app.apk lib/arm64-v8a/*.so -d extracted/

2. 确认架构和基本信息
   file libxxx.so
   rabin2 -I libxxx.so

3. 找 JNI 入口
   - 搜索 JNI_OnLoad（动态注册）
   - 搜索 Java_com_xxx_yyy（静态注册）
   - nm -D libxxx.so | grep -i java

4. IDA/Ghidra 加载分析
   - 导入 JNI 头文件（jni.h 类型）
   - 标注 JNIEnv* 参数
   - 找 RegisterNatives 调用（动态注册的函数表）

5. 定位关键逻辑
   - 从 Java 层 native 方法名追踪
   - 从字符串（密钥、URL、错误信息）交叉引用
   - 从 crypto 库函数（AES/MD5/SHA）调用追踪
```


```c
// 静态注册：函数名 = Java_包名_类名_方法名
JNIEXPORT jstring JNICALL Java_com_example_app_Security_getSign(
    JNIEnv *env, jobject thiz, jstring input) { ... }

// 动态注册：在 JNI_OnLoad 中调用 RegisterNatives
static JNINativeMethod methods[] = {
    {"getSign", "(Ljava/lang/String;)Ljava/lang/String;", (void*)native_getSign},
};

JNIEXPORT jint JNI_OnLoad(JavaVM *vm, void *reserved) {
    JNIEnv *env;
    vm->GetEnv((void**)&env, JNI_VERSION_1_6);
    jclass clazz = env->FindClass("com/example/app/Security");
    env->RegisterNatives(clazz, methods, sizeof(methods)/sizeof(methods[0]));
    return JNI_VERSION_1_6;
}
```


```text
1. 导入 JNI 类型库
   File → Load File → Parse C Header → jni.h

2. 标注第一个参数为 JNIEnv*
   右键参数 → Set type → JNIEnv*
   这样 env->FindClass / env->GetMethodID 等调用会自动识别

3. 找 RegisterNatives
   搜索对 JNIEnv vtable offset 0x35C (ARM64) 的调用
   → 第三个参数是 JNINativeMethod 数组
   → 从数组中提取所有 native 函数地址
```

---


```javascript
// Hook libc 函数
Interceptor.attach(Module.findExportByName("libc.so", "open"), {
    onEnter: function(args) {
        this.path = args[0].readUtf8String();
        console.log("[open] " + this.path);
    },
    onLeave: function(retval) {
        if (this.path.includes("su") || this.path.includes("magisk")) {
            console.log("[open] Blocked root check: " + this.path);
            retval.replace(-1);  // 返回失败
        }
    }
});

// Hook 自定义 SO 中的函数
var base = Module.findBaseAddress("libsecurity.so");
var targetFunc = base.add(0x1234);  // 偏移地址
Interceptor.attach(targetFunc, {
    onEnter: function(args) {
        console.log("arg0: " + args[0].readUtf8String());
    },
    onLeave: function(retval) {
        console.log("return: " + retval.readUtf8String());
    }
});
```


```javascript
Java.perform(function() {
    // Hook 实例方法
    var Security = Java.use("com.example.app.Security");
    Security.getSign.implementation = function(input) {
        console.log("[getSign] input: " + input);
        var result = this.getSign(input);  // 调用原方法
        console.log("[getSign] output: " + result);
        return result;
    };

    // Hook 构造函数
    Security.$init.overload('java.lang.String').implementation = function(key) {
        console.log("[Security.<init>] key: " + key);
        this.$init(key);
    };

    // Hook 重载方法
    Security.encrypt.overload('java.lang.String', 'int').implementation = function(data, mode) {
        console.log("[encrypt] data=" + data + " mode=" + mode);
        return this.encrypt(data, mode);
    };
});
```


```javascript
// 搜索内存中的字符串
Process.enumerateModules().forEach(function(module) {
    if (module.name === "libtarget.so") {
        Memory.scan(module.base, module.size, "48 65 6C 6C 6F", {  // "Hello"
            onMatch: function(address, size) {
                console.log("Found at: " + address);
            }
        });
    }
});

// 修改内存（patch 指令）
var addr = Module.findBaseAddress("libsecurity.so").add(0x5678);
Memory.patchCode(addr, 4, function(code) {
    var writer = new Arm64Writer(code, {pc: addr});
    writer.putNop();  // 替换为 NOP
    writer.flush();
});
```

---


```javascript
// Frida 通用 SSL Pinning 绕过
// 来源: https://github.com/0xCD4/SSL-bypass
Java.perform(function() {
    // 1. TrustManager 绕过
    var TrustManager = Java.registerClass({
        name: 'com.custom.TrustManager',
        implements: [Java.use('javax.net.ssl.X509TrustManager')],
        methods: {
            checkClientTrusted: function(chain, authType) {},
            checkServerTrusted: function(chain, authType) {},
            getAcceptedIssuers: function() { return []; }
        }
    });

    // 2. SSLContext 替换
    var SSLContext = Java.use('javax.net.ssl.SSLContext');
    var sslContext = SSLContext.getInstance("TLS");
    sslContext.init(null, [TrustManager.$new()], null);

    // 3. OkHttp CertificatePinner 绕过
    try {
        var CertificatePinner = Java.use('okhttp3.CertificatePinner');
        CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function() {};
    } catch(e) {}
});
```


|------|---------|
| React Native | Hook `OkHttpClientProvider` |
| WebView | Hook `WebViewClient.onReceivedSslError` |


```javascript
// Flutter SSL Pinning 绕过（需要找到 ssl_verify_peer_cert 函数）
var flutter_lib = Module.findBaseAddress("libflutter.so");
// 搜索 ssl_verify_peer_cert 的特征码
var pattern = "FF 03 05 D1 FD 7B 0F A9";  // ARM64 特征
Memory.scan(flutter_lib, Module.findModuleByName("libflutter.so").size, pattern, {
    onMatch: function(address) {
        Interceptor.replace(address, new NativeCallback(function() {
            return 0;  // 返回成功
        }, 'int', []));
    }
});
```

---


|---------|---------|
| SafetyNet/Play Integrity | Magisk Hide / Zygisk + Shamiko |


```javascript
Java.perform(function() {
    // Hook File.exists
    var File = Java.use("java.io.File");
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        var blacklist = ["su", "Superuser", "magisk", "busybox", "xposed"];
        for (var i = 0; i < blacklist.length; i++) {
            if (path.toLowerCase().includes(blacklist[i])) {
                return false;
            }
        }
        return this.exists();
    };

    // Hook System.getProperty
    var System = Java.use("java.lang.System");
    System.getProperty.overload('java.lang.String').implementation = function(key) {
        if (key === "ro.debuggable" || key === "ro.secure") {
            return "1";
        }
        return this.getProperty(key);
    };
});
```

---


|------|---------|---------|


```text
方法 1: FART（ART 环境脱壳）
- 刷入 FART ROM 或使用 Frida 版 FART
- 自动 dump 所有 ClassLoader 加载的 dex

方法 2: Frida DEX Dump
- frida -U -f com.target.app -l dex_dump.js
- 在 DexFile::OpenMemory 处 hook，dump 内存中的 dex

方法 3: BlackDex
- 免 root 脱壳工具
- 直接安装 BlackDex APK，选择目标应用脱壳

方法 4: 手动 dump
- 用 Frida 枚举所有 ClassLoader
- 找到应用的 ClassLoader → 获取 DexFile 对象
- 读取 dex 内存区域并保存
```


```javascript
Java.perform(function() {
    Java.enumerateClassLoaders({
        onMatch: function(loader) {
            try {
                var dexFiles = loader.getDexFileList();
                console.log("ClassLoader: " + loader);
                console.log("  DEX files: " + dexFiles);
            } catch(e) {}
        },
        onComplete: function() {}
    });
});
```

---


### React Native

```text
1. 解压 APK → assets/index.android.bundle（JS 代码）
2. 格式化 JS → 搜索 API 地址、密钥、签名逻辑
3. 如果有 Hermes 字节码（.hbc 文件）→ 用 hermes-dec 反编译
4. Hook: 用 Frida hook Java 层的 ReactBridge
```

### Flutter

```text
1. Flutter 代码编译为 libapp.so（Dart AOT）
2. 无法直接反编译为 Dart 源码
3. 分析方法：
   - reFlutter 工具：patch libflutter.so 获取 snapshot
   - Doldrums：解析 Dart snapshot 恢复类/函数信息
   - Frida hook libflutter.so 中的关键函数
4. 网络分析：Flutter 不走系统代理，需要特殊处理 SSL
```

---


|------|------|------|

---


|------|------|------|
