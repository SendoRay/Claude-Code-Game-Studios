# Systems Index: 堆叠三国 (Stacking Kingdoms)

> **Status**: Draft
> **Created**: 2026-05-31
> **Last Updated**: 2026-05-31
> **Source Concept**: design/gdd/game-concept.md

---

## Overview

堆叠三国是一款卡组构筑+大地图征服策略游戏，需要20个系统支撑完整体验。核心循环
是"堆叠卡牌→部署到区域→自动结算→读战报→下一回合"。四大设计支柱（堆叠即简化、
战前决策>战中操作、构筑弧线、可预见压力）约束了系统设计方向。

地图采用区域制+视觉地图方案：逻辑层是命名区域的连接图，表现层是风格化三国地图。

---

## Systems Enumeration

| # | System Name | Category | Priority | Status | Design Doc | Depends On |
|---|-------------|----------|----------|--------|------------|------------|
| 1 | 卡牌数据系统 | Core | MVP | Not Started | — | (none) |
| 2 | 回合流程系统 | Core | MVP | Not Started | — | (none) |
| 3 | 技能联动系统 | Gameplay | MVP | Not Started | — | 卡牌数据 |
| 4 | 卡牌堆叠系统 | Gameplay | MVP | Not Started | — | 卡牌数据 |
| 5 | 地形修正系统 | Gameplay | MVP | Not Started | — | 卡牌数据 |
| 6 | 资源经济系统 | Economy | MVP | Not Started | — | 回合流程 |
| 7 | 自动战斗系统 | Gameplay | MVP | Not Started | — | 卡牌堆叠, 技能联动, 地形修正 |
| 8 | 大地图系统 | Core | MVP | Not Started | — | 地形修正, 资源经济 |
| 9 | 回合部署系统 | Gameplay | MVP | Not Started | — | 卡牌堆叠, 大地图, 回合流程 |
| 10 | AI对手系统 | Gameplay | MVP | Not Started | — | 大地图, 回合流程 |
| 11 | 战报系统 | UI | MVP | Not Started | — | 自动战斗 |
| 12 | 武将管理系统 | Gameplay | Vertical Slice | Not Started | — | 卡牌数据, 资源经济 |
| 13 | 兵种管理系统 | Gameplay | Vertical Slice | Not Started | — | 卡牌数据, 资源经济 |
| 14 | 计策系统 | Gameplay | Vertical Slice | Not Started | — | 卡牌数据, 技能联动 |
| 15 | 武将招募系统 | Gameplay | Vertical Slice | Not Started | — | 武将管理, 资源经济 |
| 16 | UI框架系统 | UI | Vertical Slice | Not Started | — | (all gameplay systems) |
| 17 | 剧本系统 | Narrative | Alpha | Not Started | — | 大地图, AI对手 |
| 18 | 存档系统 | Persistence | Alpha | Not Started | — | (all state systems) |
| 19 | 教学系统 | Meta | Alpha | Not Started | — | 剧本, UI框架 |
| 20 | 长线进度系统 | Progression | Full Vision | Not Started | — | 武将管理, 剧本 |

---

## Categories

| Category | Description |
|----------|-------------|
| **Core** | 基础系统，所有其他系统依赖的数据和流程骨架 |
| **Gameplay** | 产生游戏乐趣的核心机制系统 |
| **Economy** | 资源产出和消耗，约束玩家行为的经济系统 |
| **UI** | 向玩家展示信息和接收输入的界面系统 |
| **Narrative** | 剧本和故事内容的组织系统 |
| **Persistence** | 游戏状态的保存和加载 |
| **Progression** | 跨局/长线的成长系统 |
| **Meta** | 游戏核心循环之外的辅助系统 |

---

## Priority Tiers

| Tier | Definition | Target | Systems Count |
|------|------------|--------|---------------|
| **MVP** | 核心循环运转的最低需求：组牌→部署→战斗→战报 | T1 (6-10周) | 11 |
| **Vertical Slice** | 完整单局体验：武将管理、补员、招募、计策 | T2 (4-6月) | 5 |
| **Alpha** | 所有功能粗略实现：教学剧本、存档 | T2+ | 3 |
| **Full Vision** | 跨局系统、完整内容 | T3 (1年+) | 1 |

---

## Dependency Map

### Foundation Layer (no dependencies)

1. **卡牌数据系统** — 定义所有卡牌属性、标签、技能的数据基础
2. **回合流程系统** — 游戏节奏的状态机骨架

### Core Layer (depends on Foundation)

