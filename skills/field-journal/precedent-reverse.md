
>

---


```text
□ jadx -d output_dir/ target.apk                          — APK 反编译，无数次
□ jadx --no-res --no-dex target.apk                        — 仅反编译 dex，跳过资源
□ apktool d target.apk -o unpacked/                        — APK 解包到 smali
□ apktool d -r target.apk -o unpacked/                     — 解包但跳过资源反编译
□ apktool b unpacked/ -o repacked.apk                      — 重新打包
□ jarsigner -keystore debug.keystore -storepass android repacked.apk androiddebugkey — 签名
□ adb install repacked.apk                                 — 安装到模拟器/真机
□ adb logcat | grep "frida\|hook\|SSL"                     — 过滤日志
□ frida -U -f com.example.app -l hook.js --no-pause        — Frida spawn 模式注入
□ frida -U -n "app_name" -l hook.js                        — Frida attach 模式注入
□ frida-ps -U                                              — 列出 USB 设备进程
□ objection -g com.example.app explore                     — Objection 探索模式
□ android sslpinning disable                               — Objection 禁用 SSL Pin（通用绕过）
□ android root disable                                     — Objection 禁用 Root 检测
□ android hooking list classes                             — Objection 枚举所有类
□ android hooking watch class com.example.ClassName         — Objection 监控类方法调用
□ jadx 搜索: "sign\|signature\|hmac\|md5\|sha\|encrypt\|decrypt\|AES\|RSA\|Base64\|token" — 定位签名/加密逻辑
□ apktool d 后直接 grep -r "native" smali/                  — 找 native 方法声明
□ grep -r "System.loadLibrary\|System.load" smali/         — 找 so 加载点
□ IDA Pro 打开 lib/*.so → 找 JNI_OnLoad → 静态注册/动态注册 → Frida hook native 函数
```


---


```text
□ ida64.exe target.so                                       — 打开 so/ELF/PE
□ ida64.exe -B target.so                                    — 自动批量分析（生成 .i64）
□ ida64.exe -A -S"script.py" target.so                      — 无头模式运行脚本
□ 快捷键: Shift+F12 → 字符串窗口 → 搜索 "http\|key\|secret\|encrypt\|decrypt\|AES\|RSA"
□ 快捷键: G → 跳转到地址
□ 快捷键: X → 查看交叉引用（谁调用了这个函数/数据）
□ 快捷键: F5 → 反编译（Hex-Rays）
□ 快捷键: N → 重命名函数/变量
□ 快捷键: Y → 修改类型
□ 快捷键: Ctrl+E → 导出数据
□ 快捷键: Shift+E → 导出为 C 数组
□ IDAPython: idc.get_func_name(ea) / idc.get_func_off_str(ea) / ida_xref.xrefsto(ea)
□ Ghidra: File → Import → 选文件 → 确认格式 → 双击 → Analysis → Auto Analyze
□ Ghidra: Window → Defined Strings → 搜索关键字
□ Ghidra: 右键 → References → Show References to Address
□ Ghidra: 右键 → Patch Instruction → 改汇编指令 → File → Export Program → 导出补丁后二进制
```


---


```javascript
// === 基础模板: Hook Java 方法 ===
Java.perform(function() {
    var TargetClass = Java.use("com.example.TargetClass");
    TargetClass.targetMethod.implementation = function(arg1, arg2) {
        console.log("[+] targetMethod called, arg1=" + arg1 + " arg2=" + arg2);
        var result = this.targetMethod(arg1, arg2);
        console.log("[+] targetMethod returned: " + result);
        return result;
    };
});

// === Hook Native 函数 ===
var targetModule = Process.findModuleByName("libtarget.so");
var targetAddr = Module.findExportByName("libtarget.so", "target_function");
// 或: var targetAddr = targetModule.base.add(0x12345); // 偏移
Interceptor.attach(targetAddr, {
    onEnter: function(args) { console.log("arg0=" + args[0].readCString()); },
    onLeave: function(retval) { console.log("ret=" + retval); }
});

// === Hook JNI NewStringUTF (抓 Java 传 native 的字符串) ===
var NewStringUTF = Module.findExportByName("libart.so", "NewStringUTF");
Interceptor.attach(NewStringUTF, {
    onEnter: function(args) { console.log("JNI NewStringUTF: " + args[1].readCString()); }
});

// === 绕过 SSL Pinning (通用) ===
var CertificateFactory = Java.use("javax.net.ssl.SSLContext");
// ... 信任所有证书的 TrustManager 注入

// === Hook 动态注册的 JNI 方法 ===
// 1. frida -U -f com.example.app -l enumerate_jni.js --no-pause
// 2. 找到 RegisterNatives 调用 → 获取方法表
// 3. 用 NativeFunction 包装 → Interceptor.attach
```


