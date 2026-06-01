# Auto-Battle System (自动战斗系统)

> **Status**: Draft Complete
> **Author**: user + agents
> **Last Updated**: 2026-06-01
> **Implements Pillar**: Pillar 2 (Decision Before Action — 战斗自动结算，验证决策)

## Overview

自动战斗系统执行RESOLVE阶段的战斗结算。当两个对立Stack在同一区域相遇时，系统通过多轮伤害交换（每轮双方互扣HP）来决定胜负。技能在指定阶段触发，地形修正影响兵种战力，伤害有±10%随机波动。每一轮的详细数据（谁打了谁、技能触发、伤亡数字）都记录为战报事件，传递给战报系统渲染。玩家不干预战斗过程——所有决策已在DEPLOY阶段完成。

## Player Fantasy

"我点了'开战'，看着战报一轮轮展开——第1轮周瑜的火计烧了敌方30HP！第2轮双方交锋，我的骑兵占平原优势多打10%……第4轮敌方溃败！我的部署是对的！"

战报是"看回放"的快感，而非"操作"的快感。

## Detailed Design

### Core Rules

#### 1. Battle Trigger Conditions (战斗触发条件)

战斗在RESOLVE阶段自动触发，条件：
- 玩家Stack被部署到敌方控制的区域（进攻）
- 敌方Stack被部署到玩家控制的区域（防御）
- 同一区域存在敌对双方Stack

每个区域的战斗独立结算，互不影响。

#### 2. Battle Flow (战斗流程)

```
PREPARE → PRE_BATTLE → COMBAT_ROUNDS → AFTERMATH → RESULT
```

| 阶段 | 说明 |
|------|------|
| **PREPARE** | 注入地形临时标签、应用passive技能、计算地形基础修正 |
| **PRE_BATTLE** | 触发所有pre_battle技能（伏兵、先手打击、Buff/Debuff） |
| **COMBAT_ROUNDS** | 多轮伤害交换，每轮双方互扣HP。第1轮触发clash技能 |
| **AFTERMATH** | 触发aftermath技能（治疗、追击、掠夺） |
| **RESULT** | 判定胜负、计算伤亡、更新卡牌状态 |

#### 3. Combat Round Resolution (每轮结算)

每轮双方同时造成伤害：

```
attacker_damage = attacker_modified_power × random(0.9, 1.1)
defender_damage = defender_modified_power × random(0.9, 1.1)

defender_hp -= attacker_damage
attacker_hp -= defender_damage
```

**modified_power**已包含：
- Stack基础战力（general.power + Σ troop.power）
- 地形基础修正（对各兵种的±%）
- 技能效果（buff/debuff已应用）

#### 4. Clash Skills (第1轮触发)

clash类型技能在COMBAT_ROUNDS的第1轮开始前触发，效果应用到整个战斗阶段：
- damage类型：第1轮额外造成固定伤害
- buff_self类型：提升己方modified_power（持续到战斗结束）
- debuff_enemy类型：降低对方modified_power

#### 5. Round Limit (轮数上限)

```
MAX_COMBAT_ROUNDS = 8
```

如果8轮后双方都未被消灭 → 判定为**平局**（双方撤退，各保留剩余HP）。

#### 6. Victory Determination (胜负判定)

| 结果 | 条件 | 后果 |
|------|------|------|
| **胜利** | 对方Stack总HP ≤ 0 | 胜方保留剩余HP，控制该区域 |
| **失败** | 己方Stack总HP ≤ 0 | 败方武将进入exhausted，兵种按HP判定存活/销毁 |
| **平局** | 8轮后双方都存活 | 攻击方撤退回出发区域，防守方保留控制权 |

#### 7. Casualty Calculation (伤亡计算)

战斗结束后，损失的HP分配到各单位：

```
damage_taken = initial_total_hp - remaining_total_hp

# HP按比例分摊到各单位（武将承担30%，兵种承担70%）
general_damage = damage_taken × 0.30
troop_damage = damage_taken × 0.70

# 兵种伤亡从最低HP的开始消灭
troops_sorted_by_hp = sort(stack.troops, key=hp, ascending)
remaining_troop_damage = troop_damage
for troop in troops_sorted_by_hp:
  if remaining_troop_damage >= troop.current_hp:
    troop.state = "destroyed"  # 兵种被消灭
    remaining_troop_damage -= troop.current_hp
  else:
    troop.current_hp -= remaining_troop_damage
    break
```

