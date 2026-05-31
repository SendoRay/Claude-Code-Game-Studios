# Game Concept: Stacking Kingdoms (堆叠三国)

*Created: 2026-05-31*
*Status: Draft*

---

## Elevator Pitch

> It's a card-stacking strategy game where you build combat formations by
> stacking generals, troops, and tactics cards into squads, then deploy them
> across a Three Kingdoms grand map to conquer territory through auto-resolved
> battles — combining the intuitive stacking of Stacklands with the strategic
> depth of Teamfight Tactics on a conquest map.

---

## Core Identity

| Aspect | Detail |
| ---- | ---- |
| **Genre** | Strategy / Card-stacking / Grand Map Conquest |
| **Platform** | PC + Mobile (cross-platform) |
| **Target Audience** | Theorycrafter strategists (TFT Diamond+, 率土 seasonal rankers) |
| **Player Count** | Single-player (PvE against AI warlords) |
| **Session Length** | 50-70 minutes per scenario |
| **Monetization** | Premium (buy once, play forever) |
| **Estimated Scope** | Medium-Large (4-6 months, solo) |
| **Comparable Titles** | Stacklands, Teamfight Tactics, 率土之滨 |

---

## Core Fantasy

"I conquered the entire Three Kingdoms map through brilliant card combinations
and strategic deployment — I planned the perfect formations, endured early
pressure, and when the time was right, my stacks crushed everything in their
path."

The payoff isn't just winning — it's watching the battle report narrate HOW
your stack won. Seeing "Guan Yu [Breakthrough] triggers → Cavalry pursuit →
Decisive Victory" is the moment your planning becomes a war story.

---

## Unique Hook

Like Stacklands' intuitive card-stacking interface, AND ALSO a grand strategy
conquest map with TFT-style auto-battles where skill synergies chain-explode
during resolution.

The stacking isn't just visual flavor — it's the **complexity management
solution** that makes grand map conquest playable in 50 minutes. One stack = one
decision unit. You manage 3-5 stacks, not 30 individual units.

---

## Player Experience Analysis (MDA Framework)

### Target Aesthetics (What the player FEELS)

| Aesthetic | Priority | How We Deliver It |
| ---- | ---- | ---- |
| **Sensation** | 5 | Battle report narration — watching synergies chain-fire is viscerally satisfying |
| **Fantasy** | 2 | Three Kingdoms conquest — commanding legendary generals across a historical map |
| **Narrative** | 3 | Emergent war stories through battle reports — each report is a mini-narrative of your decisions playing out |
| **Challenge** | 1 | Multi-front resource allocation with incomplete information — "which stack goes where?" |
| **Fellowship** | N/A | Single-player PvE |
| **Discovery** | 4 | Finding new synergy combinations, discovering optimal stack compositions for terrain |
| **Expression** | 6 | Build diversity — early aggro vs late-game scaling, cavalry rush vs siege specialist |
| **Submission** | N/A | Not a relaxation game — always under strategic pressure |

### Key Dynamics (Emergent player behaviors)

- Players will experiment with general + troop + terrain combinations seeking
  optimal synergies for each battlefield
- Players will plan multi-turn arcs: "I'll defend north with Zhang Fei while
  building my Guan Yu cavalry stack for the southern offensive in 3 turns"
- Players will read AI intent signals and make agonizing triage decisions about
  which threats to address
- Players will screenshot epic battle reports where their planning paid off
  perfectly

### Core Mechanics (Systems we build)

1. **Card Stacking** — Drag troops/sub-generals onto a main general to form a
   combat stack, limited by Command stat
2. **Turn-Based Deployment** — Assign each stack a mission: attack, defend,
   train, garrison, or rest
3. **Auto-Battle Resolution** — Fixed-phase combat (pre-battle → clash →
   aftermath) with clear cause-and-effect chain
4. **Grand Map Conquest** — 20-40 regions with terrain, resources, and AI
   warlords racing to expand
5. **Battle Report Narration** — Turn-by-turn text replay showing exactly which
   cards triggered, why you won/lost, and what to change next time

---

## Player Motivation Profile

### Primary Psychological Needs Served

| Need | How This Game Satisfies It | Strength |
| ---- | ---- | ---- |
| **Autonomy** | Full control over stack composition, deployment strategy, and pacing (aggro vs late-game) | Core |
| **Competence** | Mastery of synergy combinations, reading AI patterns, optimal resource allocation | Core |
| **Relatedness** | Connection to legendary Three Kingdoms characters and their historical relationships | Supporting |

