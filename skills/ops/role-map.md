

```text
1. 输出 PRIMARY（master-route）+ lead_role=lead
2. 写 scope.md（ops/scope-contract）
3. 指定 specialist_roles[] 与 handoff 条件
4. 每阶段结束：更新 timeline + workitems；决定继续/换角色/出报告
5. 禁止跳过 scope 直接 cpe 扫生产
```


```text
同一会话内：
  [lead] 规划
  [cie] 执行侦察 skill
  [cpe] 切换 pentest-tools
  …
输出时用角色前缀标签，方便 timeline 检索：
  [cpe] nuclei high findings → E-003
```


## MUST NOT
