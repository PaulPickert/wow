---
name: wow-macros
description: Write, review, fix and organize World of Warcraft macros for Classic Era (1.15.x) and WoW Forever (Classic+, 1.60.x), including editing the macros-cache.txt files in this repo. Use when the user asks for a macro, wants a macro improved or shortened, asks why a macro does not work, or wants macros added to or cleaned up in a character/account macro file.
---

# WoW macros: Classic Era and WoW Forever

Goal: produce macros that **work on the first try**, fit within **255 characters**, follow the
user's existing conventions, and can be pasted in-game or written straight into a
`macros-cache.txt` file.

Read [reference.md](reference.md) for the complete list of commands, conditionals, slot numbers,
form and stance indexes, and the differences between Classic Era and Forever. For macros used when
playing two characters at once (assist, follow, healing the main character without changing
target), read [dualbox.md](dualbox.md).

## 1. Find out which game the macro is for

| | Classic Era | WoW Forever |
|---|---|---|
| Repo folder | `classi_era/macros/` | `forever/macros/` (create on first use) |
| Client | 1.15.x, Interface 11508/11509 | 1.60.x, Interface 16001 (separate install, launched 2026-11-04) |
| Native focus (`@focus`, `/focus`) | **No.** Don't use it. | Unconfirmed. Test it first. |
| Dual spec `[spec:1]` / `[spec:2]` | No | Probably (dual spec unlocks at level 40). Test it first. |
| Class changes | Vanilla 1.12 behavior | Many changes; see reference.md §Forever |

If the user doesn't say which game, check which folder the file is in. If it's still unclear, ask.
Never assume a Forever behavior applies to Era, or the other way round.

## 2. Rules every macro must follow

1. **255 characters max** for the body, counting newlines. Check every macro with the count script
   in §5. The name can be at most **16 characters**.
2. **One GCD action per keypress.** Macros can't run a rotation. Lines after the first spell that
   triggers the GCD do nothing on the same press, unless that spell failed. Off-GCD actions can be
   stacked, for example a trinket, Berserker Rage or a racial ability before the main spell, or
   `/startattack`.
3. **No decisions based on game state** such as health, buffs, debuffs, cooldowns, mana, rage or
   combo points. Only the conditionals in reference.md exist. `/run` and `/script` can't cast,
   target or use items. They are only for UI chores such as printing, raid marks, clicking popups
   or equipping addon gear sets.
4. Start with **`#showtooltip`** so the button shows the spell's icon, cooldown and tooltip. Name
   the spell explicitly (`#showtooltip Shield Bash`) when the first `/cast` line is a stance or
   form change, or when the macro is mostly equip lines.
5. Use spell names exactly as they appear in the spellbook. Add a rank only on purpose, with no
   space before the parenthesis: `Frost Nova(Rank 1)`, `Healing Touch(Rank 5)`.
6. Use **`!`** for toggles that must not switch off when pressed again: `!Auto Shot`,
   `!Attack`, `!Shoot`, `!Stealth`, `!Prowl`, `!Cat Form`, `!Aspect of the Hawk`.
7. List conditional clauses from most specific to most general, and use a bare `[]` as the
   fallback (it means the current target). Separate clauses with `;`.

## 3. The user's conventions (copy these)

These come from the existing files in `classi_era/macros/`.

