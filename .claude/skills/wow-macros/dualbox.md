# Dual boxing: playing 2 characters at once

## Rules (Blizzard policy since 2020)

- Running several accounts at once is **allowed**.
- **Input broadcasting is banned.** That covers any software or hardware that sends one key
  press to several game clients. **One key press is one action, in one client.** Switching
  windows by hand (Alt-Tab, or a window-switcher key) and pressing keys in each one is fine.
- So the follower character needs macros that make each of its key presses do as much as
  possible on its own. Assisting and following are what make that work.

Terms used below: **Main** is the character you actively play. **Follower** (or **Alt**) is the
second character. Replace `Main` with the real character name. On a cross-realm group, add the
realm: `Name-Realm`.

## Setup on the follower (one time)

- Prefer **`/startattack` in the macro** over the option Interface → Combat → **Attack on
  Assist** (CVar `assistAttack`). The option makes **every** assist start melee auto-attack,
  including on casters and healers, which can break crowd control or pull mobs. `/startattack`
  only fires in the macros that contain it. The option is only useful if you assist with the
  default key instead of a macro.
- Keep **Auto Self Cast** on, so a friendly spell cast with no friendly target lands on yourself.
- Optional addons: BoxTrack (Classic Era, role setup wizard), plus auto-accept for invites,
  quests and summons.

## Two ways to pick up the main character's target

| Method | Example | Changes follower's target? | Works when |
|---|---|---|---|
| `/assist Main` | `/assist Main` then `/cast Frostbolt` | Yes, it takes over Main's target | Always, in party, raid or solo. Most robust |
| Cast at Main's target directly | `/cast [@party1target,harm,nodead][] Frostbolt` | **No** | Only in a 2-person party, where `party1` is always the other character. In a raid, `party1` is someone from your subgroup |

Assisting is simpler, and works well with `/startattack` in the same macro. The direct-cast version keeps
the follower's own target free, for example so it can stay targeted on the main character for
heals. You can also mix them: assist with no modifier, and the follower's own target with Shift
held.

## Follower macros

**Follow (press again after every fight or jump):**
```
/follow Main
```
`/follow Name` follows directly, without changing your target. `/tar Name` followed by `/follow`
also works, but it throws away your current target. That version is in the user's old
`+follow naestra` macro.

**Assist + melee attack (melee follower):**
```
#showtooltip
/assist [nomod] Main
/startattack
/cast Sinister Strike
```

**Assist + spell (caster follower):**
```
#showtooltip
/assist [nomod] Main
/cast Frostbolt
```
Hold Shift to skip the assist and cast at the follower's own target. When Main has no target,
`/assist` prints an error and the follower keeps its own target.

**Cast at Main's target without changing target (2-person party only):**
```
#showtooltip
/cast [@party1target,harm,nodead][harm,nodead] Shadow Word: Pain
```

**Heal or buff the main character, mouseover first, no target change:**
```
#showtooltip
/cast [@mouseover,help,nodead][@Main,help,nodead][@player] Flash Heal
```
With `[mod:shift]`, heal yourself:
```
#showtooltip
/cast [mod:shift,@player][@mouseover,help,nodead][@Main,nodead] Flash Heal
```

**Healer that DPSes only while Alt is held:**
```
#showtooltip
/assist [mod:alt] Main
/cast [mod:alt,harm] Smite; [@Main,nodead] Heal
```

**Hunter or Warlock follower: send the pet at Main's target:**
```
/assist Main
/petattack
```

**Catch up after a fight (drop target, stop attacking, follow):**
```
/stopattack
/cleartarget
/follow Main
```

## Macros for the main character

**Mark the kill target (the follower can then see what to attack):**
```
/run SetRaidTarget("target",8)
```

**Invite the follower:**
```
/invite Follower
```

**Stop the follower's auto-attack before crowd control:** this can't be done from the main
character's client. Press `/stopattack` on the follower before you sheep or sap.

## Limits

- `/follow` breaks when the follower is moved by hand, gets stunned, or the main character
  mounts a flight path or takes a portal.
- Follow doesn't work inside some instances or through doors and portals. Expect to steer the
  follower by hand there.
- The same `/assist Main` key on the follower does nothing if Main is dead or out of range.
  Name-based `[@Main]` needs Main to be in your group.
- WoW Forever ⚠: modern-API restrictions may limit chat and raid-marker macros (see
  reference.md §Forever). Assist and follow are expected to work unchanged. Test them in-game.
