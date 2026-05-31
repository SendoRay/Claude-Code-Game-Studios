# Turn Flow System (回合流程系统)

> **Status**: Draft Complete
> **Author**: user + agents
> **Last Updated**: 2026-05-31
> **Implements Pillar**: Pillar 2 (Decision Before Action), Pillar 4 (Readable Pressure)

## Overview

回合流程系统是游戏的节奏骨架，定义每回合的6个阶段（情报→收入→部署→确认→结算→战报）的严格线性序列和转换规则。它是纯状态机——不执行具体逻辑，只在正确的时机通知对应系统执行工作（如：进入INCOME阶段时通知资源经济系统发放收入）。系统保证Pillar 2（所有决策发生在部署阶段）和Pillar 4（情报先于决策展示）的节奏约束。

## Player Fantasy

玩家不直接感知"回合系统"本身，但体验到的节奏感——"先看情报知道威胁在哪，再收钱知道有多少资源，然后做艰难的部署决策，最后一键确认等待结果"——完全由本系统编排。成功的标志是：玩家觉得每回合的节奏自然流畅，从不因"现在该干什么"而困惑。

## Detailed Design

### Core Rules

#### 1. Turn Phases (严格线性序列)

每回合按以下固定顺序执行6个阶段，不可跳过或倒退：

| 阶段 | 名称 | 控制权 | 持续方式 | 触发的系统 |
|------|------|--------|----------|------------|
| 1 | INTEL（情报） | 系统自动 | 展示完毕后自动进入下一阶段 | AI对手系统（显示AI意图预告） |
| 2 | INCOME（收入） | 系统自动 | 资源结算完毕后自动进入下一阶段 | 资源经济系统（发放回合收入） |
| 3 | DEPLOY（部署） | 玩家控制 | 玩家主动点击"结束部署"按钮 | 卡牌堆叠系统、回合部署系统 |
| 4 | COMMIT（确认） | 玩家确认 | 显示部署总览，玩家点击"开战" | 无（纯确认UI） |
| 5 | RESOLVE（结算） | 系统自动 | 所有战斗计算完成后自动进入下一阶段 | 自动战斗系统（执行所有区域战斗） |
| 6 | REPORT（战报） | 玩家控制 | 玩家可浏览战报，点击"下一回合"推进 | 战报系统（展示结果） |

**设计要点**：
- 阶段1-2是自动的"信息展示"，玩家只需观看
- 阶段3是玩家自由操作的核心决策时间（无时间限制）
- 阶段4是防误操作的确认步骤（可取消回到阶段3）
- 阶段5是不可中断的批量计算
- 阶段6是可选深度——玩家可跳过详细战报直接进入下一回合

#### 2. Turn Counter

```
turn_number: int (从1开始)
max_turns: int (由剧本定义，默认无上限)
```

每次从REPORT进入下一回合时 `turn_number += 1`。

#### 3. Game-Level States

回合系统还管理游戏级别的状态：

```
SCENARIO_START → PLAYING → VICTORY / DEFEAT
```

- `SCENARIO_START`: 剧本初始化（设置地图、初始卡牌、AI）
- `PLAYING`: 正常回合循环
- `VICTORY`: 胜利条件达成（由剧本系统判定）
- `DEFEAT`: 失败条件触发（所有区域丢失 / 超过max_turns）

---

### States and Transitions

```
SCENARIO_START
    → [初始化完成] → INTEL

INTEL
    → [情报展示完毕] → INCOME

INCOME
    → [资源结算完毕] → DEPLOY

DEPLOY
    → [玩家点击"结束部署"] → COMMIT
    → [胜利条件达成] → VICTORY (罕见：占领最后区域在部署阶段触发)

COMMIT
    → [玩家确认"开战"] → RESOLVE
    → [玩家取消] → DEPLOY (回退)

RESOLVE
    → [所有战斗结算完毕] → REPORT
    → [胜利条件达成] → VICTORY
    → [失败条件触发] → DEFEAT

REPORT
    → [玩家点击"下一回合"] → INTEL (turn_number += 1)
    → [超过max_turns] → DEFEAT
    → [胜利条件达成] → VICTORY

VICTORY → (游戏结束界面)
DEFEAT → (游戏结束界面)
```

**回退规则**：唯一允许的回退是 COMMIT → DEPLOY。所有其他转换不可逆。

---

### Interactions with Other Systems

| 系统 | 交互方式 | 时机 |
|------|----------|------|
| **AI对手系统** | 回合系统通知"进入INTEL" → AI系统返回本回合意图数据 | INTEL阶段开始时 |
| **资源经济系统** | 回合系统通知"进入INCOME" → 经济系统计算并发放资源 | INCOME阶段开始时 |
| **回合部署系统** | DEPLOY阶段开放部署操作；COMMIT时锁定部署方案 | DEPLOY/COMMIT阶段 |
| **卡牌堆叠系统** | DEPLOY阶段允许修改Stack组成 | DEPLOY阶段 |
| **自动战斗系统** | 回合系统通知"进入RESOLVE" → 战斗系统执行所有区域战斗 | RESOLVE阶段 |
| **战报系统** | RESOLVE完成后，将战斗结果传递给战报系统展示 | REPORT阶段 |
| **剧本系统** | 提供max_turns、胜利/失败条件、每回合事件触发 | 全局 |

