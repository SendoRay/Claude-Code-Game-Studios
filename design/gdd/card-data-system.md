# Card Data System (卡牌数据系统)

> **Status**: In Design
> **Author**: user + agents
> **Last Updated**: 2026-05-31
> **Implements Pillar**: Pillar 1 (Stack = Simplicity), Pillar 3 (Build-Up to Payoff)

## Overview

卡牌数据系统是游戏的数据基础层，定义所有卡牌类型（武将卡、兵种卡、计策卡）的属性模型、标签体系、和数值结构。它本身不产生玩法，而是为堆叠系统、技能联动、自动战斗等7个下游系统提供统一的数据契约。任何卡牌的属性查询、标签匹配、数值计算都通过本系统的接口完成。设计目标是：数据定义完整且无歧义，使下游系统在实现时无需猜测卡牌行为。

## Player Fantasy

本系统为纯基础设施层，玩家不直接感知其存在。玩家体验到的"我的武将组合天衣无缝"或"这张计策卡翻盘了"的快感，由下游的堆叠系统和自动战斗系统交付。卡牌数据系统的成功标志是：玩家永远不会因为数据定义的模糊或不一致而产生困惑。

## Detailed Design

### Core Rules

#### 1. Card Type Hierarchy

所有卡牌共享以下基础属性：

| 属性 | 类型 | 说明 |
|------|------|------|
| `id` | string | 唯一标识符，格式：`[type]_[faction]_[name]` |
| `name` | string | 显示名称（中文） |
| `type` | enum | `general` / `troop` / `tactic` |
| `rarity` | enum | `common` / `rare` / `legendary` |
| `tags` | array[string] | 标签集合（用于技能联动系统匹配） |
| `flavor_text` | string | 战报中的叙事文本 |

---

#### 2. 武将卡 (General Card)

武将是堆叠的核心——一个 Stack 必须有且只有一个武将作为"栈顶"。

| 属性 | 类型 | 范围 | 说明 |
|------|------|------|------|
| `power` | int | 1-100 | 武力值，影响战斗基础伤害 |
| `command` | int | 2-6 | 统率值，决定该武将最多堆叠多少张兵种卡 |
| `skill` | SkillRef | — | 该武将唯一的核心技能引用 |
| `skill_trigger` | enum | `pre_battle` / `clash` / `aftermath` / `passive` | 技能触发阶段 |
| `hp` | int | 50-300 | 武将体力，归零则该 Stack 溃散 |
| `faction` | enum | `shu` / `wei` / `wu` / `qun` | 势力归属 |

**武将标签池**（每个武将携带1-3个）：
`猛将` / `统帅` / `谋士` / `内政` / `水战` / `守城` / `攻城` / `骑术` / `弓术`

**稀有度与属性关系**：

| 稀有度 | power范围 | command范围 | HP范围 | 技能强度 |
|--------|-----------|-------------|--------|----------|
| Common | 20-45 | 2-3 | 50-120 | 单一效果 |
| Rare | 40-70 | 3-5 | 100-200 | 条件增强 |
| Legendary | 65-100 | 4-6 | 180-300 | 链式/AOE效果 |

---

#### 3. 兵种卡 (Troop Card)

兵种卡堆叠在武将下方，填充 command 槽位。分为通用型和精锐型。

| 属性 | 类型 | 范围 | 说明 |
|------|------|------|------|
| `troop_type` | enum | `infantry` / `cavalry` / `archer` / `navy` / `siege` | 兵种大类 |
| `power` | int | 5-40 | 兵种攻击力，叠加到Stack总战力 |
| `hp` | int | 30-150 | 兵种生命值 |
| `slot_cost` | int | 1（通用）/ 2（精锐） | 占用的command槽位数 |
| `is_elite` | bool | — | 是否为精锐（命名型） |
| `subtype` | string | nullable | 精锐变体名称（如"西凉铁骑"） |

**兵种标签池**：
- 大类标签（自动）：`步兵` / `骑兵` / `弓兵` / `水军` / `器械`
- 特性标签（精锐专属，0-2个）：`重甲` / `突击` / `火攻` / `伏兵` / `运输`

**通用 vs 精锐**：

| 类别 | slot_cost | power范围 | 额外标签 | 获取方式 |
|------|-----------|-----------|----------|----------|
| 通用 | 1 | 5-15 | 无 | 每回合补充 |
| 精锐 | 2 | 20-40 | 1-2个特性标签 | 招募/战利品 |

---

#### 4. 计策卡 (Tactic Card)

计策卡在部署阶段附加到 Stack 上（不占用 command 槽位），在战斗的指定阶段触发。

| 属性 | 类型 | 说明 |
|------|------|------|
| `effect_type` | enum | `damage` / `buff` / `debuff` / `terrain` / `special` |
| `trigger_phase` | enum | `pre_battle` / `clash` / `aftermath` |
| `effect_value` | int | 效果数值（伤害量/增益百分比等） |
| `condition` | string | nullable | 触发条件描述（如 "目标有水军标签"） |
| `duration` | int | 持续回合数（0=即时） |
| `max_per_stack` | int | 每Stack最多携带几张该计策（默认1） |

