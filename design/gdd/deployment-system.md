# Turn Deployment System (回合部署系统)

> **Status**: Draft Complete
> **Author**: user + agents
> **Last Updated**: 2026-06-01
> **Implements Pillar**: Pillar 2 (Decision Before Action — 部署是核心决策时刻)

## Overview

回合部署系统管理DEPLOY阶段中玩家将Stack分配到地图区域的操作。它连接堆叠系统（"组了什么队"）和大地图系统（"派到哪里"），是玩家每回合最重要的决策：进攻哪里、防守哪里、放弃哪里。系统在DEPLOY阶段开放，COMMIT后锁定。部署受邻接限制——Stack只能移动到当前位置的邻接区域。

## Player Fantasy

"关羽的全骑兵Stack在襄阳（城池），但我想进攻江夏（江河）触发水战地利。然而北边南阳被AI威胁……我是南下进攻还是北上防守？——这就是每回合的核心抉择。"

## Detailed Design

### Core Rules

#### 1. Deployment Actions (部署动作)

DEPLOY阶段中，玩家对每个Stack可执行以下指令：

| 指令 | 说明 | 限制 |
|------|------|------|
| **进攻** | 移动到邻接的敌方/中立区域 | 目标必须邻接当前位置 |
| **移动** | 移动到邻接的己方区域 | 目标必须邻接当前位置 |
| **驻守** | 留在当前区域不动 | 无限制（默认行为） |
| **休息** | 留在当前位置，加速武将恢复 | 该Stack本回合不参与战斗 |

#### 2. Movement Rules (移动规则)

- 每回合每个Stack最多移动1个区域（邻接限制）
- 不能跳过区域（无跨区域移动）
- 进攻 = 移动到非己方区域 → RESOLVE阶段触发战斗
- 移动 = 移动到己方区域 → 无战斗，纯重新定位

#### 3. Stack Location (Stack位置)

每个Stack有一个当前位置（region_id）：
- 新组建的Stack初始位置 = 玩家选择的己方区域（首次部署）
- 已有Stack保持上回合结束时的位置
- 休息中的Stack不能被进攻选中（驻守但不应战→由其他驻军应战）

#### 4. Multiple Stacks per Region (同区域多Stack)

- 一个区域可以有多个己方Stack驻守
- 进攻时每个Stack独立战斗（不联合作战，MVP限制）
- 防守时：如果区域有多个防守Stack，AI进攻按顺序依次与每个Stack战斗

#### 5. Deployment Preview (部署预览)

部署前显示预测信息：
- 目标区域地形和效果（哪些技能会触发）
- 敌方存在提示（"该区域有敌军驻守"，但不显示具体Stack组成）
- 战力对比指示器（优势/均势/劣势，基于可见信息估算）

---

### States and Transitions

Stack部署状态：

```
unassigned → [分配到区域] → assigned(region_id, action)
assigned → [取消] → unassigned （DEPLOY阶段内可自由取消）
assigned → [COMMIT确认] → locked
locked → [COMMIT取消] → assigned （回到DEPLOY）
locked → [RESOLVE开始] → executing
```

---

### Interactions with Other Systems

| 系统 | 交互方式 |
|------|----------|
| **卡牌堆叠系统** | 获取ready/full状态的Stack列表 |
| **大地图系统** | 读取邻接图、区域控制权、区域驻军信息 |
| **回合流程系统** | DEPLOY阶段开启/锁定部署操作 |
| **自动战斗系统** | 提供每个区域的攻防双方Stack匹配 |
| **地形修正系统** | 部署预览时查询地形效果 |

## Formulas

### 可达区域

```
reachable_regions(stack) → set[region]:
  current = stack.current_region
  return current.adjacent  # 直接邻接的区域（移动距离=1）
```

### 战力预估（部署预览）

```
estimated_advantage(stack, target_region):
  my_power = preview_power(stack)  # 含地形修正预估
  enemy_power = estimated_enemy_power(target_region)  # 模糊估计
  ratio = my_power / max(enemy_power, 1)
  if ratio > 1.3: return "优势"
  if ratio > 0.7: return "均势"
  return "劣势"
```

## Edge Cases

| # | 边界情况 | 处理规则 |
|---|----------|----------|
| 1 | 玩家不部署任何Stack | 允许。所有Stack保持驻守。回合正常推进 |
| 2 | Stack所在区域被敌方占领（上回合失败后Stack还在） | Stack被迫撤退到最近的己方邻接区域。如果没有→Stack被俘（武将exhausted，兵种destroyed） |
| 3 | 同一区域多个玩家Stack同时进攻不同目标 | 允许。每个Stack独立执行各自的进攻指令 |
| 4 | 进攻路径经过敌方区域 | 不允许。每回合只能移动1格，必须先攻占中间区域 |
| 5 | 休息中的Stack所在区域被进攻 | 休息Stack不参与防御战斗。如果区域无其他防守Stack且被攻占，休息Stack被迫撤退 |
| 6 | DEPLOY阶段中招募新兵后立即组Stack并部署 | 允许。招募、组Stack、部署都在DEPLOY阶段完成 |

## Dependencies

**上游依赖**：

| 系统 | 依赖内容 |
|------|----------|
| 卡牌堆叠系统 | ready/full状态的Stack列表 |
| 大地图系统 | 邻接图、区域控制权 |
| 回合流程系统 | DEPLOY阶段信号 |

**下游依赖方**：无（叶节点系统）

## Tuning Knobs

| 参数名 | 当前值 | 安全范围 | 影响 |
|--------|--------|----------|------|
| `MOVE_RANGE` | 1 | 1-2 | 每回合移动距离。=2则机动性大增，战略更灵活但也更混乱 |
| `REST_RECOVERY_BONUS` | +50% HP恢复 | 25-100% | 休息的恢复加速。太高→随时能恢复；太低→休息无意义 |
| `MAX_STACKS_PER_REGION` | 3 | 2-5 | 单区域最大Stack数。防止堆叠过多Stack到一个区域 |

## Visual/Audio Requirements

- 拖拽Stack到地图区域的交互（从Stack面板拖到地图区域）
- 有效目标区域高亮（绿色=己方可移动，红色=可进攻，灰色=不可达）
- 已部署的Stack在地图上显示移动箭头（从当前位置指向目标）
- 部署确认时所有箭头同时闪烁

## UI Requirements

- 左侧：Stack面板（所有可部署Stack）
- 中央：地图（拖放目标）
- 右侧：选中Stack/区域的详情
- 底部："结束部署"按钮 + Stack统计（"3/5 Stack已部署"）
- 部署预览面板：地形效果 + 战力预估 + 技能触发预测

## Acceptance Criteria

| # | 验收条件 | 验证方式 |
|---|----------|----------|
| 1 | Stack只能部署到邻接区域 | 单元测试 |
| 2 | 进攻非己方区域在RESOLVE触发战斗 | 集成测试 |
| 3 | 移动到己方区域不触发战斗 | 单元测试 |
| 4 | 驻守Stack正确参与区域防御 | 集成测试 |
| 5 | 休息Stack不参与战斗 | 单元测试 |
| 6 | DEPLOY阶段外不可修改部署 | 单元测试 |
| 7 | 部署预览正确显示地形效果和战力预估 | 手动测试 |

## Open Questions

| # | 问题 | 何时决定 |
|---|------|----------|
| 1 | 是否需要"强行军"机制（花粮草移动2格）？ | Alpha阶段评估 |
| 2 | 多Stack联合作战（2v1合并战力）是否在后续版本支持？ | Vertical Slice评估 |