## Formulas

### 回合时间预算（设计约束）

```
target_session_minutes = 50-70
estimated_turns = target_session_minutes / avg_turn_minutes

avg_turn_minutes breakdown:
  INTEL:   ~15s (自动)
  INCOME:  ~5s (自动)
  DEPLOY:  ~2-3min (玩家思考+操作)
  COMMIT:  ~5s (确认)
  RESOLVE: ~10-30s (计算+动画)
  REPORT:  ~15-60s (玩家可选阅读)

avg_turn_minutes ≈ 3-4.5min
estimated_turns ≈ 12-20 turns per scenario
```

## Edge Cases

| # | 边界情况 | 处理规则 |
|---|----------|----------|
| 1 | 玩家在DEPLOY阶段不做任何操作直接"结束部署" | 允许。所有Stack保持上回合状态（未部署的Stack视为"原地驻守"） |
| 2 | RESOLVE阶段没有任何战斗发生（无接触区域） | 正常：跳过战斗计算，REPORT阶段显示"本回合无战事" |
| 3 | 第1回合的INTEL阶段（AI还没有历史行为） | 显示AI初始势力范围和"暂无情报"提示 |
| 4 | 胜利/失败条件在RESOLVE中间触发 | 完成当前战斗结算后再触发结束，不中断计算 |
| 5 | 玩家所有武将都在exhausted状态（无法部署） | 允许空部署（跳过DEPLOY直接COMMIT），回合仍推进（恢复需要回合流逝） |
| 6 | max_turns到达时玩家正在DEPLOY阶段 | 不强制中断。当前回合正常完成，在REPORT后判定超时失败 |

## Dependencies

**上游依赖**：无（Foundation层）

**下游依赖方**：

| 系统 | 依赖性质 |
|------|----------|
| 资源经济系统 | 需要INCOME阶段信号来发放收入 |
| 回合部署系统 | 需要DEPLOY阶段开启/锁定信号 |
| AI对手系统 | 需要INTEL阶段信号来展示AI意图 + 需要turn_number推进AI决策 |

## Tuning Knobs

| 参数名 | 当前值 | 安全范围 | 影响 |
|--------|--------|----------|------|
| `INTEL_DISPLAY_DURATION` | 2s | 1-5s | 情报展示时间。太短→玩家来不及看；太长→节奏拖沓 |
| `INCOME_DISPLAY_DURATION` | 1s | 0.5-3s | 收入动画时间 |
| `RESOLVE_ANIMATION_SPEED` | 1x | 0.5x-3x | 战斗结算动画速度（玩家可调） |
| `AUTO_ADVANCE_REPORT` | false | bool | 是否自动跳过战报进入下一回合（玩家设置项） |
| `MAX_TURNS_DEFAULT` | 30 | 15-50 | 默认最大回合数。太少→玩家时间压力大；太多→拖沓 |
| `FIRST_TURN_SKIP_INTEL` | true | bool | 第1回合是否跳过无意义的情报阶段 |

## Visual/Audio Requirements

- 阶段切换需要明确的视觉过渡（如标题横幅滑入："情报阶段"/"部署阶段"）
- DEPLOY→COMMIT需要"确认感"的音效反馈
- RESOLVE阶段播放战斗结算的紧张BGM

## UI Requirements

- 屏幕上方常驻显示：当前阶段名 + 回合数（如"第5回合 · 部署阶段"）
- DEPLOY阶段底部显示"结束部署"按钮（需二次确认）
- COMMIT阶段显示部署总览面板 + "开战"/"取消"两个按钮
- REPORT阶段显示"下一回合"按钮（始终可见）

## Acceptance Criteria

| # | 验收条件 | 验证方式 |
|---|----------|----------|
| 1 | 6个阶段按严格线性顺序执行，无跳过 | 单元测试：模拟完整回合，断言阶段顺序 |
| 2 | COMMIT→DEPLOY回退正常工作 | 单元测试：在COMMIT取消，验证回到DEPLOY |
| 3 | 除COMMIT外所有阶段转换不可逆 | 单元测试：在RESOLVE/REPORT尝试回退 → 失败 |
| 4 | turn_number在每回合结束时+1 | 单元测试 |
| 5 | 胜利/失败条件正确触发游戏结束 | 集成测试 |
| 6 | 空部署（无任何操作）不阻塞回合推进 | 单元测试 |
| 7 | 各阶段正确通知对应系统 | 集成测试：监听信号，验证每阶段触发正确信号 |

## Open Questions

| # | 问题 | 何时决定 |
|---|------|----------|
| 1 | 是否需要"快速模式"跳过INTEL+INCOME动画？ | UI框架设计时确认 |
| 2 | 多人热座模式时回合如何交替？ | 如果加入PvP再定 |
| 3 | 是否允许在REPORT阶段进行某些管理操作（查看卡牌详情）？ | UI设计时确认 |
