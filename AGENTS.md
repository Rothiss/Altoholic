# AGENTS.md

WoW 1.12 Vanilla addon (Altoholic, backport of the TBC version). No build system, tests, linter, or CI — the only verification is loading in a 1.12 client. `## Interface: 11200` in `Altoholic.toc`.

## Loading model
- `Altoholic.toc` is the entry point; order there matters. It lists XML/template files, **not** module `.lua` files.
- Each `Modules/<Name>.xml` loads its own `<Name>.lua` via `<Script file="<Name>.lua"/>` (line 2). Adding a module means a new `.lua` + `.xml` pair plus a toc entry; don't add the `.lua` to the toc.
- Core load order: Libs embedded in `Libs/` → `Locale/` → `AltoholicTemplates.xml` → `Core.lua` (defines the `Altoholic` object + default DB) → `Items/` → `Altoholic.lua` (menu, shared helpers) → `Altoholic.xml` (main frame) → `Modules/`.

## Language/API constraints (Vanilla)
- Code targets old Lua (uses `table.getn`, etc.). Avoid `#`-length on non-array tables, `goto`, and post-1.12 APIs.
- Vanilla lacks some TBC APIs; `UIDropDownMenu_CreateInfo` is shimmed in `Core.lua`. Verify any API against the 1.12 client before relying on it.

## Global namespace (critical)
- Addons share one global namespace and generic names collide with other addons — this repo already renamed all frames to `Alto*` after a conflict with the "Mail" addon. Prefix every new frame/global/function with `Alto`/`Altoholic` (`AltoholicFrame`, `AltoSummary`, `AltoMail`, ...).

## Localization
- Every UI string goes through AceLocale-2.2. A new key means adding `["key"] = ...` to **all 5** files in `Locale/` (`enUS` authoritative; `frFR`, `deDE`, `zhTW`, `zhCN` translate, missing keys silently fall back to English).
- Use LibBabble constants (`BI` inventory, `BZ` zones, `BF` factions, `BB` bosses) for loot/skill/reputation data names, never translated literals — see `Core.lua` tables and `Items/Loots.lua`.
- `Items/Suggestions.lua` early-returns for zhCN/zhTW; those locales read `Items/Suggestions_zhCN.lua` / `Suggestions_zhTW.lua` instead. Keep all three in sync.

## Versioning
- Bump the version in 3 places: `Altoholic.toc` `## Version`, `Core.lua` (`Altoholic.vars.version`), and top of `changelog.txt`.

## Slash commands
- `/alto search <words>` (max 2 words), `/alto show`, `hide`, `toggle`.