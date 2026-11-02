# WoW macro reference: Classic Era 1.15.x and WoW Forever 1.60.x

Both games run on the modern Classic client, so they use the **modern macro syntax**
(`[@unit]`, `;` between clauses, `/castsequence`, and so on), not the vanilla 1.12 private-server
syntax. Anything marked ⚠ is unconfirmed. Test it in-game before relying on it.

## Line types

| Line | Meaning |
|---|---|
| `#showtooltip [Spell\|Item\|slot]` | Shows the icon and tooltip of the first castable line, or of the argument you give. Accepts conditionals: `#showtooltip [stealth] Pounce; Bash` |
| `#show [Spell]` | Shows the icon only, with no tooltip |
| `/cast`, `/use` | Casts a spell or uses an item. They are interchangeable. Takes conditionals, `Spell(Rank n)`, an item name, a slot number, or `bag slot` |
| `/castsequence [cond] reset=N/target/combat/shift/ctrl/alt A, B, C` | Moves to the next spell each time the current one succeeds. Gets stuck if a spell is on cooldown or fails |
| `/castrandom A, B` / `/userandom` | Casts a random spell or uses a random item from the list |
| `/stopcasting` | Interrupts your own cast. Put it first in interrupts and escapes |
| `/startattack [cond]` / `/stopattack` | Turns melee auto-attack on or off |
| `/stopmacro [cond]` | Stops the macro here if the condition is true |
| `/cancelaura Name` | Removes a buff, for example `/cancelaura Blessing of Salvation`. Works in combat for most buffs |
| `/cancelform [cond]` | Leaves a shapeshift form |
| `/dismount [cond]` | Dismounts |
| `/target Name`, `/targetexact Name`, `/tar` | Targets a unit by name (partial match, except `/targetexact`) |
| `/targetenemy [cond]`, `/targetfriend`, `/targetlasttarget`, `/assist [@unit]`, `/cleartarget [cond]` | Targeting commands |
| `/petattack [@unit]`, `/petfollow`, `/petstay`, `/petpassive`, `/petdefensive`, `/petaggressive`, `/petautocasttoggle Spell`, `/petdismiss` | Pet commands |
| `/equip Item`, `/equipslot N Item` | Equips gear. You can't change weapons mid-swing, and a shield needs `/equipslot 17` |
| `/click FrameName [button]` | Clicks a UI button. Used to chain secure buttons, for example macros longer than 255 characters with an addon |
| `/run code`, `/script code` | Lua for UI work only. It can't cast, use items or target. Example: `/run SetRaidTarget("target",8)` |
| `/console cvar value` | Changes a game setting (CVar) |
| `/follow`, `/trade`, `/timer`, `/invite`, `/ra`, `/p`, `/w` | Social and utility commands |

**Hiding error spam** (for example "Ability not ready yet"):
```
/console Sound_EnableErrorSpeech 0
... your casts ...
/console Sound_EnableErrorSpeech 1
/run UIErrorsFrame:Clear()
```

## Conditional syntax

- `[a,b]` means a **and** b. `[a][b]` means a **or** b (the first clause that matches wins).
- `no` reverses a condition (`nocombat`, `noharm`). Inside one condition, `/` means **or**:
  `[form:1/3]`, `[mod:shift/ctrl]`.
- `[@unit]` (alias `target=unit`) sets which unit the conditions check and which unit the spell
  is cast on.

### Units
`player`, `target`, `targettarget`, `mouseover`, `pet`, `party1-4`, `raid1-40`, `partypet1-4`,
`@cursor` (ground-targeted spells only, for example `[@cursor] Flare`, `[@cursor] Blizzard`),
`@none`.
`partyNtarget`, `raidNtarget`, `pettarget` mean "the target of that unit". They let you cast on
another character's target without changing your own target. A **character name** also works as a
unit when that character is in your group: `[@Mainchar]`.
**`focus` doesn't exist in Classic Era.** In modern WoW, focus is a second, saved target that
you set with `/focus` and cast at with `[@focus]`, without changing your normal target. In Era,
focus addons only add their own frames and generate macros. On Forever ⚠ it's unknown whether
focus exists. Instead of focus, use `[@Name]` or `[@party1target]`.

