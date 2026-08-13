

   ```bash
   unzip target.apk -d apk
   ls apk/assets/bin/Data/Managed/Metadata/
   ```
   ```bash
   Il2CppDumper libil2cpp.so global-metadata.dat output/
   ```


|------|------|---------|------|


```typescript
// hook.ts
import "frida-il2cpp-bridge";

Il2Cpp.perform(() => {
    const Assembly = Il2Cpp.domain.assembly("Assembly-CSharp").image;

    // hook static method
    const PlayerData = Assembly.class("PlayerData");
    PlayerData.method("AddCoin").implementation = function (n: number) {
        console.log("[+] AddCoin called with:", n);
        return this.method("AddCoin").invoke(99999); // 改成 99999
    };

    // hook instance method
    const Purchase = Assembly.class("Purchase");
    Purchase.method("VerifyReceipt").implementation = function () {
        console.log("[+] VerifyReceipt → always true");
        return true;
    };
});
```

```bash
# 编译 + 注入
npm install frida-il2cpp-bridge
frida-compile hook.ts -o hook.js
frida -U -f com.target.game -l hook.js --no-pause
```


```text
1. 打开 libil2cpp.so，跑 il2cpp_load_metadata.py
2. 跳到 dump.cs 中 IsPurchaseValid 对应的偏移
3. 函数开头改成 MOV W0, #1; RET（ARM64）
4. Apply Patches → Save → 替换回 APK → 重签名
```


```text
1. 确认 IL2CPP（看 lib/abi 下有无 libil2cpp.so）
2. 找到 metadata（assets/bin/Data/Managed/Metadata/global-metadata.dat 或被加密）
3. Il2CppDumper / Inspector 还原
4. IDA + 脚本带回元信息
5. dump.cs 搜业务关键词
6. 选择 patch 还是 hook
7. 验证（启动 + 实际场景）
```


```text
1. Frida 在 fopen/open 系调用上挂钩，看谁读 global-metadata.dat
2. 在 mmap/read 后 dump 内存里已解密的元数据
3. 把 dump 出来的内存当 metadata 喂给 Il2CppDumper
```


- frida-tools 16.x, frida-il2cpp-bridge 0.9+