**计策标签池**：`火攻` / `水攻` / `伏兵` / `鼓舞` / `离间` / `治疗` / `侦查`

---

#### 5. 标签系统 (Tag System)

标签是技能联动系统的匹配基础。规则：

1. 每张卡牌携带 1-4 个标签
2. 标签分类：
   - **角色标签**：猛将/统帅/谋士/内政（武将专属）
   - **兵种标签**：步兵/骑兵/弓兵/水军/器械（兵种卡自动获得）
   - **能力标签**：水战/守城/攻城/骑术/弓术（武将能力）
   - **特性标签**：重甲/突击/火攻/伏兵/运输（精锐兵种专属）
   - **计策标签**：火攻/水攻/伏兵/鼓舞/离间/治疗/侦查
3. Stack 的有效标签 = 栈顶武将标签 ∪ 所有兵种卡标签 ∪ 所有计策卡标签
4. 标签重复不叠加（集合语义，非计数语义）
5. 技能联动系统通过查询 Stack 的有效标签集来判断技能是否触发

---

### States and Transitions

卡牌在游戏中的生命周期状态：

| 状态 | 说明 | 可转入 |
|------|------|--------|
| `in_pool` | 在玩家卡池中，可用于组Stack | → `in_stack` |
| `in_stack` | 已堆叠到某个Stack中 | → `in_pool`, → `deployed` |
| `deployed` | Stack已部署到地图区域 | → `in_stack`(回收), → `exhausted` |
| `exhausted` | 战斗后疲劳/损伤状态 | → `in_pool`(恢复后) |
| `destroyed` | 兵种卡HP归零被消灭 | → (removed from game) |

**武将卡不会被永久销毁**——HP归零后进入 `exhausted` 状态，需要休息回合恢复。
**兵种卡可以被销毁**——HP归零时从游戏中移除（需要补充新兵）。

---

### Interactions with Other Systems

| 下游系统 | 依赖的数据接口 | 调用方式 |
|----------|----------------|----------|
| **卡牌堆叠系统** | `command`值、`slot_cost`值、卡牌状态 | 查询武将command上限，检查槽位是否足够 |
| **技能联动系统** | `skill`、`skill_trigger`、`tags`集合 | 查询Stack有效标签集，判断技能触发条件 |
| **地形修正系统** | `tags`（兵种标签）、`troop_type` | 匹配兵种类型与地形克制表 |
| **自动战斗系统** | `power`、`hp`、全部战斗属性 | 读取Stack总战力、执行伤害计算 |
| **资源经济系统** | `rarity`、`is_elite` | 决定卡牌获取成本和补充消耗 |
| **武将管理系统** | 武将全部属性 | 管理武将收集、状态恢复 |
| **兵种管理系统** | 兵种全部属性 | 管理兵种补充、精锐招募 |

## Formulas

### Stack总战力计算（供自动战斗系统调用）

```
stack_power = general.power + Σ(troop[i].power)
```

### Stack总HP计算

```
stack_hp = general.hp + Σ(troop[i].hp)
```

### Command槽位校验

```
is_valid_stack = Σ(troop[i].slot_cost) ≤ general.command
```

### 稀有度-属性生成范围

```
general_power(rarity):
  common    → [20, 45]
  rare      → [40, 70]
  legendary → [65, 100]

general_command(rarity):
  common    → [2, 3]
  rare      → [3, 5]
  legendary → [4, 6]

general_hp(rarity):
  common    → [50, 120]
  rare      → [100, 200]
  legendary → [180, 300]
```

### 标签集合查询

```
stack_tags(stack) = set(general.tags) ∪ ⋃(troop[i].tags) ∪ ⋃(tactic[j].tags)
```

### 示例计算

```
关羽(legendary): power=85, command=5, hp=250, tags={猛将, 骑术}
+ 骑兵(通用): power=10, hp=50, slot=1, tags={骑兵}
+ 骑兵(通用): power=10, hp=50, slot=1, tags={骑兵}
+ 西凉铁骑(精锐): power=30, hp=100, slot=2, tags={骑兵, 重甲, 突击}

Stack总战力 = 85 + 10 + 10 + 30 = 135
Stack总HP = 250 + 50 + 50 + 100 = 450
槽位使用 = 1 + 1 + 2 = 4 ≤ 5 ✓
有效标签 = {猛将, 骑术, 骑兵, 重甲, 突击}
```

## Edge Cases

[To be designed]

## Dependencies

[To be designed]

## Tuning Knobs

[To be designed]

## Visual/Audio Requirements

[To be designed]

## UI Requirements

[To be designed]

## Acceptance Criteria

[To be designed]

## Open Questions

[To be designed]