### Conditions checked on the unit (default `@target`)
`exists`, `help`, `harm`, `dead`, `party`, `raid`

### Conditions checked on the player
| Conditional | Notes |
|---|---|
| `combat` | You are in combat |
| `stealth` | Rogue Stealth, Druid Prowl, Shadowmeld |
| `form:n` / `stance:n` | Same thing under two names. `[form]` with no number means any form. Indexes are below |
| `equipped:type` / `worn:type` | `Shields`, `Daggers`, `Two-Handed Swords`, `Bows`, `Guns`, `Crossbows`, `Thrown`, `Wands`, `Fishing Poles`, or a slot name such as `Main Hand` |
| `channeling` / `channeling:Spell` | For example `[nochanneling:Arcane Missiles]` |
| `mounted`, `swimming`, `indoors`, `outdoors`, `resting` | `flying` and `flyable` are always false in Era |
| `pet`, `pet:Name`, `pet:Family` | For example `[nopet] Call Pet`, `[pet:Imp]` |
| `group`, `group:party`, `group:raid` | |
| `known:Spell` | ⚠ Retail added it in 10.0.2. It may not exist in Era |
| `spec:1` / `spec:2` | Era doesn't have it. ⚠ Forever may, since it has dual spec |

### Conditions checked on the UI
`mod`, `mod:shift/ctrl/alt`, `button:1-5` (mouse button used on the action button, so
`[button:2]` means right-click), `actionbar:n`, `bonusbar:n`, `cursor`, `shapeshift`.

## /castsequence

```
/castsequence [conditions] reset=<rules> Spell1, Spell2, Spell3
```
- Each press tries **the next spell in the list**. The sequence only advances when that spell
  **actually starts**. If it fails (GCD, out of range, no mana, on cooldown), the next press tries
  the same spell again.
- After the last spell it starts over from the first.
- `reset=` rules, combined with `/` (for example `reset=8/target/combat`):
  - `N`: go back to Spell1 if you haven't pressed the button for N seconds. Each press restarts
    the timer.
  - `target`: go back when you change target.
  - `combat`: go back when you leave combat.
  - `shift`, `alt`, `ctrl`: go back when you press the button with that key held.
- Items and ranks work: `/castsequence Healthstone, Major Healing Potion`.
- With `;` you can have different sequences per condition:
  `/castsequence [mod:shift] A, B; C, D`.
- `#showtooltip` with no argument shows the **next** step.

**When to use it:** spells with **no cooldown** that should come in a fixed order. Examples are
applying DoTs on a new target, a series of buffs, or a consumable chain.

```
#showtooltip
/castsequence reset=target Corruption, Curse of Agony, Immolate
```
```
#showtooltip
/castsequence reset=60 Power Word: Fortitude, Divine Spirit, Shadow Protection
```
**Don't use it for:** rotations with cooldown abilities. When Spell2 is on cooldown the sequence
stays stuck on it, and pressing the button does nothing until the cooldown ends or a reset
happens. A resisted spell still counts as cast and is skipped.
⚠ **The "null" trick:** `/castsequence reset=30 Hunter's Mark, null` casts Mark once and then
does nothing until the reset. It's popular on retail, but test it in Classic.

## Focus-style healing priority

On **Forever** ⚠, if `@focus` exists there, set the focus (usually the tank) once:
```
/focus [@mouseover,exists,nodead][]
```
Then heal in the order mouseover, then target, then focus, then yourself:
```
#showtooltip
/cast [@mouseover,help,nodead][help,nodead][@focus,help,nodead][@player] Flash Heal
```
- With `[help,nodead]` second, a friendly target wins over focus. When you're targeting the boss,
  the heal goes to the focused tank.
- Other uses of focus: Polymorph or Sap on a second mob (`/cast [@focus,harm,nodead] Polymorph`),
  interrupts (`/cast [@focus,harm,nodead][] Counterspell`), and Misdirection-style tank
  helpers.