### Player Type Appeal (Bartle Taxonomy)

- [x] **Achievers** — Map completion percentage, general collection, scenario victories
- [x] **Explorers** — Discovering synergy combinations, testing unusual stack compositions
- [ ] **Socializers** — Not primary (single-player), but shareable battle reports
- [ ] **Killers/Competitors** — AI conquest provides competitive satisfaction without PvP toxicity

### Flow State Design

- **Onboarding curve**: Tutorial scenario (Yellow Turban Rebellion) with 20
  regions, 2 weak AI opponents, pre-built starting stack. Teaches one system per
  turn for 5 turns, then opens up.
- **Difficulty scaling**: AI warlords have escalating aggression and strategy;
  later scenarios add more opponents and tighter resource constraints.
- **Feedback clarity**: Battle reports show exactly why each battle was won/lost.
  Deployment preview shows advantage/disadvantage before committing. Every
  failure is a learning moment.
- **Recovery from failure**: Losing a region isn't game over — comeback arcs are
  intentionally supported. Only losing ALL regions ends the game.

---

## Core Loop

### Moment-to-Moment (30 seconds)

Drag a card onto a stack → see Command slots fill → check the terrain preview
→ commit deployment. The satisfying "click" is when a synergy forms: "Guan Yu
(Cavalry/Fierce) + Cavalry + Cavalry + Plains terrain = Breakthrough stack."

**Result validation**: After resolution, the battle report narrates your
decisions in story form. This is the ultimate payoff — your planning becomes
a war story worth reading.

### Short-Term (5-15 minutes = 1 turn)

Intelligence report → resource collection → deployment decisions → end turn →
watch all battles resolve simultaneously → read reports → plan next turn.

"One more turn" comes from ALL directions simultaneously:
- Power escalation: "My new general makes my stack even stronger"
- Map pressure: "The AI is about to take that grain region — I must respond"
- Unfinished build: "One more cavalry card and my Guan Yu stack is complete"
- Revenge: "Zhang Jiao took my border region — I'm coming for his capital"

### Session-Level (50-70 minutes = 1 scenario)

Start with 3 regions and 3 generals → expand across 20-35 regions → face
escalating AI pressure → commit to a strategic arc (early aggro or late scaling)
→ execute a decisive campaign → conquer the scenario.

Natural arc: Early struggle → mid-game turning point → late-game dominance
(or desperate comeback).

### Long-Term Progression

- **General Codex**: Collect all 80 generals across scenarios
- **Merit Upgrades**: Permanent mild bonuses (starting general +1, starting gold
  +50) — convenience, not power
- **Scenario variety**: 4 scenarios with different map layouts, starting generals,
  and victory conditions
- **Build mastery**: Learning which compositions dominate which terrains and
  matchups

### Retention Hooks

- **Curiosity**: "What if I ran an all-cavalry Lv Bu rush? What about a Zhou Yu
  fire attack build on the river map?"
- **Investment**: General codex progress, merit upgrades, scenario completion
- **Social**: Shareable battle reports and legendary victories
- **Mastery**: Optimizing stack compositions, achieving faster conquests, winning
  on higher difficulties

---

## Game Pillars

### Pillar 1: Stack = Simplicity

Card stacking isn't just visual flavor — it's the core design invention that
makes grand map conquest playable in 50 minutes. One stack = one decision unit.
Managing 3-5 stacks instead of 30 individual units keeps decisions meaningful
without overwhelming.

*Design test*: "If we're debating whether to let players micromanage individual
units vs. operating as stacks, this pillar says: operate as stacks."

### Pillar 2: Decision Before Action

All meaningful choices happen in the deployment phase. Battle is an auto-resolved
validation moment, not an operation moment. The player's skill is in reading the
situation and committing resources — not in reaction speed or APM.

*Design test*: "If we're debating whether to add mid-battle player actions
(dodging, skill activation, target switching), this pillar says: don't."

### Pillar 3: Build-Up to Payoff

Players must be able to choose their strategic arc — endure early and explode
late, or pressure early and crush threats in the cradle. The game must support
multiple pacing strategies, not force a single rhythm.

*Design test*: "If a mechanic makes every game play out with the same tempo
(e.g., always rush, or always turtle), this pillar says: that mechanic is broken."

