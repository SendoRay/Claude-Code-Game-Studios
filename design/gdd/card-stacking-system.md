# Card Stacking System (卡牌堆叠系统)

> **Status**: Draft Complete
> **Author**: user + agents
> **Last Updated**: 2026-05-31
> **Implements Pillar**: Pillar 1 (Stack = Simplicity)

## Overview

卡牌堆叠系统管理玩家将兵种卡和计策卡拖拽到武将卡上形成作战Stack的交互规则。这是游戏的标志性操作——"拖卡组队"的物理手感和即时视觉反馈构成了核心乐趣循环的起点。系统在DEPLOY阶段开放，玩家可自由组建、拆解、重组Stack，直到点击"结束部署"为止。堆叠受command槽位限制，是Pillar 1（Stack=Simplicity）的机制保证。

## Player Fantasy

"把3张骑兵卡一张张叠到关羽卡下面，看着队伍越来越强、技能指示器从灰变亮的那一刻——组队的快感就是这个游戏最直觉的乐趣。"

堆叠操作应该像打扑克时理牌一样自然——拿起来、放上去、听到"嗒"一声归位。

## Detailed Design

### Core Rules

#### 1. Stack Structure (堆叠结构)

一个合法Stack的组成：

```
[武将卡] ← 栈顶（必须有且仅有1张）
  ├── [兵种卡] × N（受command限制）
  └── [计策卡] × M（受TACTIC_PER_STACK_LIMIT限制，不占command槽位）
```

**合法性规则**：
- 每个Stack必须有且仅有1张武将卡作为栈顶
- 兵种卡总slot_cost ≤ 武将的command值
- 计策卡数量 ≤ TACTIC_PER_STACK_LIMIT（当前=1）
- 最小合法Stack = 1武将 + 1兵种（部署前必须满足）

#### 2. Stacking Operations (堆叠操作)

| 操作 | 输入 | 规则 | 反馈 |
|------|------|------|------|
| **添加兵种** | 拖拽兵种卡→武将卡 | slot_cost检查；状态检查(in_pool) | 成功：卡牌滑入Stack+音效；失败：弹回+抖动+提示原因 |
| **添加计策** | 拖拽计策卡→Stack | max_per_stack检查 | 同上 |
| **移除卡牌** | 拖拽Stack内卡牌→卡池区域 | 始终允许（DEPLOY阶段内） | 卡牌滑回卡池 |
| **更换武将** | 拖拽新武将→已有Stack | 新武将command ≥ 当前兵种总slot_cost | 旧武将回池，新武将接管Stack |
| **整Stack拆解** | 长按Stack→"解散"按钮 | 始终允许 | 所有卡牌动画飞回卡池 |

#### 3. Timing Rules (时机规则)

- 堆叠操作**仅在DEPLOY阶段**开放
- DEPLOY阶段内，玩家可无限次组建/拆解/重组
- 进入COMMIT阶段后，Stack组成锁定（但可取消回DEPLOY继续调整）
- RESOLVE/REPORT阶段Stack不可修改

#### 4. Visual Stack Representation (视觉表示)

Stack在屏幕上的呈现：
- 武将卡全尺寸显示在顶部
- 兵种卡以半露出方式堆叠在武将下方（扇形展开或瀑布式）
- 计策卡以小图标形式附着在Stack侧边
- 空的command槽位显示为虚线框（暗示"还能放"）
- 已满的Stack显示"满编"标记

#### 5. Drag-and-Drop Rules (拖放规则)

- **有效目标高亮**：拖起一张卡时，所有可接受该卡的目标（武将/Stack）发光
- **无效目标变暗**：不能接受的目标暗淡+禁止图标
- **释放位置容错**：释放在武将卡附近（而非正上方）也算有效拖放
- **Touch优化**：长按0.2s触发拖拽；拖拽时卡牌放大1.2x防手指遮挡；所有触摸目标≥44x44px

---

### States and Transitions

Stack在堆叠系统中的状态：

| 状态 | 说明 |
|------|------|
| `empty` | 只有武将，无兵种/计策 |
| `building` | 正在DEPLOY阶段组建中 |
| `ready` | 满足最小组成要求（≥1兵种），可部署 |
| `full` | 所有command槽位已满 |
| `deployed` | 已分配到地图区域（由回合部署系统管理） |
| `locked` | COMMIT后不可修改 |

```
empty → [添加兵种] → building/ready
ready → [继续添加] → full
ready/full → [移除卡牌] → empty/building
ready/full → [部署到区域] → deployed
deployed → [取消部署/COMMIT取消] → ready/full
ready/full/deployed → [进入COMMIT确认] → locked
locked → [COMMIT取消] → ready/full
```

---

### Interactions with Other Systems

| 系统 | 交互方式 |
|------|----------|
| **卡牌数据系统** | 查询command值、slot_cost、card状态；修改card状态(in_pool↔in_stack) |
| **技能联动系统** | 堆叠变化时实时查询技能触发预判 → 更新UI指示器 |
| **回合部署系统** | 提供ready/full状态的Stack列表供部署选择 |
| **回合流程系统** | 接收DEPLOY阶段开始/结束信号，控制操作开放/锁定 |

## Formulas

### 槽位计算

```
remaining_slots(stack) = general.command - Σ(troop[i].slot_cost)
can_add_troop(stack, troop) = troop.slot_cost ≤ remaining_slots(stack)
                              AND troop.state == "in_pool"
```

