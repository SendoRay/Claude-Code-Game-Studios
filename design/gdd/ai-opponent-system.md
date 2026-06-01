# AI Opponent System (AI对手系统)

> **Status**: Draft Complete
> **Author**: user + agents
> **Last Updated**: 2026-06-01
> **Implements Pillar**: Pillar 4 (Readable Pressure — AI意图可预见)

## Overview

AI对手系统控制非玩家势力的行为：扩张领土、组建Stack、部署进攻/防御。AI在每回合的INTEL阶段向玩家展示意图预告（如"曹操正在集结兵力，目标可能是南阳"），确保玩家的压力是可预见的而非随机伏击。AI不追求"强得无法战胜"，而是提供有节奏的、可阅读的挑战。

## Player Fantasy

"情报显示曹操在向南阳集结重兵——我有2回合准备时间。是加固防线还是先发制人？AI的意图清晰可见，但我的资源不够同时应对所有方向。"

## Detailed Design

### Core Rules

#### 1. AI Architecture (AI架构)

AI使用简单的**优先级评分系统**（非行为树/状态机，降低复杂度）：

每回合AI执行：
1. 评估所有可进攻的区域，计算战略价值得分
2. 评估自身Stack战力，决定进攻/防御/扩军
3. 选择得分最高的行动执行
4. 生成意图预告供INTEL阶段展示

#### 2. AI Decision Scoring (决策评分)

```
action_score(action) = base_value + modifiers

进攻评分:
  base = target_region.income_value
  + (20 if would_complete_province)
  + (10 × weakness_factor)     # 防守方战力越低分越高
  - (15 × risk_factor)          # 己方损失风险

防御评分:
  base = threatened_region.income_value
  + (30 if region_completes_province)  # 保住全州更重要
  + (10 × threat_level)

扩军评分:
  base = 20
  + (10 if total_power < player_total_power)  # 落后时优先扩军
```

#### 3. Intent Signaling (意图预告 — Pillar 4核心)

AI在做出决策后生成预告信息：

| 意图类型 | 展示内容 | 提前回合数 |
|----------|----------|------------|
| **集结** | "曹操在[区域]集结兵力" | 1-2回合前 |
| **进攻** | "曹操的军队正向[区域]推进" | 当回合（已确定目标） |
| **扩军** | "曹操势力正在扩充军备" | 当回合 |
| **防御** | "曹操加强了[区域]的防守" | 当回合 |

**规则**：
- AI的实际行动必须与预告一致（不能假情报——MVP简化）
- 预告不显示AI Stack的具体组成（只有方向和意图）
- 玩家可通过侦查计策获取更多细节

#### 4. AI Factions (AI势力)

每个剧本定义1-3个AI势力，各有不同倾向：

| 属性 | 说明 |
|------|------|
| `faction_id` | 势力标识 |
| `faction_name` | 显示名称（如"曹操势力"） |
| `aggression` | 进攻倾向（0.0-1.0）。高=频繁进攻，低=偏防守 |
| `expansion_priority` | 扩张优先级。决定AI是优先扩地盘还是强化已有地盘 |
| `difficulty_tier` | 难度等级（影响AI资源和决策质量） |

#### 5. AI Economy (AI经济简化)

AI不使用与玩家完全相同的经济规则：
- AI每回合获得固定资源量（基于难度+控制区域数），不精确模拟粮草收支
- AI的Stack组成由难度模板决定（不逐卡招募）
- 目的：减少AI系统复杂度，同时保持挑战感

#### 6. Difficulty Scaling (难度缩放)

| 难度 | AI资源倍率 | 决策质量 | 进攻频率 |
|------|-----------|----------|----------|
| Easy | 0.8x | 有时做次优选择 | 低（3-4回合一次） |
| Normal | 1.0x | 通常做最优选择 | 中（2-3回合一次） |
| Hard | 1.3x | 总是做最优选择 | 高（1-2回合一次） |

---

### States and Transitions

AI每回合状态循环：

```
EVALUATE → DECIDE → SIGNAL → EXECUTE

EVALUATE: 扫描地图，评估所有可能行动
DECIDE: 选择最高得分行动
SIGNAL: 生成意图预告（供INTEL阶段）
EXECUTE: 在RESOLVE阶段执行（与玩家同时结算）
```