### Pillar 4: Readable Pressure

Map threats must be telegraphed in advance (AI intent preview) but arrive on
multiple fronts simultaneously so the player can't perfectly respond to
everything. Pressure comes from agonizing triage, not from ambush.

*Design test*: "If the AI can take a critical region with zero prior warning,
this pillar says: that's a design bug."

### Anti-Pillars (What This Game Is NOT)

- **NOT real-time action**: No twitch reactions, no QTE, no manual aiming.
  Battle validates planning, not reflexes.
- **NOT an economy sim**: No multi-resource production chains, no citizen AI,
  no factory automation. Economy exists only to fuel combat decisions.
- **NOT full information**: Players can't see enemy stack internals by default.
  Uncertainty is a strategy resource — scouting has value.
- **NOT infinite collection**: Card library has hard caps. Limits = trade-offs
  = strategy. No hoarding.

---

## Visual Identity Anchor

*Note: To be fully defined during `/art-bible`. Preliminary direction below.*

**Direction**: Historical Ink & Strategy (水墨策略风)

- **Visual rule**: "Cards feel like pieces on a war table — clean, readable,
  with weight"
- **Principles**:
  - Clarity over decoration — stacked cards must be instantly readable at any
    zoom level
  - Ink-brush character portraits — distinctive silhouettes that read at card
    size
  - Map as terrain painting — regions are geographic, not grid-based
- **Color philosophy**: Faction-coded warm vs cool tones (player: vermillion/gold;
  enemies: varied blues, greens, purples). Terrain uses muted earth tones so
  cards pop against the map.

---

## Inspiration and References

| Reference | What We Take From It | What We Do Differently | Why It Matters |
| ---- | ---- | ---- | ---- |
| **Stacklands** | Intuitive card-stacking UI, drag-to-combine physicality | Add grand strategy map layer; stacking is for combat formation, not resource crafting | Proves stacking UI is intuitive and satisfying (96% Steam rating, 500K+ sales) |
| **Teamfight Tactics** | Auto-battle resolution, composition synergies, draft/build decisions | Single-player PvE on conquest map; no real-time draft pressure, more strategic planning time | Proves auto-battle + composition building sustains 33M+ monthly players |
| **率土之滨** | Grand map conquest, general collection, battle report as narrative, seasonal arcs | Single-session (50min vs 2-3 month season); no pay-to-win; offline single-player | Proves Three Kingdoms + strategy + battle reports has massive audience (billions in revenue) |

**Non-game inspirations**:
- Romance of the Three Kingdoms (novel) — dramatic arcs of underdog generals
  rising through brilliant strategy
- Chess/Go — the satisfaction of reading 3 moves ahead, the weight of commitment
- War table briefings in films — moving pieces on a map, committing forces,
  watching reports come in

---

## Target Player Profile

| Attribute | Detail |
| ---- | ---- |
| **Age range** | 20-40 |
| **Gaming experience** | Mid-core to Hardcore strategy players |
| **Time availability** | 50-70 minute sessions; 2-3 sessions per week |
| **Platform preference** | PC primary, mobile secondary (commute sessions) |
| **Current games they play** | TFT, 率土之滨, Slay the Spire, Into the Breach, Stacklands |
| **What they're looking for** | Deep strategy without time commitment of full 4X; single-session completion; "I outsmarted the system" feeling |
| **What would turn them away** | Real-time pressure, pay-to-win, grinding without strategy depth, RNG without counterplay |

---

## Technical Considerations

| Consideration | Assessment |
| ---- | ---- |
| **Recommended Engine** | Godot 4 — lightweight, exports to PC + Mobile, GDScript for rapid iteration, strong 2D support |
| **Key Technical Challenges** | Card drag-and-drop physics feel; AI behavior system (intent + expansion); battle resolution engine; three-layer map zoom system |
| **Art Style** | 2D stylized — ink-brush portraits + clean card design + painted map |
| **Art Pipeline Complexity** | Medium (custom 2D card art, but can start with placeholder icons) |
| **Audio Needs** | Moderate — ambient Three Kingdoms music, card placement SFX, battle resolution audio cues |
| **Networking** | None (single-player offline) |
| **Content Volume** | 12-18 generals, 5 troop types, 10-12 tactic cards, 20-35 map regions per scenario, 4 scenarios total |
| **Procedural Systems** | Minimal — AI behavior has randomized intent within parameters; battle rewards have weighted random selection |

