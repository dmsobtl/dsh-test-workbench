# Skill: 测试会话诊断

## 触发条件

当用户要求分析上次测试的效率、排查为什么测试花了很长时间、或者想优化测试流程时使用。

## 工作流程

1. 找到最近的会话 JSONL 文件（通常在 `~/.dsh/sessions/` 下）
2. `analyze_session` 分析该会话
3. 重点关注：
   - **redundant calls**：是否有重复操作（说明定位策略有问题）
   - **tool errors**：哪些工具调用失败了（说明选择器或环境有问题）
   - **token efficiency**：token 利用率低说明 prompt 可能需要优化
   - **excessive steps**：单 turn 超过 10 步说明可能陷入循环

## 常见模式与建议

| 问题模式 | 表现 | 建议 |
|---------|------|------|
| 选择器不稳定 | 同一元素 click 多次失败 | 改用 role/testid 选择器 |
| 页面未加载完 | verify 连续失败后成功 | launch 后加等待条件 |
| 死循环 | 同一操作重复 5+ 次 | 检查退出条件 |
| token 浪费 | efficiency < 0.05 | 简化 system prompt |

## 对比分析

如果有多次运行的会话，用 `compare_sessions` 对比：
- 确认优化是否有效（steps 减少、errors 减少）
- 检测是否引入了新的回归（原本通过现在失败）

## 产出

- 问题清单 + 严重级别
- 优化建议
- 与上次的对比 delta
