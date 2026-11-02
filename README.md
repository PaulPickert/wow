# wow

Personal World of Warcraft account management: macros, settings, and notes for
**Classic Era** and **WoW Forever** (Classic+, launched 2026-11-04).

## Layout

| Path | Content |
|---|---|
| `classi_era/macros/` | Exported `macros-cache.txt` files: `accountN_*` hold account-wide macros, `<realm>_<class>_*` hold character macros |
| `forever/` | WoW Forever data (macros etc.) |
| `.claude/skills/wow-macros/` | Claude Code skill for writing and reviewing macros for both games |

## Tasks

- [x] Create a macro skill for Classic Era and WoW Forever (`.claude/skills/wow-macros`)
- [ ] **Research APIs for WoW Forever item databases.** Goal: look up items (stats, IDs, icons,
      sources) programmatically, for macros, gear planning, and other account tooling.
      Leads to check:
  - Blizzard Game Data API (develop.battle.net): check whether a Forever namespace exists
    (Classic Era uses `static-classic1x-<region>`). Note that the Classic item endpoints have
    historically left out stats and tooltip data.
  - Wowhead: check whether it has a Forever database or tooltip data (there's no official
    public API; older tools used the tooltip/XML feed).
  - wago.tools: DB2 client tables extracted from the Forever build (Item, ItemSparse, …), which
    can be exported as CSV.
  - Community data used by Forever addons: QuestieDB (Forever bundle), and the item catalog of
    the "Forever Guide" addon, which is built from wago.tools plus a Classic fallback.
  - npm `wow-classic-items`: check whether it gets a Forever data set.
  - Decide whether to call a live API or keep a local snapshot (CSV/JSON) in this repo.
