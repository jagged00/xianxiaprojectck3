# Codex Changelog & Handoff Notes

This file is for future Codex instances working on this CK3 xianxia/cultivation mod.
It summarizes *what has been changed recently*, *why*, and *what constraints to preserve*.

## Current design goals (do not regress)

1. **Players should not passively auto-progress realms via AI pulses.**
   - Player realm advancement should primarily use breakthrough decisions/events.
   - AI monthly simulation should remain AI-only for progression logic.

2. **AI cultivators should feel lore-accurate for xianxia/jianghu.**
   - Fast/high progression for talented meridians.
   - Social hierarchy pressure from higher realms.
   - Personality-driven behavior toward talented juniors.

3. **Sect identity should not feel copy-paste.**
   - Keep doctrine-level realm flavor modifiers.
   - Keep sect/faith-specific signature modifiers (e.g., Divine Beast spirit companion).

---

## Major systems added/changed

### 1) Player breakthrough cooldown
- Breakthrough decisions were updated so players get `player_breakthrough_cooldown` (5 years) after attempting breakthroughs.
- AI bypasses this cooldown gate.
- Intention: stop player spam while preserving world simulation pacing.

### 2) AI monthly cultivation pulse expansion
- `on_monthly_pulse` runs cultivation AI events including:
  - `cultivation_ai.0001` progression
  - `cultivation_ai.0002` sect joining
  - `cultivation_ai.0003` weaker-liege resentment behavior
  - `cultivation_ai.0004` personality-driven treatment of talented juniors
  - `cultivation_ai.0005` sect/doctrine flavor modifier assignment

### 3) AI-only guard restored for progression
- `cultivation_ai.0001` trigger includes `is_ai = yes`.
- This is critical and must be kept to prevent player passive realm jumps.

### 4) Higher-realm political pressure
- Added trigger `has_higher_cultivation_realm_than_liege`.
- Higher-realm cultivators apply strong resentment toward weaker lieges.
- Exception logic exists for parent/close-dynasty-family lieges to avoid immersion-breaking behavior.

### 5) Personality-driven “genius handling”
- High-realm AI personalities branch behavior:
  - Ruthless/cunning profiles suppress talented lower-realm cultivators.
  - Benevolent profiles invest in talented juniors.
- This is implemented in monthly AI event logic and tied to personality traits.

### 6) Sect flavor after perk unlocks
- Added `has_any_cultivation_realm_perk` trigger.
- It now includes alternate perk branch nodes (not just one node per realm) to avoid false negatives.
- Once true, sect-specific flavor modifiers are applied via `cultivation_ai.0005`.

#### Doctrine-level realm flavor modifiers
- `orthodox_realm_insight`
- `unorthodox_realm_insight`
- `demonic_realm_insight`
- `vagrant_realm_insight`

#### Faith-specific signature modifiers (examples)
- `divine_beast_spirit_companion` (prowess-focused spirit beast fantasy)
- `wudang_taiji_harmony`
- `jade_chamber_alluring_presence`
- `blood_moon_killing_intent`
- `ancient_relic_treasure_resonance`



### 7) Parser/stability hotfixes (2026-04-05)
- Fixed multiple cultivation parser blockers that were preventing systems from loading fully:
  - `common/dynasty_legacies/cultivation_legacies.txt` now uses `legacy_tracks = { ... }` wrapper.
  - Replaced deprecated trigger/effect forms in cultivation scripts (`has_liege` -> `exists = liege`, `is_child` checks -> `is_adult = no`, `set_character_flag` -> `add_character_flag`).
  - Corrected on_action wiring to use non-zero monthly orchestrator event id (`cultivation_ai.2000`) and made startup bootstrap event `cultivation_ai.1000` use `scope = none`.
  - Fixed typoed breakthrough option loc key reference (`cultivation_breakthrough.1400.wait`).

### 8) Breakthrough unstick guard
- Breakthrough decisions now allow either:
  - legacy `can_attempt_*` character flags, or
  - corresponding breakthrough perks.
- Intention: prevent characters from getting stuck at early realms (especially Qi Gathering/Qi Refining) when migrated saves or script ordering leave perks present but flags missing.
- Keep this behavior unless you add a reliable global flag-sync migration effect.

### 9) Dual cultivation dependency decoupling
- Replaced the old Carnalitas-dependent `dual_cultivation_interaction` logic with a cultivation-native interaction baseline.
- Rationale: avoid hard parser/runtime failures when external Carnalitas scripted triggers/effects are absent.
- If Carnalitas integration is desired again, reintroduce it behind safe compatibility gating (separate optional file/module), not in the core cultivation path.

### 10) Self-healing qi purge decision (2026-04-05)
- Added `purge_mortal_afflictions_decision` for cultivators above Qi Gathering (Qi Refining+).
- Decision now requires at least one qualifying disease/injury trait to be valid.
- Decision removes major disease/injury traits and applies temporary stabilization via `realm_stabilization`.
- Includes a 1-month cooldown (`purge_mortal_afflictions_cooldown`) and is tuned for frequent AI self-cleanse behavior.

