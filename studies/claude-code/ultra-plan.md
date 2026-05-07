---
title: Claude Code Ultra Plan
description: 三探索 + 一评审的并行探索机制，模拟人类团队的头脑风暴 + 独立评审流程。
tags: [claude-code, ultra-plan, multi-agent, parallel]
---

# Claude Code Ultra Plan

## 模式概览

```
输入问题
    ↓
┌──────────┬──────────┬──────────┐
│ Explorer │ Explorer │ Explorer │  ← 并行执行（三种策略）
│  保守     │  激进     │  创新     │
└────┬─────┴────┬─────┴────┬─────┘
     └──────────┼──────────┘
                ↓
         ┌────────────┐
         │   Critic   │  ← 独立评审（只读）
         │   评审员    │
         └──────┬─────┘
                ↓
         最优方案 + 评审意见
```

---

## 伪代码实现

```python
class UltraPlan:
    def solve(self, problem: str) -> str:
        # 1. 启动 3 个 Explorer Agent（并行）
        explorers = [
            SubAgent(config=load("explorer-conservative.md")),
            SubAgent(config=load("explorer-aggressive.md")),
            SubAgent(config=load("explorer-innovative.md")),
        ]

        solutions = []
        with ThreadPoolExecutor(max_workers=3) as executor:
            futures = [
                executor.submit(exp.execute, problem)
                for exp in explorers
            ]
            for future in futures:
                solutions.append(future.result())

        # 2. 启动 Critic Agent（独立评审，避免自我偏差）
        critic = SubAgent(config=load("critic.md"), read_only=True)
        evaluation = critic.execute(
            f"请客观评审以下三个方案，选出最优：\n"
            f"方案A（保守）：{solutions[0]}\n"
            f"方案B（激进）：{solutions[1]}\n"
            f"方案C（创新）：{solutions[2]}"
        )

        # 3. 提取最优方案
        best = self._extract_best(evaluation, solutions)
        return f"{best}\n\n【评审意见】{evaluation}"
```

---

## 为什么需要独立 Critic

**自我偏差问题**：让同一个模型评审自己的输出容易产生偏见——模型倾向于认为自己生成的方案更好。

**解决方案**：Critic Agent 是**独立实例**，只读权限，不参与方案生成。它的唯一任务是客观比较三个外部方案。

---

## 适用场景

| 场景 | 价值 |
|------|------|
| **架构设计** | 多方案对比，避免陷入单一思路 |
| **代码重构** | 不同重构策略评估 |
| **技术选型** | 多维度打分，量化决策依据 |
| **算法设计** | 多种实现方案的性能对比 |

---

## 与单一 Agent 的对比

| 维度 | 单一 Agent | Ultra Plan（3E + 1C） |
|------|-----------|----------------------|
| **方案多样性** | 低（一种思路） | 高（三种策略） |
| **决策质量** | 依赖单次推理 | 多方案对比 + 独立评审 |
| **Token 成本** | 低 | 高（约 4 倍） |
| **耗时** | 短 | 较长（并行但仍需等待全部完成） |
| **适用场景** | 简单任务 | 复杂决策、高 stakes 场景 |