- **Name prefixes** (the user's own scheme). The prefix is a letter plus a space, for example
  `M Frostbolt`:

  | Prefix | Class |
  |---|---|
  | `+` | Every class (generic or utility macro, no space required: `+Trinket 1`) |
  | `D` | Druid |
  | `H` | Hunter |
  | `K` | Warrior (German *Krieger*) |
  | `M` | Mage |
  | `P` | Priest **and** Paladin (the user chose the same letter for both) |
  | `R` | Rogue |
  | `S` | Shaman |
  | `W` | Warlock |

  In `firemaw_hunter`, `E` is also used for enchanting and trade-chat adverts. Use the prefix in
  the account-wide file, where macros for every class sit side by side. In character files it is
  optional. Keep names at 16 characters or fewer.
- **Offensive spells** first fix the target, then cast:
  ```
  #showtooltip
  /targetenemy [noharm][dead]
  /cast Frostbolt
  ```
  The user's older macros use `/cleartarget [dead,help]` plus `/targetenemy [noharm]`. That
  pair stays on a dead enemy (see reference.md §Common bugs), so write new macros with the
  single line above. `/targetenemy` picks the nearest enemy. In raids, it can pick a mob that
  hasn't been pulled yet, so leave it out of raid macros if the user prefers.
  For melee, add `/startattack`. Rogues and Druids use `/startattack [nostealth]`.
- **Healing and buffs** cast on the mouseover target, then the current target, then yourself:
  `/cast [@mouseover,help,nodead][help][@player] Flash Heal`
- **Casts on an enemy under the mouse:**
  `/cast [@mouseover,harm,nodead][] Polymorph`
- **Interrupts and escapes** start with `/stopcasting`. Mages also add `/cancelaura Ice Block`.
- **Stance dancing (Warrior):** `/cast [form:2] Battle Stance` followed by `/cast Execute`. Press
  once to change stance and again to cast. For shield abilities, add `/equipslot 16 <MH>` and
  `/equipslot 17 <Shield>` before the cast.
- **Druid forms:** `/cancelform [noform:N]` followed by `/cast !<Form>`. Healing spells use a bare
  `/cancelform`.
- **Hunters:** before shots, add `/cast !Auto Shot` after the shot. Before CC or traps, use
  `/petpassive`, `/stopattack`, `/stopcasting`, then `/cast [combat] Feign Death`.
- **Gear sets:** `/run EquipSet("Name")`. This comes from an addon, so keep using it as written.
- **Item and slot use:** use slot numbers for trinkets and gear (`#showtooltip 13`, `/use 13`). To
  apply weapon poisons, use `/use <Poison>` then `/use 16` or `/use 17`.

**Intentional patterns. Don't flag these as bugs:**
- `/cancelaura Aspect of X` followed by `/cast Aspect of X` (`H Asp. Cheetah`, `H Asp. Pack`).
  The user has a reason for it.
- Wing Clip plus Raptor Strike on one button (`H Melee`). Wing Clip goes first, and Raptor Strike
  (off the GCD, "on next swing") adds damage. The mana cost is accepted.
- Macros listing talents from different trees, such as Cold Blood and Adrenaline Rush (`R AR/CB`).
  One macro serves every spec, and whichever talent is known gets cast.

When the user's macro already does something one way, keep that style unless it's broken. If
you see a bug, point it out. For example, `#sowtooltip` is a typo, `[dead][help]` (OR) and
`[dead,help]` (AND) mean different things, and the misspelling `Rejuvanation` is fine as a name but
not as a spell name.

## 4. Workflow

1. **Clarify** what the macro is for, which class it's for, which game (§1), and whether it's
   account-wide or for one character.
2. **Draft** it using the conventions above, with conditionals from reference.md.
3. **Check** every macro you write against this list:
   - It's 255 characters or fewer, and the name is 16 characters or fewer.
   - It has at most one GCD spell per press path.
   - Spell names and ranks are correct for the game version (Forever renamed some spells, for
     example Bane of Agony).
   - Conditionals exist in that game version.
   - `#showtooltip` shows the right icon.
4. **Explain** in one or two lines what each press does, including modifier keys.
5. **Deliver** it as a fenced code block ready to paste. If the user wants it in a file, also
   edit the file (§5).
6. Mark every behavior that is unconfirmed on Forever with a short "test in-game" note.

## 5. Editing `macros-cache.txt` files

Format (Windows CRLF line endings, one block per macro):

```
VER 3 0000000001000005 "M Arcane Missile" "INV_MISC_QUESTIONMARK"
#showtooltip Arcane Missiles
/cast [nochanneling:Arcane Missiles] Arcane Missiles
END
```

- **ID** is 16 hex digits and must be unique within the file. Account-wide macros use
  `00000000000000XX`. Character macros use `00000000010000XX`. New macros get the highest
  existing ID + 1.
- **Icon:** `"INV_MISC_QUESTIONMARK"` is the question-mark icon, which becomes dynamic with
  `#showtooltip`. A number is the icon file ID; you can find these on Wowhead icon pages.
- **Slot limits:** Classic Era has 120 account-wide and 18 per character. `zt_druid` is already
  full at 18, and `account1` holds 103. Retail now has 30 per character. Forever ⚠ hasn't
  published its limits; check in-game with
  `/run print(MAX_ACCOUNT_MACROS, MAX_CHARACTER_MACROS)`. Tell the user when a file is full.
- Repo file names: `accountN_macros-cache.txt` for account-wide macros and
  `<realm>_<class>_macros-cache.txt` for character macros. In the game install they are:
  - Account: `World of Warcraft\_classic_era_\WTF\Account\<ACCOUNT>\macros-cache.txt`
  - Character: `...\WTF\Account\<ACCOUNT>\<Realm>\<Character>\macros-cache.txt`
  - Forever uses its own install folder. Confirm the folder name on the user's machine; during
    the beta it was `_classic_beta_`.
- **Importing:** the game must be fully closed. Exit through Logout or Exit, not Alt+F4, because
  the client rewrites these files when it closes. Macros are also synced to Blizzard's servers,
  and the server copy can overwrite local edits on login. If edits revert, paste the macros in-game
  instead.
- Keep CRLF line endings when editing. Check the result with:

```bash
awk '/^VER/{n=$0;b="";next}/^END/{l=length(b); if(l>255) print "TOO LONG ("l"): "n; next}{sub(/\r$/,""); b=(b==""?$0:b"\n"$0)}' FILE
```

## 6. Things to refuse or warn about

- **Multiboxing (dual boxing) is allowed. Input broadcasting is banned.** Input broadcasting means
  software or hardware that sends one key press to several game clients. The rule is one key
  press, one action, in one client. Help with macros for the follower character (see
  [dualbox.md](dualbox.md)), but not with broadcasting tools or key-mirroring setups.
- **Unknown `/run` scripts** from the internet can be used to scam gold or items. Explain what
  any script the user pastes does before endorsing it.
- **Automation that skips the hardware-event rule** isn't possible inside the game, and doing it
  with outside software is a bannable offense.