**武将不会被永久消灭**——HP归零后进入exhausted状态。

#### 8. Battle Event Log (战斗事件记录)

每个战斗动作生成一条事件，传递给战报系统：

```
BattleEvent {
  round: int           # 轮次（0=pre_battle, 1-8=combat, 9=aftermath）
  type: string          # "skill_trigger" / "damage" / "casualty" / "result"
  source: string        # 造成事件的单位名
  target: string        # 受影响的单位名
  value: int            # 数值（伤害量/治疗量）
  description: string   # 战报文本（如"关羽·突破 造成40点额外伤害"）
}
```

---

### States and Transitions

战斗系统无持久状态——每场战斗是独立的纯函数：

```
输入: attacker_stack, defender_stack, terrain_type
输出: BattleResult {
  winner: "attacker" / "defender" / "draw"
  attacker_remaining_hp: int
  defender_remaining_hp: int
  rounds_fought: int
  events: array[BattleEvent]
  casualties: { destroyed_troops: [...], exhausted_generals: [...] }
}
```

---

### Interactions with Other Systems

| 系统 | 交互方式 |
|------|----------|
| **卡牌堆叠系统** | 读取Stack组成（武将+兵种+计策） |
| **技能联动系统** | 各阶段查询技能触发结果 |
| **地形修正系统** | 查询地形标签注入 + 兵种基础修正 |
| **回合流程系统** | RESOLVE信号触发所有区域战斗执行 |
| **战报系统** | 输出BattleEvent数组供战报渲染 |
| **卡牌数据系统** | 战后更新卡牌状态（HP、exhausted、destroyed） |
| **大地图系统** | 战后更新区域控制权 |

## Formulas

### Stack修正战力

```
modified_power(stack, terrain, role):
  base = stack_power(stack)  # general.power + Σ(troop.power)
  
  # 地形基础修正（按兵种分别计算）
  terrain_bonus = 0
  for troop in stack.troops:
    troop_modifier = terrain_modifier(terrain, troop.troop_type)
    terrain_bonus += troop.power × troop_modifier / 100
  
  # 技能修正（buff/debuff百分比）
  skill_modifier = 1.0 + (buff_percent - debuff_percent) / 100
  
  return (base + terrain_bonus) × skill_modifier
```

### 每轮伤害

```
round_damage(attacker_power, variance=0.1):
  return floor(attacker_power × uniform(1-variance, 1+variance))
```

### 伤亡分摊

```
general_share = 0.30   # 武将承担30%伤害
troop_share = 0.70     # 兵种承担70%伤害
```

### 示例战斗

```
攻击方: 关羽Stack (power=135, hp=450) 进攻 平原
  关羽 power=85, hp=250, skill=突破(clash, 骑兵>=2, damage 40)
  骑兵×2 power=10ea, hp=50ea
  西凉铁骑 power=30, hp=100

防守方: 张角Stack (power=80, hp=300) 驻守 平原
  张角 power=40, hp=150
  步兵×3 power=10ea, hp=50ea

地形: 平原 → 攻击方获得「平原地利」标签
  骑兵+10% → 骑兵power: 10→11, 10→11, 30→33
  攻击方modified_power = 85 + 11 + 11 + 33 = 140

PREPARE: 平原地利标签注入攻击方
PRE_BATTLE: 无pre_battle技能触发
CLASH(轮1): 关羽突破触发(骑兵来源数=3≥2) → 防守方-40HP
  攻击伤害: 140 × ~1.0 ≈ 140   防守方HP: 300-40-140 = 120
  防守伤害: 80 × ~1.0 ≈ 80     攻击方HP: 450-80 = 370
轮2:
  攻击伤害: 140 × ~0.95 ≈ 133  防守方HP: 120-133 < 0 → 溃败!

结果: 攻击方胜利，2轮结束，剩余HP ≈ 290
```

## Edge Cases

