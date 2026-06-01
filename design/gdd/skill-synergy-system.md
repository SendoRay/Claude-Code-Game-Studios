# Skill Synergy System (技能联动系统)

> **Status**: Draft Complete
> **Author**: user + agents
> **Last Updated**: 2026-05-31
> **Implements Pillar**: Pillar 3 (Build-Up to Payoff)

## Overview

技能联动系统负责在战斗结算时判定哪些武将技能被触发、产生什么效果。它读取卡牌数据系统提供的Stack有效标签集，按照每个技能的触发条件（标签计数阈值）判断是否激活，然后将效果传递给自动战斗系统执行。技能不会链式触发——每个技能独立基于初始状态判定，确保结果可预测且易于平衡。

## Player Fantasy

"我精心凑齐了3张骑兵卡放在关羽麾下——战报中看到'关羽·突破'触发的那一刻，就是我组卡策略的回报。"

技能触发是玩家"组合有效"的奖励信号。每次看到技能名在战报中亮起，都在强化"我的构筑是对的"的正反馈。

## Detailed Design

### Core Rules

#### 1. Skill Definition (技能定义)

每个武将拥有唯一的核心技能，数据结构如下：

| 属性 | 类型 | 说明 |
|------|------|------|
| `skill_id` | string | 唯一标识符 |
| `skill_name` | string | 显示名称（中文） |
| `trigger_phase` | enum | `pre_battle` / `clash` / `aftermath` / `passive` |
| `condition_type` | enum | `always` / `tag_present` / `tag_count` |
| `condition_tag` | string | nullable | 条件引用的标签名 |
| `condition_threshold` | int | 标签计数阈值（仅tag_count时使用） |
| `effect_type` | enum | `damage` / `buff_self` / `debuff_enemy` / `heal` / `special` |
| `effect_target` | enum | `self_stack` / `enemy_stack` / `all_enemies` / `all_allies` |
| `effect_value` | int | 效果数值（伤害量/百分比增益等） |
| `effect_description` | string | 战报中的效果描述文本 |

#### 2. Condition Types (条件类型)

| 条件类型 | 语法 | 示例 |
|----------|------|------|
| `always` | 无条件触发 | 张飞·咆哮：总是触发 |
| `tag_present` | Stack有效标签包含指定标签 | 周瑜·火计：Stack含"火攻"标签时触发 |
| `tag_count` | Stack有效标签中指定标签的来源卡数≥阈值 | 关羽·突破：Stack含≥2张带"骑兵"标签的卡时触发 |

**注意 tag_count 的计数语义**：虽然标签集合本身不重复，但 tag_count 计数的是"有多少张卡贡献了该标签"，而非标签集合大小。例：3张骑兵卡 → 骑兵标签来源数=3。

#### 3. Trigger Phase Resolution (触发阶段)

战斗分3个阶段（由自动战斗系统管理），技能按阶段触发：

| 阶段 | 时机 | 典型技能效果 |
|------|------|------------|
| `pre_battle` | 战斗开始前 | 先手打击、Buff/Debuff施加、伏兵效果 |
| `clash` | 主战斗阶段 | 增伤、追击、AOE伤害 |
| `aftermath` | 战斗结束后 | 治疗、掠夺资源、追击溃军 |
| `passive` | 全程生效 | 属性加成（+power, +hp等） |

**执行顺序**：同阶段内多个技能同时判定，互不影响。结果按固定顺序应用：
1. Debuff → 2. Buff → 3. Damage → 4. Heal → 5. Special

#### 4. No Chaining Rule (无链式触发)

技能判定基于战斗开始时的Stack初始状态。一个技能的效果不会改变另一个技能的触发条件。这保证：
- 玩家可以在部署阶段100%预测哪些技能会触发
- 无无限循环风险
- 战报可以按阶段清晰展示

#### 5. Passive Skills (被动技能)

`passive` 类型技能在Stack组建时即生效（不需要进入战斗）：
- 效果持续整个战斗过程
- 典型效果："+15% stack_power" / "+30 stack_hp"
- 条件仍需满足（被动技能不是无条件的，除非condition_type=always）

---

### States and Transitions

技能本身无状态——它是纯函数式判定：

```
输入: Stack初始状态 (tags, card counts)
输出: [触发的技能列表 + 各自效果]
```

每回合战斗时重新判定，不存储上一回合的触发历史。

---

### Interactions with Other Systems

| 系统 | 交互方式 |
|------|----------|
| **卡牌数据系统** | 读取：武将的skill定义、Stack有效标签集、各卡牌标签 |
| **自动战斗系统** | 输出：每阶段触发的技能列表+效果 → 战斗系统执行效果 |
| **战报系统** | 输出：技能触发事件（技能名、触发条件、效果描述）→ 战报渲染 |
| **卡牌堆叠系统** | 间接：堆叠组合决定了哪些技能能被触发（玩家的核心决策点） |

## Formulas

### 技能触发判定

```
can_trigger(skill, stack) → bool:
  match skill.condition_type:
    "always" → true
    "tag_present" → skill.condition_tag in stack_tags(stack)
    "tag_count" → count_tag_sources(stack, skill.condition_tag) >= skill.condition_threshold

count_tag_sources(stack, tag) → int:
  count = 0
  for card in stack.all_cards:
    if tag in card.tags:
      count += 1
  return count
```

