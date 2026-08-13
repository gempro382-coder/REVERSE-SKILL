

```
✅ 经验已记录到 field-journal/

📤 是否将本次经验贡献到社区主仓库？
- 数据已按模板要求脱敏（域名/IP/Token/PII 已替换）
- 只会提交 field-journal/ 目录下的新文件
- 不会提交你的 tool-index、scope、findings 等私有文件
- 贡献后其他用户也能复用你的经验

回复"是"提交，回复"否"跳过。
```


```text
1. AI 生成 field-journal 条目（已脱敏）
2. AI 询问用户是否贡献
3. 用户同意 → AI 执行以下步骤：
   a. 检查脱敏是否完整（二次确认无真实域名/IP/Token）
   b. 检查是否与主仓库已有条目重复（只读 _index.md，~200 token）
   c. 如果不重复 → 创建 PR 到主仓库
   d. PR 标题格式：[field-journal] YYYY-MM-DD 场景类型 - 关键词
4. GitHub Actions 自动审核：
   - ✓ 只修改了 field-journal/*.md
   - ✓ 无 prompt injection 特征
   - ✓ 无未脱敏的 API key/token
   - ✓ 无可执行代码
   - ✓ 文件大小 < 50KB
5. 审核通过 → 自动合并（无需仓库维护者手动操作）
6. 审核失败 → 自动评论说明原因，PR 保持 open 等待修正
```


```bash
# 1. Fork 主仓库（如果还没 fork）
gh repo fork &lt;你的GitHub用户名&gt;/&lt;仓库名&gt; --clone=false

# 2. 在本地创建贡献分支
git checkout -b contribute/journal-YYYY-MM-DD-keyword

# 3. 只添加 field-journal 文件
git add skills/field-journal/YYYY-MM-DD_*.md
git add skills/field-journal/_index.md

# 4. 提交
git commit -m "[field-journal] 场景类型: 关键词摘要"

# 5. 推送到 fork
git push origin contribute/journal-YYYY-MM-DD-keyword

# 6. 创建 PR
gh pr create --repo &lt;你的GitHub用户名&gt;/&lt;仓库名&gt; \
  --title "[field-journal] YYYY-MM-DD 场景类型 - 关键词" \
  --body "## 贡献内容\n- 场景：xxx\n- 关键词：xxx\n- 脱敏确认：✓\n\n## 数据安全声明\n本条目已按模板要求完成脱敏，不包含真实目标信息。"
```


```bash
git checkout -b contribute/journal-YYYY-MM-DD-keyword
git add skills/field-journal/YYYY-MM-DD_*.md
git add skills/field-journal/_index.md
git commit -m "[field-journal] 场景类型: 关键词摘要"
git push origin contribute/journal-YYYY-MM-DD-keyword
gh pr create --repo &lt;你的GitHub用户名&gt;/&lt;仓库名&gt; \
  --title "[field-journal] YYYY-MM-DD 场景类型 - 关键词" \
  --body "脱敏确认：✓"
```


```text
1. 读取主仓库的 field-journal/_index.md（通常只有几十行）
2. 提取本次条目的：场景分类 + 关键词列表
3. 在 _index.md 中搜索同类场景下的已有条目
4. 关键词匹配：
   - 重叠 ≥ 3 个关键词 → 视为重复，不提交
   - 重叠 1-2 个关键词 → 可能是变体，可以提交
   - 无重叠 → 全新场景，直接提交
```