---

### Interactions with Other Systems

| 系统 | 交互方式 |
|------|----------|
| **大地图系统** | 读取区域控制权、邻接图、战略价值 |
| **回合流程系统** | INTEL阶段展示AI意图；回合推进触发AI决策 |
| **自动战斗系统** | AI Stack参与战斗（使用相同规则） |
| **卡牌数据系统** | AI Stack使用相同的卡牌数据结构 |

## Formulas

### 战略价值评分

```
attack_score(target, ai_stack):
  value = target.income_value
  if would_complete_province(target, ai): value += 20
  
  defender_power = estimate_defender_power(target)
  my_power = stack_power(ai_stack)
  
  if my_power > defender_power × 1.3:
    value += 15  # 有把握的进攻加分
  elif my_power < defender_power × 0.7:
    value -= 30  # 明显劣势大幅减分
  
  return value
```

## Edge Cases

| # | 边界情况 | 处理规则 |
|---|----------|----------|
| 1 | AI所有Stack被消灭 | AI进入"重建"模式：不进攻，专注扩军，2-3回合后恢复 |
| 2 | AI只剩1个区域 | AI全力防守该区域，不进攻 |
| 3 | 多个AI势力互相接壤 | AI之间也会互相攻击（基于相同评分逻辑）。玩家可利用AI内斗 |
| 4 | AI意图预告后玩家加固了防守 | AI仍执行原计划（MVP不做"读心"——AI不根据玩家反应改变已确定的行动） |
| 5 | AI被完全消灭（0区域） | 该AI势力从游戏中移除 |

## Dependencies

**上游依赖**：

| 系统 | 依赖内容 |
|------|----------|
| 大地图系统 | 地图拓扑、区域控制权 |
| 回合流程系统 | 回合推进信号、INTEL阶段 |

**下游依赖方**：无

## Tuning Knobs

| 参数名 | 当前值 | 安全范围 | 影响 |
|--------|--------|----------|------|
| `AI_AGGRESSION_DEFAULT` | 0.5 | 0.2-0.8 | 默认进攻倾向 |
| `AI_RESOURCE_MULTIPLIER` | 1.0 | 0.5-2.0 | AI资源倍率（难度核心旋钮） |
| `INTENT_SIGNAL_ADVANCE` | 1回合 | 0-2回合 | 意图提前多久展示。=0则无预警（违反Pillar 4） |
| `AI_REBUILD_TURNS` | 3 | 2-5 | Stack全灭后重建所需回合 |
| `PROVINCE_COMPLETE_BONUS` | 20 | 10-40 | 集齐全州对AI决策的权重 |

## Visual/Audio Requirements

- INTEL阶段：AI意图以地图上的箭头/图标展示
- AI势力用独特颜色标识（曹操=蓝，孙权=绿，袁绍=紫等）
- AI行动结算时地图上显示移动动画

## UI Requirements

- INTEL面板：列出所有AI势力的当回合意图
- 地图上显示AI威胁方向箭头
- AI势力信息面板：势力名、控制区域数、估算战力

## Acceptance Criteria

| # | 验收条件 | 验证方式 |
|---|----------|----------|
| 1 | AI每回合产生有效行动（不卡死） | 集成测试：模拟50回合 |
| 2 | AI意图预告与实际行动一致 | 单元测试 |
| 3 | AI不攻击明显劣势的目标（power < 0.7x防守方） | 行为测试 |
| 4 | AI之间互相攻击 | 集成测试 |
| 5 | 难度缩放正确影响AI资源和决策 | 单元测试 |
| 6 | AI全灭后正确进入重建模式 | 单元测试 |

## Open Questions

| # | 问题 | 何时决定 |
|---|------|----------|
| 1 | AI是否需要"性格"（曹操好战、刘备保守、孙权水战）？ | Alpha阶段根据叙事需求决定 |
| 2 | AI是否应该利用地形（优先在有利地形防守）？ | 实现时决定复杂度 |
| 3 | 多AI之间是否需要"联盟"/"外交"机制？ | Full Vision阶段 |