On **Classic Era**, which has no focus, use the tank's **name** in place of the focus. It works
for group members only:
```
#showtooltip
/cast [@mouseover,help,nodead][help,nodead][@Tankname,help,nodead][@player] Flash Heal
```
In a 2-person group (dual box), `@party1` works too, without needing a name.

## Form and stance indexes (Classic Era)

| Class | 1 | 2 | 3 | 4 | 5 | 6 |
|---|---|---|---|---|---|---|
| Warrior | Battle | Defensive | Berserker | | | |
| Druid | Bear / Dire Bear | Aquatic | Cat | Travel | Moonkin (talent) | |
| Rogue | Stealth | | | | | |
| Priest | Shadowform (talent) | | | | | |
| Paladin | Auras, in the order you learned them. ⚠ Check with `/run print(GetShapeshiftForm())` | | | | | |

Rule of thumb for Druids: the index follows the order of buttons on your stance bar. A Druid who
hasn't learned Aquatic Form yet has Cat Form at index 2. Check with
`/run print(GetShapeshiftForm())`.

## Equipment slot numbers

1 head · 2 neck · 3 shoulder · 4 shirt · 5 chest · 6 waist · 7 legs · 8 feet · 9 wrist ·
10 hands · 11–12 rings · 13–14 trinkets · 15 back · 16 main hand · 17 off-hand · 18 ranged ·
19 tabard. For bags, use `/use <bag 0-4> <slot>`.

## Proven patterns

```
# Melee opener + attack, target nearest enemy if none
#showtooltip
/targetenemy [noharm][dead]
/startattack
/cast Bloodthirst

# Heal: mouseover > target > self
#showtooltip
/cast [@mouseover,help,nodead][help,nodead][@player] Flash Heal

# Two actions on one button without modifier keys (game state decides)
#showtooltip
/cancelaura [stealth] Stealth
/cast [nostealth] !Stealth

# Pet ability by pet family
#showtooltip
/cast [pet:Wolf] Furious Howl; [pet:Cat] Prowl; [pet:Bat] Dive

# Buff series on pet, one press per scroll
#showtooltip
/castsequence [@pet] reset=60 Scroll of Agility IV, Scroll of Strength IV, Scroll of Stamina IV, Scroll of Protection IV

# Stance-dance (press twice when not in the right stance)
#showtooltip Overpower
/cast [nostance:1] Battle Stance
/cast Overpower
/startattack

# Druid powershift (leave form and re-enter on second press)
#showtooltip
/cancelform [form:3]
/cast !Cat Form

# Hunter shot that never cancels Auto Shot
#showtooltip
/cast Arcane Shot
/cast !Auto Shot

# Off-GCD stack before main spell
#showtooltip Arcane Power
/use 13
/cast Berserking
/cast Arcane Power

# Mage interrupt that also works from Ice Block
#showtooltip
/stopcasting
/cancelaura Ice Block
/cast Counterspell

# Cancel unwanted buffs (raid tanks / Salvation)
/cancelaura Blessing of Salvation
/cancelaura Greater Blessing of Salvation

# Self-cast an item buff (juju, scrolls on pet)
#showtooltip
/use [@player] Juju Might
```

## Common bugs

These were all found in the user's own files during a review on 2026-11-02:

- **`/cleartarget [dead,help]`** only clears a target that is dead **and** friendly. A dead
  enemy usually still counts as `harm`, so the next line, `/targetenemy [noharm]`, doesn't
  switch away from it either. Use one line instead: `/targetenemy [noharm][dead]`.
- **`[]` before another clause:** `[]` is always true, so in `/cast [][@mouseover,harm] Taunt`
  the mouseover clause is never checked. Put `[]` last.
- **`[@target][@player]`:** `@target` on its own has no condition, so it is always picked. Add a
  condition: `[@target,help][@player]`.
- **`[@mouseover,harm]` with no `[]` fallback:** the macro does nothing when the mouse isn't over
  a unit.
- **`/equipslot 17` with no item name:** does nothing. To empty a slot, use
  `/run PickupInventoryItem(17) PutItemInBackpack()` (only out of combat).