### 技能效果数值

```
## Buff/Debuff (百分比修正)
modified_power = base_power × (1 + buff_percent/100 - debuff_percent/100)

## Direct Damage
damage_dealt = skill.effect_value (固定值，不受其他修正影响)

## Heal
hp_restored = min(skill.effect_value, max_hp - current_hp)
```

### 示例

```
关羽(legendary): skill="突破", trigger=clash, condition=tag_count("骑兵", 2)
  effect: damage, target=enemy_stack, value=40

Stack: 关羽 + 骑兵 + 骑兵 + 西凉铁骑
count_tag_sources(stack, "骑兵") = 3 >= 2 → 触发!
→ clash阶段对敌方Stack造成40点额外伤害
```

## Edge Cases

| # | 边界情况 | 处理规则 |
|---|----------|----------|
| 1 | 精锐兵种贡献2个标签（如"骑兵"+"重甲"），tag_count如何计算 | 该卡对"骑兵"贡献1次计数，对"重甲"贡献1次计数。一张卡对同一标签只计1次 |
| 2 | 武将自身标签是否参与tag_count | 是。武将标签"骑术"≠"骑兵"（不同标签），但如果条件引用的是武将携带的标签则计入 |
| 3 | 计策卡标签是否参与技能触发判定 | 是。计策卡的标签加入Stack有效标签集，参与所有条件判定 |
| 4 | 两个Stack在同一区域对同一敌人战斗 | 每个Stack独立判定自己的技能。不存在跨Stack触发 |
| 5 | passive技能的条件在战斗中途因兵种死亡而不再满足 | Passive在战斗开始时判定一次，中途不重新判定。开始时满足→全程生效 |
| 6 | 技能效果值为0 | 合法：某些special类型技能效果不是数值型的（如"侦查：揭示敌方Stack内容"） |
| 7 | 敌方AI的武将技能如何触发 | 完全相同的规则。AI Stack和玩家Stack使用同一套判定逻辑 |

## Dependencies

**上游依赖**：

| 系统 | 依赖内容 |
|------|----------|
| 卡牌数据系统 | skill定义、tags集合、tag来源卡计数 |

**下游依赖方**：

| 系统 | 依赖性质 |
|------|----------|
| 自动战斗系统 | 需要每阶段的技能触发结果来执行战斗 |

## Tuning Knobs

| 参数名 | 当前值 | 安全范围 | 影响 |
|--------|--------|----------|------|
| `TAG_COUNT_MAX_THRESHOLD` | 3 | 2-5 | 最高要求的标签计数。过高→几乎无法触发（挫败感）；过低→太容易触发（无构筑挑战） |
| `PASSIVE_BUFF_CAP` | +30% | 10-50% | 被动加成上限。过高→被动技能碾压主动技能 |
| `SKILL_DAMAGE_RATIO` | 技能伤害占总伤害的20-35% | 10-50% | 技能伤害权重。过高→纯靠技能赢；过低→技能无存在感 |
| `DEBUFF_DURATION_MAX` | 1(本场战斗) | 1 | 当前Debuff仅持续当场战斗。未来可能扩展到跨回合 |

## Visual/Audio Requirements

- 技能触发时需要明确的视觉反馈（技能名+图标闪现在Stack上方）
- 不同效果类型用不同颜色：damage=红, buff=金, debuff=紫, heal=绿
- 战报中技能触发行需要高亮显示（区别于普通攻击行）

## UI Requirements

- 部署阶段：当Stack满足技能触发条件时，武将卡上显示"技能就绪"指示器
- 部署阶段：当Stack不满足条件时，显示灰色指示器 + 悬停提示缺少什么（如"还需1张骑兵"）
- 卡牌详情面板：显示技能完整信息（名称、条件、效果）

## Acceptance Criteria

| # | 验收条件 | 验证方式 |
|---|----------|----------|
| 1 | always类型技能100%触发 | 单元测试 |
| 2 | tag_present正确判定标签存在/不存在 | 单元测试：有/无目标标签的Stack各一 |
| 3 | tag_count正确计数标签来源卡数量 | 单元测试：边界值(刚好=阈值, 差1, 超过) |
| 4 | 技能不会链式触发（后触发的技能不受先触发技能效果影响） | 单元测试：设计一个理论上会链式触发的场景 → 验证不触发 |
| 5 | 同阶段多技能按正确顺序应用（Debuff→Buff→Damage→Heal→Special） | 单元测试 |
| 6 | Passive技能在战斗开始时判定，中途不变 | 单元测试 |
| 7 | AI和玩家使用完全相同的判定逻辑 | 代码审查：确认同一函数 |
| 8 | UI正确显示"技能就绪/未就绪"指示器 | 手动测试 |

## Open Questions

| # | 问题 | 何时决定 |
|---|------|----------|
| 1 | 是否需要"反制"机制（一个技能可以取消另一个）？ | Alpha阶段，观察平衡后决定 |
| 2 | 未来是否开放"条件组合"（tag_count AND tag_present）？ | 如果MVP的3种条件类型不够表达力再扩展 |
| 3 | 技能是否需要冷却（不能每回合都触发）？ | 设计自动战斗系统时确认 |
