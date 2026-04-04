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

---

## Suggested next improvements

- Expand faith-specific signature modifiers to all sect faiths (not only current keyed examples).
- Move large realm-comparison trigger ladders to reusable scripted effects/values for maintainability.
- Add dedicated debug decision/event to print current cultivation state (realm, meridian, sect flavor modifiers, pulse eligibility).