| # | 边界情况 | 处理规则 |
|---|----------|----------|
| 1 | 双方同一轮HP都归零 | 攻击方判定为败（防守方享有主场优势） |
| 2 | 同一区域多个Stack（2v1） | MVP不支持多Stack联合作战。每个Stack独立战斗。如果攻击方有2个Stack攻同一区域，按顺序依次战斗（前一个打完再下一个） |
| 3 | 技能伤害导致pre_battle阶段就消灭对方 | 允许。战斗直接跳到RESULT，不进入COMBAT_ROUNDS |
| 4 | ±10%波动使弱方偶尔翻盘 | 设计预期。波动仅影响边缘对局（战力差<20%），大优势方几乎不可能翻车 |
| 5 | 平局后攻击方撤退——撤退到哪里？ | 回到出发区域（部署前所在的区域） |
| 6 | 所有兵种被消灭但武将还有HP | 武将单独进入exhausted。没有兵种=无法组Stack=需要补兵才能再战 |
| 7 | 战斗在无人防守的区域（空区域进攻） | 无战斗，直接占领 |

## Dependencies

**上游依赖**：

| 系统 | 依赖内容 |
|------|----------|
| 卡牌堆叠系统 | Stack组成数据 |
| 技能联动系统 | 各阶段技能触发结果 |
| 地形修正系统 | 地形标签+基础修正 |

**下游依赖方**：

| 系统 | 依赖性质 |
|------|----------|
| 战报系统 | BattleEvent数组 |
| 大地图系统 | 区域控制权变更 |
| 卡牌数据系统 | 卡牌状态更新（HP、destroyed、exhausted） |

## Tuning Knobs

| 参数名 | 当前值 | 安全范围 | 影响 |
|--------|--------|----------|------|
| `DAMAGE_VARIANCE` | 0.10 (±10%) | 0-0.25 | 伤害波动。=0则完全确定；>0.2则运气成分过大 |
| `MAX_COMBAT_ROUNDS` | 8 | 4-12 | 轮数上限。太少→高HP对局总平局；太多→战报冗长 |
| `GENERAL_DAMAGE_SHARE` | 0.30 | 0.2-0.5 | 武将承担伤害比例。太高→武将太脆；太低→兵种纸糊 |
| `DEFENDER_TIEBREAK` | defender wins | — | 同归于尽时谁胜。防守方优势=鼓励先侦查再进攻 |
| `SEQUENTIAL_ATTACK_ORDER` | deployment order | — | 多Stack攻同一区域时的出战顺序 |

## Visual/Audio Requirements

- RESOLVE阶段：地图上显示战斗图标（交叉剑），可点击展开详情
- 战斗动画可选：简单→区域闪烁；中等→Stack图标碰撞；详细→逐轮文字滚动
- 胜负音效：胜利=号角，失败=低沉鼓声，平局=撤退号角

## UI Requirements

- RESOLVE阶段自动播放所有区域战斗（按地图从左到右顺序）
- 每场战斗结束后短暂显示结果标签（胜利/失败/平局）
- 点击已结算的战斗区域 → 查看详细战报（由战报系统渲染）
- 可通过设置调节RESOLVE播放速度（1x/2x/3x）

## Acceptance Criteria

| # | 验收条件 | 验证方式 |
|---|----------|----------|
| 1 | 战斗流程5个阶段按顺序执行 | 单元测试 |
| 2 | 技能在正确阶段触发（pre_battle/clash/aftermath） | 单元测试 |
| 3 | 地形修正正确应用到兵种战力 | 单元测试 |
| 4 | ±10%伤害波动在范围内 | 单元测试（1000次模拟，验证分布） |
| 5 | 8轮上限后正确判定平局 | 单元测试 |
| 6 | 同归于尽时防守方胜 | 单元测试 |
| 7 | 伤亡正确分摊到武将(30%)和兵种(70%) | 单元测试 |
| 8 | 兵种HP归零时正确标记destroyed | 单元测试 |
| 9 | BattleEvent日志完整记录每轮动作 | 单元测试 |
| 10 | 相同输入+固定seed产生相同结果（可重放） | 单元测试 |

## Open Questions

| # | 问题 | 何时决定 |
|---|------|----------|
| 1 | 是否支持多Stack联合攻击同一区域（协同作战）？ | Alpha阶段评估 |
| 2 | 战斗中兵种是否有克制关系（骑兵克弓兵等）？ | 平衡测试后决定是否需要额外层次 |
| 3 | 是否需要"撤退"机制（战斗中途判断不利可跑）？ | 违反Pillar 2（战中无决策），暂不加 |
| 4 | 随机种子如何管理（每回合一个seed？每场战斗一个？） | 实现时确认 |