3. **技能联动系统** — depends on: 卡牌数据
4. **卡牌堆叠系统** — depends on: 卡牌数据
5. **地形修正系统** — depends on: 卡牌数据
6. **资源经济系统** — depends on: 回合流程

### Feature Layer (depends on Core)

7. **自动战斗系统** — depends on: 卡牌堆叠, 技能联动, 地形修正
8. **大地图系统** — depends on: 地形修正, 资源经济
9. **回合部署系统** — depends on: 卡牌堆叠, 大地图, 回合流程
10. **AI对手系统** — depends on: 大地图, 回合流程
11. **战报系统** — depends on: 自动战斗
12. **武将管理系统** — depends on: 卡牌数据, 资源经济
13. **兵种管理系统** — depends on: 卡牌数据, 资源经济
14. **计策系统** — depends on: 卡牌数据, 技能联动
15. **武将招募系统** — depends on: 武将管理, 资源经济
16. **剧本系统** — depends on: 大地图, AI对手

### Presentation Layer (depends on Features)

17. **UI框架系统** — depends on: all gameplay systems

### Polish Layer (depends on everything)

18. **存档系统** — depends on: all state systems
19. **教学系统** — depends on: 剧本, UI框架
20. **长线进度系统** — depends on: 武将管理, 剧本

---

## Recommended Design Order

| Order | System | Priority | Layer | Est. Effort |
|-------|--------|----------|-------|-------------|
| 1 | 卡牌数据系统 | MVP | Foundation | M |
| 2 | 回合流程系统 | MVP | Foundation | S |
| 3 | 技能联动系统 | MVP | Core | M |
| 4 | 卡牌堆叠系统 | MVP | Core | M |
| 5 | 地形修正系统 | MVP | Core | S |
| 6 | 资源经济系统 | MVP | Core | M |
| 7 | 自动战斗系统 | MVP | Feature | L |
| 8 | 大地图系统 | MVP | Feature | M |
| 9 | 回合部署系统 | MVP | Feature | M |
| 10 | AI对手系统 | MVP | Feature | M |
| 11 | 战报系统 | MVP | Feature | S |
| 12 | 武将管理系统 | VS | Feature | M |
| 13 | 兵种管理系统 | VS | Feature | S |
| 14 | 计策系统 | VS | Feature | S |
| 15 | 武将招募系统 | VS | Feature | M |
| 16 | UI框架系统 | VS | Presentation | L |
| 17 | 剧本系统 | Alpha | Feature | M |
| 18 | 存档系统 | Alpha | Polish | M |
| 19 | 教学系统 | Alpha | Polish | M |
| 20 | 长线进度系统 | Full Vision | Polish | S |

Effort: S = 1 session, M = 2-3 sessions, L = 4+ sessions

---

## Circular Dependencies

None found.

---

## High-Risk Systems

| System | Risk Type | Risk Description | Mitigation |
|--------|-----------|-----------------|------------|
| 自动战斗系统 | Design | 四阶段结算 + 兵种克制 + 技能联动的交互矩阵庞大，平衡难度高 | 先用原型验证简单版本（已完成），逐步加入复杂度 |
| AI对手系统 | Technical | 即使简化AI，意图预告+多势力扩张逻辑对首款游戏仍有挑战 | MVP只用1个AI + 固定战力值模拟，不做完整卡组 |
| UI框架系统 | Scope | 三层缩放地图 + 卡牌拖拽 + 移动端适配是大量UI工程 | 推迟到VS阶段，MVP用最简UI |

---

## Progress Tracker

| Metric | Count |
|--------|-------|
| Total systems identified | 20 |
| Design docs started | 0 |
| Design docs reviewed | 0 |
| Design docs approved | 0 |
| MVP systems designed | 0/11 |
| Vertical Slice systems designed | 0/5 |

---

## Key Design Decision

**地图方案确认**：区域制 + 视觉地图。底层逻辑为命名区域连接图（无坐标、无网格），
表现层为风格化三国地图（区域用省份形状边界表示）。

**战报方案确认**（原型验证）：战报为可选深度内容。默认显示结果标签（胜/败+关键原因），
玩家可展开查看完整回放。非强制弹窗。

---

## Next Steps

- [ ] Design MVP-tier systems in order (use `/design-system [system-name]`)
- [ ] Run `/design-review` on each completed GDD
- [ ] Run `/gate-check systems-design` when all 11 MVP GDDs are complete
- [ ] Validate with `/vertical-slice` before committing to Production
