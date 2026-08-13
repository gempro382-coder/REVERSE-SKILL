

- lead_role: lead
- specialists: [cre, cie]


|------|-------------|----------------|--------------|

- path_type: solve


|------|------|---------|------|


```bash
# 差分电池(三件套: runner / compare / CLI 桥)
bash run_diff.sh                # 核心算法 194/194
python3 diff_lights.py          # 灯光 120/120
python3 diff_render.py          # 打包 120/120
python3 diff_ops.py             # operations 240/240
# 反汇编
otool -tV -p 0x22660 <orig>.so
# 构建
cargo build --workspace --target x86_64-apple-darwin --release
```


- OS: macOS (Darwin), Apple Silicon + Rosetta