- **Unconditional `/cancelform` in a Moonkin spell:** it knocks you out of Moonkin Form. Use
  `/cancelform [noform:5]`.
- **Copying a template too far:** for example, adding the trap macro's `/cast [combat] Feign
  Death` line to Wyvern Sting, which can be cast in combat anyway.

- `#sowtooltip` or another typo in `#showtooltip`: the line is silently ignored.
- Spaces inside a rank, as in `Frost Nova (Rank 1)`: may fail. Use `Frost Nova(Rank 1)`.
- `/cast A` followed by `/cast B` on the same press when A triggers the GCD: B never casts. Use
  conditionals, or one macro per spell.
- `/castsequence` with spells that have a cooldown gets stuck (see §/castsequence).
- `[target=Totem Name]` / `[@Mob Name]`: names only work for **group members**. To hit a totem,
  use `/target Grounding Totem`, then `/petattack`, then `/targetlasttarget`.
- `/startattack` on Rogues and Druids breaks Stealth. Use `/startattack [nostealth]`.
- `/targetenemy` with no conditional switches away from a target you chose on purpose. Use
  `[noharm]`.
- `[@mouseover]` with no `[]` fallback does nothing when the mouse isn't over a unit.
- An equip command in combat while casting cancels the cast, and swapping weapons triggers a
  1.5-second attack cooldown.

## WoW Forever: what's different (from the beta, Sep–Oct 2026; ⚠ re-check after launch)

**Client and API**
- It's a separate install, with Interface 16001. Era addons don't load there. The UI API is
  closer to modern WoW, including "secret values" that hide some combat data from addons.
- The 255-character limit appears unchanged (players asked for more). The number of slots
  hasn't been published.
- Beta players confirmed that `[@mouseover,help,nodead]`, `[@mouseover,harm,nodead]` and
  `[stance:n]` work.
- Modern restrictions may also apply here ⚠. Retail has them as of 2026-03: at most 3 raid
  markers set by macro in quick succession, no posting to guild or community chat from macros
  during boss encounters, and a rate limit on party, raid and whisper chat from macros.
- There's dual spec (from level 40), so `[spec:1]`/`[spec:2]` may work ⚠.

**Class changes that affect macros**
- **Warlock:** Curse of Agony and Curse of Doom are renamed **Bane of Agony** and **Bane of
  Doom**. Only one Bane per Warlock can be on a target. Curse of Shadows was removed. Curse of
  the Elements now covers all magic schools. Firestone and Spellstone are now weapon imbues that
  stack with oils.
- **Warrior:** Thunder Clap works in Defensive Stance, with a 6-second cooldown. The *Vanguard*
  talent allows Charge in Defensive Stance, but beta players report it **doesn't work from a
  macro**. Put that Charge on a plain spell button. Tactical Mastery is learned at level 14 and
  keeps 10 Rage on stance change.
- **Hunter:** Aimed Shot is baseline at level 20. Multi-Shot and Aimed Shot share a cooldown.
  Traps can be used **in combat**, with a 30-second cooldown, so the `/cast [combat] Feign Death`
  trap trick isn't needed there. Frost and Fire traps have separate cooldowns.
- **Druid:** Potions and most consumables work while shapeshifted, so don't add `/cancelform`.
  Barkskin, Shadowmeld and War Stomp work in forms. Faerie Fire is free in Bear and Cat Form,
  with a 6-second cooldown. Heals still need `/cancelform`. Nature's Grasp is baseline at
  level 10.
- **Priest:** Prayer of Healing heals the **target's** party, which makes
  `[@mouseover,help][]` useful for it. Fear Ward is available to all Priests.
- **Rogue:** Sap no longer breaks Stealth, so `/cast [stealth] Sap` chains with your opener.
- New races and class combinations (Skyborne). Each race has 2 active racials, so check racial
  names before using them in off-GCD stacks.

Sources: warcraft.wiki.gg (Macro conditionals, Forever, Patch 1.60.1), wofwforever.com macro
guide, forevermeta.me addon guide, Blizzard forums (Forever beta addon feedback).