### 拖放容错范围

```
valid_drop_radius = card_width × 1.5  (PC)
valid_drop_radius = card_width × 2.0  (Mobile, 更大容错)
```

### Stack战力预览

```
preview_power(stack) = general.power + Σ(troop[i].power)
preview_hp(stack) = general.hp + Σ(troop[i].hp)
skill_ready = skill_synergy_system.can_trigger(general.skill, stack)
```

## Edge Cases

| # | 边界情况 | 处理规则 |
|---|----------|----------|
| 1 | 玩家拖拽卡牌到屏幕外/无效区域 | 卡牌动画弹回原位 |
| 2 | 武将卡被替换时，新武将command < 当前兵种总slot_cost | 阻止替换，提示"统率不足，需先移除X张兵种" |
| 3 | 同时用两根手指拖拽两张卡（多点触控） | 只响应第一个触摸点，忽略后续 |
| 4 | 拖拽过程中DEPLOY阶段被其他事件中断 | 不应发生（DEPLOY阶段由玩家主动结束），但如果发生则取消拖拽、卡牌回原位 |
| 5 | 卡池中无可用兵种卡（全部in_stack或destroyed） | UI提示"无可用兵种"，不阻塞流程（玩家可用现有Stack部署） |
| 6 | 玩家创建了Stack但不部署就结束DEPLOY | 允许。未部署的Stack视为"驻守当前位置"（如果之前有位置）或"待命"（留在后方） |

## Dependencies

**上游依赖**：

| 系统 | 依赖内容 |
|------|----------|
| 卡牌数据系统 | command值、slot_cost、卡牌状态管理 |

**下游依赖方**：

| 系统 | 依赖性质 |
|------|----------|
| 自动战斗系统 | 需要完整的Stack数据（武将+兵种+计策）作为战斗输入 |
| 回合部署系统 | 需要ready/full状态的Stack列表供地图部署 |

## Tuning Knobs

| 参数名 | 当前值 | 安全范围 | 影响 |
|--------|--------|----------|------|
| `DRAG_HOLD_DELAY_MS` | 200ms | 100-400ms | 长按触发拖拽延迟。太短→误触多；太长→操作卡顿 |
| `CARD_SNAP_SPEED_MS` | 150ms | 80-300ms | 卡牌归位动画时间。影响操作"手感" |
| `DROP_TOLERANCE_MULTIPLIER` | 1.5x(PC)/2.0x(Mobile) | 1.0-3.0x | 拖放容错范围。太小→难以放准；太大→误放到相邻Stack |
| `TACTIC_PER_STACK_LIMIT` | 1 | 1-3 | 每Stack可附加的计策数（继承自卡牌数据系统） |
| `STACK_ANIMATION_STYLE` | "waterfall" | "waterfall"/"fan" | 兵种卡在Stack中的展示方式 |

## Visual/Audio Requirements

- **拖拽中**：卡牌半透明+轻微阴影提升，跟随手指/鼠标
- **有效目标**：发光边框脉冲（绿色/金色）
- **放置成功**：卡牌滑入位置 + "嗒"音效 + 微震动反馈(Mobile)
- **放置失败**：卡牌弹回 + 短促"嗡"音 + 抖动动画
- **Stack满编**：金色边框 + "满编"标记闪现
- **技能就绪变化**：技能图标从灰色渐变为彩色 + 微光效果

## UI Requirements

- **卡池区域**：屏幕底部/左侧，展示所有in_pool状态的卡牌
- **Stack构建区域**：屏幕中央/右侧，展示当前所有Stack
- **信息面板**：点击任何卡牌显示详情（属性、标签、技能）
- **槽位指示器**：每个Stack下方显示 "X/Y" 已用/总槽位
- **技能就绪指示器**：武将卡上方显示技能是否满足触发条件
- **快速操作**：双击卡牌自动添加到最合适的Stack（有空位+同类标签优先）

## Acceptance Criteria

| # | 验收条件 | 验证方式 |
|---|----------|----------|
| 1 | 拖拽兵种到武将形成Stack，slot_cost正确扣除 | 单元测试 |
| 2 | 超过command限制时拒绝添加并给出正确提示 | 单元测试 + UI测试 |
| 3 | 从Stack移除卡牌正确归还slot并更新卡牌状态 | 单元测试 |
| 4 | 替换武将时正确校验新武将command是否足够 | 单元测试 |
| 5 | 拖放容错范围在PC和Mobile上分别正确 | 手动测试 |
| 6 | Stack技能预判在每次添加/移除卡牌时实时更新 | 集成测试 |
| 7 | DEPLOY阶段外不可执行堆叠操作 | 单元测试 |
| 8 | 触摸操作满足44x44px最小触摸目标 | UI审查 |
| 9 | 多点触控情况下只响应第一个触摸点 | 手动测试(Mobile) |
| 10 | 动画时间≤300ms，不阻塞下一次操作 | 性能测试 |

## Open Questions

| # | 问题 | 何时决定 |
|---|------|----------|
| 1 | Stack在地图上如何视觉表示（缩略Stack图标 vs 武将头像）？ | 设计大地图系统时确认 |
| 2 | 是否需要"Stack模板"功能（保存/加载常用组合）？ | Alpha阶段根据玩家反馈决定 |
| 3 | PC拖拽是否支持框选多张卡批量添加？ | UI迭代时决定 |