---


```text
# 快速侦察
$ file target.bin                                          — 确认文件类型
$ strings target.bin | grep -iE "http\|key\|flag\|secret\|AES\|RSA\|password" — 字符串侦察
$ rabin2 -I target.bin                                     — 二进制信息（arch/bits/nx/pie/canary）
$ rabin2 -z target.bin                                     — 数据段字符串
$ rabin2 -E target.bin                                     — 导出表
$ rabin2 -i target.bin                                     — 导入表
$ rabin2 -s target.bin                                     — sections
$ rabin2 -R target.bin                                     — relocations
$ rabin2 -l target.so                                      — 链接的库

# 反汇编
$ r2 -A target.bin                                         — 打开 + 自动分析
$ r2 -d target.bin                                         — 调试模式
[0x00400000]> aaaa                                         — 完整分析
[0x00400000]> afl                                          — 列出所有函数
[0x00400000]> afl~keyword                                  — 按名称过滤函数
[0x00400000]> s main                                       — 跳转到 main
[0x00400000]> pdf                                          — 反汇编当前函数
[0x00400000]> pdc                                          — 伪代码反编译
[0x00400000]> iz                                           — 字符串列表
[0x00400000]> axt 0x00401234                               — 找谁引用了这个地址
[0x00400000]> wx 0x90 @ 0x00401200                         — 在 0x00401200 写入 NOP（patch）
[0x00400000]> oo+                                          — 重新打开为可写（patch 后保存）

# Go 逆向专项
$ go version target.bin                                    — 检测 Go 版本
$ GoReSym -i target.bin -o symbols.json                    — 恢复 Go 符号
$ strings target.bin | grep "github.com\|gitlab.com"        — 找第三方包名

# Rust 逆向专项
$ strings target.bin | grep -E "^[a-z_]+::"                — 找模块路径
$ strings target.bin | grep "cargo"                         — 找 Cargo 信息
```

---


```text
□ IDA: Edit → Patch program → Change byte → 修改指令字节 → Edit → Patch program → Apply patches to input file
□ Ghidra: 右键 → Patch Instruction → 修改 → File → Export Program → 选择格式 → 导出
□ r2: wx <hex_bytes> @ <address>                           — 直接 patch
□ xxd target.bin | sed 's/xxxx/yyyy/' | xxd -r > patched.bin — 命令行 patch
□ echo -ne '\x90\x90\x90' | dd of=target.bin bs=1 seek=0x1234 conv=notrunc — 直接写入
□ apktool b 后 jarsigner 签名 → adb install
□ iOS: optool install -p "@executable_path/libFridaGadget.dylib" target.ipa — 注入 Frida Gadget
□ iOS: ldid -S target.app/target                            — 自签名（绕过 code sign）
```

---


---


```text
# .NET / C#
dnSpy.exe target.dll                                        — 直接看 IL 反编译源码
de4dot target.dll -o cleaned.dll                            — 去混淆（.NET Reactor/ConfuserEx等）
ILSpy target.dll                                            — 备选 .NET 反编译器

# Python
uncompyle6 target.pyc                                       — pyc 反编译
pycdc target.pyc                                            — 备选 pyc 反编译
strings target.pyc                                          — 快速看字符串引用

# Go
GoReSym -i target.exe -o symbols.json                       — 恢复 Go 符号（Windows PE）
go_parser target.bin --types                                — 恢复 Go 类型信息
strings target.bin | grep -E "^(main|github)\.\w+"          — 找包名

# Rust
strings target.bin | grep "^[a-z_][a-z0-9_]*::"             — Rust mangled 符号模式
cargo tree (如果有源码)                                      — 分析依赖

# WASM
wasm2c target.wasm -o target.c                              — WASM 转 C
wasm-decompile target.wasm                                  — WASM 伪代码
wasm-objdump -x target.wasm -j Import -j Export             — 看导入导出
strings target.wasm | grep -E "env\."                       — 找 JS 交互点

# Mach-O / iOS
class-dump target.app/target -o headers/                    — 导出 ObjC 类
jtool2 --analyze target                                     — Mach-O 分析
otool -l target | grep crypt                                — 检查 FairPlay 加密
install_name_tool -change old.dylib new.dylib target        — 修改 dylib 依赖
```

---


---
