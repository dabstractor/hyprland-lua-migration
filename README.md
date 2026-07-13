# Hyprland 0.55 Lua Migration Skill

A comprehensive, source-verified upgrade guide for migrating Hyprland
configurations from the **v0.54 "hyprlang"** syntax (`hyprland.conf`) to the
**v0.55 Lua** API (`hyprland.lua`). Hyprland introduced this breaking change
with no official upgrade guide — this skill is that guide.

This is an **Agent Skill** (a `SKILL.md` plus `references/`). It is designed to
be loaded by Pi and other Agent Skills-compatible clients so an agent can
accurately convert, explain, and troubleshoot Hyprland configs.

## What's inside

- `SKILL.md` — the guided migration walkthrough: TL;DR table, step-by-step
  procedure, a fully-translated example config, common traps, and a tour of new
  capabilities.
- `references/` — eight exhaustive reference docs covering every option, bind,
  dispatcher, rule, monitor field, animation, gesture, event, and object — each
  old construct shown beside its new Lua equivalent.
  1. `01-core-syntax.md` — file mechanics, require, env, exec, sections, types
  2. `02-variables.md` — every `hl.config` option with old→new/removed markers
  3. `03-binds.md` — bind/flag translation, mouse binds, submaps, global keybinds
  4. `04-dispatchers.md` — every old dispatcher → `hl.dsp.*` with params
  5. `05-window-layer-rules.md` — all window/layer rule props & effects
  6. `06-monitors-workspaces.md` — monitor fields (HDR/ICC/VRR) & workspace rules
  7. `07-animations-gestures.md` — curves, animation tree, the new gesture system
  8. `08-events-objects-custom-layouts.md` — events, query API, timers, custom layouts

## Installation (Pi)

```sh
pi install <this directory or git URL>
```

Or drop the `hyprland-0.55-lua-migration/` directory into one of Pi's skill
discovery paths (`~/.pi/agent/skills/`, `~/.agents/skills/`, `.pi/skills/`, …).

## Trigger

The skill auto-invokes whenever a task involves converting a Hyprland config,
explaining the 0.54→0.55 changes, or debugging a config broken by the upgrade.

## Sources

Built by diffing Hyprland 0.55 (`example/hyprland.lua`, `src/config/lua/**`,
`meta/generateLuaStubs.py`) against 0.54 (`example/hyprland.conf`), and the new
wiki against the pre-lua wiki. Every parameter table is mirrored from source.