---

## Risks and Open Questions

### Design Risks

- **Session length vs depth**: 50-70 minutes must feel complete without being
  shallow. If conquest is too easy it's boring; too hard it exceeds time budget.
- **Stack readability at scale**: Late-game with 5+ stacks across 30+ regions —
  can the player maintain situational awareness?
- **AI predictability**: If AI intent is too readable, no tension. If too opaque,
  violates Pillar 4 (Readable Pressure).

### Technical Risks

- **Card stacking UX on mobile**: Drag-and-drop with small cards on touch
  screens — needs careful hitbox and gesture design.
- **Three-layer map zoom**: Smooth transitions between card-level, region-level,
  and strategic views without performance issues.
- **AI system complexity**: Even "simple" AI with expansion logic, intent
  signaling, and territory control is non-trivial for a first project.

### Market Risks

- **Three Kingdoms fatigue**: Extremely crowded IP space in Chinese market —
  needs clear mechanical differentiation.
- **Single-player premium in mobile**: Mobile players expect F2P; premium
  single-player strategy has smaller audience on mobile.
- **Comparison to existing games**: Players may expect 率土之滨 multiplayer
  depth or Stacklands production values from day one.

### Scope Risks

- **Content volume**: 12-18 unique generals with balanced skills is significant
  design + art work for solo dev.
- **Balance tuning**: Terrain modifiers + troop counters + skill synergies
  creates a large interaction matrix to balance.
- **First game ambition**: Grand strategy + card game + auto-battler is three
  genres combined — each has pitfalls.

### Open Questions

- **Is stacking sufficient complexity?** Does the stack interface deliver enough
  strategic depth, or does it need additional decision layers? → Prototype will
  answer (T0).
- **What's the right AI difficulty curve?** How aggressive should AI warlords be
  at each scenario difficulty? → Playtest data needed.
- **Mobile card size**: What's the minimum card size for readable stacking on
  phone screens? → UX prototype needed.
- **Battle report pacing**: How detailed should reports be before they become
  tedious? → Player testing required.

---

## MVP Definition

**Core hypothesis**: "Stacking cards into combat formations and deploying them
across a small map creates meaningful strategic decisions that are fun to resolve
through auto-battle — and battle reports make the payoff satisfying."

**Required for MVP (T1)**:
1. Card stacking mechanic — drag troops/sub-generals onto main general, limited
   by Command stat
2. 5-region mini-map with 1 AI opponent
3. Turn-based deployment (attack/defend only — no train/garrison/rest)
4. Auto-battle resolution with text battle report
5. 3-5 generals, 2 troop types, 3 tactic cards
6. Win condition: conquer all 5 regions

**Explicitly NOT in MVP**:
- Full 20+ region map
- Multiple AI opponents
- Training/garrison/rest missions
- City building
- General recruitment system
- Long-term progression (codex, merit upgrades)
- Music, sound effects, polished art
- Mobile adaptation

### Scope Tiers

| Tier | Content | Features | Timeline |
| ---- | ---- | ---- | ---- |
| **T0 — Core Mechanic** | 3 generals, 2 troop types | Single battle: stack → resolve → report. No map. | 2-4 weeks |
| **T1 — MVP** | 5 regions, 1 AI, 5 generals, 2 troops, 3 tactics | Mini-map conquest + deployment + auto-battle + reports | 6-10 weeks |
| **T2 — Tutorial Scenario** | 20 regions, 2 AI, 12 generals, 5 troops, 10 tactics | Full Yellow Turban scenario + all 5 missions + city building + recruitment | 4-6 months |
| **T3 — Full Vision** | 4 scenarios, 80 generals, codex, merit system, balance pass | Complete game with all scenarios, progression, and polish | 1+ year |

---

## Next Steps

- [ ] Configure engine with `/setup-engine` (Godot 4, PC + Mobile)
- [ ] Prototype core stacking mechanic with `/prototype` — validate T0 before
      investing in full design
- [ ] If prototype PROCEEDS: create art bible with `/art-bible`
- [ ] Decompose concept into systems with `/map-systems`
- [ ] Author per-system GDDs with `/design-system`
- [ ] Plan architecture with `/create-architecture`
- [ ] Validate readiness with `/gate-check`