### 11) Sect-wide defensive pact pressure (2026-04-05)
- Added a same-sect defensive pact orchestration pulse (`cultivation_ai.3000`).
- Triggered at game start and yearly, it forces independent cultivator rulers of the same sect faith to ally, creating larger sect-coalition conflicts over time.

### 12) Realm-tier hostile scheme rebalance (2026-04-05)
- Rebalanced cultivation realm traits so lower realms are significantly more vulnerable to hostile schemes, while high realms (especially Heavenly Being+) are progressively harder to target.
- Added escalating `owned_personal_scheme_success_chance_add` by realm so higher-realm cultivators become increasingly oppressive scheme initiators against lower realms.
- Design intent: "no cheesy murder plots" against top cultivators, while preserving predatory pressure downward in the realm hierarchy.

### 13) Meridian-first marriage AI + crippled-prisoner enforcement (2026-04-05)
- Marriage acceptance now heavily prioritizes superior meridians, strongly favors heavenly/excellent foundations, and applies near-absolute refusal for `crippled_meridians` candidates.
- Existing `cripple_cultivator_interaction` remains the canonical way to permanently mortalize imprisoned cultivators via `cultivation_cripple_mortalize_effect`.
- AI willingness to use the crippling interaction is now non-zero and strongly weighted toward ruthless/high-realm actors.

### 14) Bloodline exception marriage logic + cultivator fertility safeguard (2026-04-05)
- Refined the `crippled_meridians` marriage taboo so bloodline traits are now the only acceptance exception path.
- Added scripted trigger `has_any_cultivation_bloodline_trait` and used it to enforce:
  - extreme refusal for blocked-meridian matches without bloodline traits, and
  - a large acceptance override only when the blocked-meridian target carries a cultivation bloodline trait.
- Increased bloodline marriage valuation weights so bloodline pairings are materially prioritized in sect matchmaking.
- Added monthly fertility safeguard event (`cultivation_ai.2100`) that strips `infertile` from cultivators.
- Removed the True Immortal fertility penalty (`fertility = 1.0`) to avoid top-realm sterility by trait balance.

---

## Practical guidance for future Codex instances

1. **Before changing progression logic:**
   - Confirm `cultivation_ai.0001` remains AI-only.
   - Ensure player advancement still respects breakthrough decision flow.

2. **Before changing perk-based sect flavor:**
   - Update `has_any_cultivation_realm_perk` if perk trees are renamed/branched.
   - Keep both doctrine-level and faith-specific flavor layers coherent.

3. **Before changing social behavior events:**
   - Re-check family exceptions in weaker-liege resentment.
   - Keep personality branches legible and lore-consistent (ruthless vs benevolent archetypes).

4. **When adding new sect faiths:**
   - Add a signature modifier + localization.
   - Add assignment logic in `cultivation_ai.0005`.

5. **When balancing numbers:**
   - Prefer small iterative tuning; many systems interact (progression, perks, opinions, death chance, AI pulses).

6. **When touching compatibility-sensitive script APIs:**
   - Prefer currently valid CK3 scripted keys over legacy aliases.
   - Avoid introducing hard dependencies on external-mod-only scripted triggers/effects in core cultivation files.

7. **If breakthrough progression appears stuck in saves:**
   - Verify decision availability against both perk state and `can_attempt_*` flags.
   - Add/maintain migration-safe fallback checks before tightening eligibility.

8. **When adding body-cleansing/healing cultivation decisions:**
   - Gate them at an appropriate realm threshold (do not allow at Mortal/Qi Gathering unless intended).
   - Use an explicit affliction requirement so the decision is unavailable when there is nothing to purge.
   - Use explicit cooldown values that match design intent (currently 1 month) and keep effects to disease/injury cleanup rather than broad personality-trait rewriting.

9. **When adding sect solidarity/alliance automation:**
   - Prefer periodic orchestration (game start + yearly) over heavy monthly full-world alliance loops.
   - Restrict to independent rulers with cultivation sect faiths to avoid runaway alliance spam across non-sect realms.

10. **When tuning realm-level intrigue/scheme balance:**
   - Keep a steep vulnerability gradient: low realms should be plot-vulnerable, high realms should be near-untouchable.
   - Pair defense scaling (`enemy_hostile_scheme_success_chance_add`) with offense scaling (`owned_personal_scheme_success_chance_add`) so high realms can still dominate lower realms politically.

11. **When tuning marriage AI for cultivation dynasties:**
   - Treat `crippled_meridians` as a hard block unless the target has a cultivation bloodline trait.
   - Keep bloodline valuation high enough that bloodline carriers can override generic political noise when needed.

12. **When safeguarding cultivator fertility:**
   - Keep a recurring cleanup pass for hard infertility traits (currently monthly `cultivation_ai.2100`).
   - Avoid fertility penalties on late-realm cultivation traits unless explicit design requires sterility.

---

## Suggested next improvements

- Expand faith-specific signature modifiers to all sect faiths (not only current keyed examples).
- Move large realm-comparison trigger ladders to reusable scripted effects/values for maintainability.
- Add dedicated debug decision/event to print current cultivation state (realm, meridian, sect flavor modifiers, pulse eligibility).
