---
name: hyprland-lua-migration
description: >
  Migrate and rewrite Hyprland configs from the old v0.54 hyprlang (.conf) syntax
  to the new v0.55 Lua (hyprland.lua) API. Use when converting an existing
  hyprland.conf, writing new Hyprland Lua config, explaining what changed between
  0.54 and 0.55, or troubleshooting a config that broke after upgrading. Covers
  every keyword, variable, bind, dispatcher, window/layer/workspace rule, monitor,
  animation, curve, and gesture — old format shown beside the new Lua equivalent.
---

# Hyprland 0.54 → 0.55 Migration Guide (hyprlang → Lua)

Hyprland **0.55** replaced its bespoke "hyprlang" config language with **Lua**.
The config file is now a Lua script that calls functions on a global `hl` table.
There is **no official upgrade guide** — this skill is that guide.

When you are asked to migrate, convert, explain, or fix a Hyprland config, follow
this document. Every old construct has a direct new equivalent shown side by
side. Exhaustive detail (every option, every param) lives in `references/` — read
the relevant reference whenever you need the complete list for a subsystem.

## Operating procedure

This skill is not just reference — it is a **procedure**. Follow it every time.

### 1. Survey the whole config, then get approval before touching anything

Unless the user gave a narrow, fully-specified task ("change this one bind to
X"), treat **all of `~/.config/hypr/`** as in scope: the main `hyprland.lua` or
`hyprland.conf`, **plus** every file it `require`s or sources. Read each file end
to end and catalog **every** upgrade opportunity — old-syntax leftovers,
renamed/moved/inverted/removed options (Reference 02), and the traps listed
below.

Then **stop and report — do not edit yet.** Hand back a reviewable plan grouped
by file: for each change show the old form, the new form, and why (cite the
specific trap or reference row). Begin converting only after the user approves.
This survey-and-confirm loop is the default; skip it only when the instruction
is already an explicit "go change X to Y".

### 2. Verify continuously while you work

Every time you change a file, reload and surface errors before moving on:

```sh
hyprctl reload && hyprctl configerrors
```

`configerrors` prints every config warning/error Hyprland found on reload. Fix
its reports before doing anything else. Cross-check behavior by running commands
live in the CLI — `hyprctl repl`, `hyprctl eval 'LUA'`, or `hyprctl dispatch
'LUA'` (see Reference 01 §12). Never leave a file producing errors you haven't
reported to the user.

## TL;DR — the shape of the change

| | 0.54 (hyprlang) | 0.55 (Lua) |
| --- | --- | --- |
| File | `~/.config/hypr/hyprland.conf` | `~/.config/hypr/hyprland.lua` |
| Comment | `# ...` | `-- ...` |
| Variable | `$term = kitty` | `local term = "kitty"` |
| Source | `source = file.conf` | `require("file")` |
| Env | `env = NAME,VALUE` | `hl.env("NAME", "VALUE")` |
| Autostart | `exec-once = cmd` | `hl.on("hyprland.start", function() hl.exec_cmd("cmd") end)` |
| Section | `general { x = 1 }` | `hl.config({ general = { x = 1 } })` |
| Monitor | `monitor=name,WxH@RR,XxY,scale` | `hl.monitor({ output=..., mode=..., position=..., scale=... })` |
| Bind | `bind[fl]=MOD,KEY,disp,params` | `hl.bind("MOD + KEY", hl.dsp.<disp>(<params>), { flag=true })` |
| Window rule | `windowrulev2=r,class:re` | `hl.window_rule({ match={class="re"}, <effects> })` |
| Layer rule | `layerrule = blur, ns` | `hl.layer_rule({ match={namespace="ns"}, blur=true })` |
| Workspace rule | `workspace=N, gapsin:0` | `hl.workspace_rule({ workspace="N", gaps_in=0 })` |
| Bezier | `bezier = n, x,y,x,y` | `hl.curve("n", {type="bezier", points={{x,y},{x,y}}})` |
| Animation | `animation = leaf, 1, 8, c, style` | `hl.animation({ leaf=..., enabled=true, speed=8, bezier="c", style=... })` |
| Bool values | `true/false/yes/no/on/off/0/1` | Lua `true` / `false` **only** |

## Step-by-step migration procedure

1. **Rename** `hyprland.conf` → `hyprland.lua` (or start fresh; keep the old file open for reference).
2. **Convert comments** `#` → `--`.
3. **Convert variables** `$X = v` → `local X = "v"`; concatenation uses `..`.
4. **Convert `source`** → `require("name")` (one protected scope per file — split liberally).
5. **Convert `env`/`envd`** → `hl.env("NAME","VALUE")` (3rd arg `true` = dbus).
6. **Convert `exec`/`exec-once`** → top-level `hl.exec_cmd()` (every reload) or the `hyprland.start` event (once).
7. **Convert sections** (`general{}`, `decoration{}`, `input{}`, …) → `hl.config({ section = { ... } })`. Nested blocks become nested tables.
8. **Convert `monitor=`** → `hl.monitor({...})` (named fields).
9. **Convert `device:name{}`** → `hl.device({ name="...", ... })`.
10. **Convert binds** → `hl.bind("MOD + KEY", hl.dsp.*(...), { flags })`.
11. **Convert window/layer/workspace rules** → `hl.window_rule` / `hl.layer_rule` / `hl.workspace_rule`.
12. **Convert bezier/animation** → `hl.curve` / `hl.animation`.
13. **Convert gestures**: remove `workspace_swipe*` vars; add `hl.gesture({...})`.
14. **Convert permissions** → `hl.permission({...})` + `ecosystem.enforce_permissions`.
15. **Fix type literals**: `yes/no/on/off/1/0` → `true/false`; quote strings; vec2 `a b` → `{a,b}`; gradient `c c 45deg` → `{colors={...}, angle=45}`.
16. **Drop removed options** (see Reference 02 "REMOVED" rows) and **relocate moved ones** (`general.no_cursor_warps`→`cursor.no_warps`, `misc.vfr`→`debug.vfr`, `misc.no_direct_scanout`→`render.direct_scanout` inverted).

Run `hyprctl reload && hyprctl configerrors` and fix everything it reports before moving on — see **Operating procedure** for the full loop.

## The example config, fully translated

This is the official `example/hyprland.conf` rewritten as `hyprland.lua`. Use it
as a template and as proof that every concept maps over.

```lua
-- hyprland.lua
local terminal    = "kitty"
local fileManager = "dolphin"
local menu        = "hyprlauncher"
local mainMod     = "SUPER"

------------------ MONITORS ------------------
hl.monitor({ output = "", mode = "preferred", position = "auto", scale = "auto" })

------------------ ENVIRONMENT ----------------
hl.env("XCURSOR_SIZE", "24")
hl.env("HYPRCURSOR_SIZE", "24")

------------------ AUTOSTART (was exec-once) --
hl.on("hyprland.start", function()
    -- hl.exec_cmd(terminal); hl.exec_cmd("nm-applet"); hl.exec_cmd("waybar & hyprpaper")
end)

------------------ LOOK & FEEL ----------------
hl.config({
    general = {
        gaps_in = 5, gaps_out = 20, border_size = 2,
        col = {
            active_border   = { colors = {"rgba(33ccffee)", "rgba(00ff99ee)"}, angle = 45 },
            inactive_border = "rgba(595959aa)",
        },
        resize_on_border = false, allow_tearing = false, layout = "dwindle",
    },
    decoration = {
        rounding = 10, rounding_power = 2,
        active_opacity = 1.0, inactive_opacity = 1.0,
        shadow = { enabled = true, range = 4, render_power = 3, color = 0xee1a1a1a },
        blur   = { enabled = true, size = 3, passes = 1, vibrancy = 0.1696 },
    },
    animations = { enabled = true },
})

-- Curves & animations (were inside animations{} block)
hl.curve("easeOutQuint",   { type = "bezier", points = { {0.23, 1},    {0.32, 1} } })
hl.curve("easeInOutCubic", { type = "bezier", points = { {0.65, 0.05}, {0.36, 1} } })
hl.curve("linear",         { type = "bezier", points = { {0, 0},       {1, 1} } })
hl.curve("almostLinear",   { type = "bezier", points = { {0.5, 0.5},   {0.75, 1} } })
hl.curve("quick",          { type = "bezier", points = { {0.15, 0},    {0.1, 1} } })
hl.curve("easy",           { type = "spring", mass = 1, stiffness = 71.2633, dampening = 15.8273644 })

hl.animation({ leaf = "windows", enabled = true, speed = 4.79, spring = "easy" })
hl.animation({ leaf = "windowsIn", enabled = true, speed = 4.1, spring = "easy", style = "popin 87%" })
hl.animation({ leaf = "workspaces", enabled = true, speed = 1.94, bezier = "almostLinear", style = "fade" })
-- ... (full set in Reference 07)

hl.config({ dwindle = { preserve_split = true } })
hl.config({ master   = { new_status = "master" } })
hl.config({ scrolling= { fullscreen_on_one_column = true } })
hl.config({ misc = { force_default_wallpaper = -1, disable_hyprland_logo = false } })

------------------ INPUT ----------------------
hl.config({
    input = {
        kb_layout = "us", kb_variant = "", kb_model = "", kb_options = "", kb_rules = "",
        follow_mouse = 1, sensitivity = 0,
        touchpad = { natural_scroll = false },
    },
})

-- Was: gesture = 3, horizontal, workspace
hl.gesture({ fingers = 3, direction = "horizontal", action = "workspace" })

-- Was: device { name = epic-mouse-v1; sensitivity = -0.5 }
hl.device({ name = "epic-mouse-v1", sensitivity = -0.5 })

------------------ KEYBINDINGS -----------------
hl.bind(mainMod .. " + Q", hl.dsp.exec_cmd(terminal))
hl.bind(mainMod .. " + C", hl.dsp.window.close())   -- was killactive
hl.bind(mainMod .. " + E", hl.dsp.exec_cmd(fileManager))
hl.bind(mainMod .. " + V", hl.dsp.window.float({ action = "toggle" }))  -- was togglefloating
hl.bind(mainMod .. " + R", hl.dsp.exec_cmd(menu))
hl.bind(mainMod .. " + P", hl.dsp.window.pseudo())  -- was pseudo
hl.bind(mainMod .. " + J", hl.dsp.layout("togglesplit"))  -- was layoutmsg

-- movefocus (old: movefocus, l/r/u/d)
hl.bind(mainMod .. " + left",  hl.dsp.focus({ direction = "left" }))
hl.bind(mainMod .. " + right", hl.dsp.focus({ direction = "right" }))
hl.bind(mainMod .. " + up",    hl.dsp.focus({ direction = "up" }))
hl.bind(mainMod .. " + down",  hl.dsp.focus({ direction = "down" }))

-- workspace / movetoworkspace (loops show the power of Lua)
for i = 1, 10 do
    local key = i % 10
    hl.bind(mainMod .. " + " .. key,         hl.dsp.focus({ workspace = i }))
    hl.bind(mainMod .. " + SHIFT + " .. key, hl.dsp.window.move({ workspace = i }))
end

-- special workspace
hl.bind(mainMod .. " + S",         hl.dsp.workspace.toggle_special("magic"))
hl.bind(mainMod .. " + SHIFT + S", hl.dsp.window.move({ workspace = "special:magic" }))

-- scroll workspaces
hl.bind(mainMod .. " + mouse_down", hl.dsp.focus({ workspace = "e+1" }))
hl.bind(mainMod .. " + mouse_up",   hl.dsp.focus({ workspace = "e-1" }))

-- mouse move/resize (was bindm ... movewindow/resizewindow)
hl.bind(mainMod .. " + mouse:272", hl.dsp.window.drag(),   { mouse = true })
hl.bind(mainMod .. " + mouse:273", hl.dsp.window.resize(), { mouse = true })

-- media keys (old bindel/bindl -> flags table)
hl.bind("XF86AudioRaiseVolume", hl.dsp.exec_cmd("wpctl set-volume -l 1 @DEFAULT_AUDIO_SINK@ 5%+"), { repeating = true, locked = true })
hl.bind("XF86AudioMute",        hl.dsp.exec_cmd("wpctl set-mute @DEFAULT_AUDIO_SINK@ toggle"),     { locked = true })
hl.bind("XF86AudioPlay",        hl.dsp.exec_cmd("playerctl play-pause"), { locked = true })

------------------ WINDOW RULES ----------------
hl.window_rule({
    name = "suppress-maximize-events",
    match = { class = ".*" },          -- was match:class = .*
    suppress_event = "maximize",
})
hl.window_rule({
    name  = "fix-xwayland-drags",
    match = { class = "^$", title = "^$", xwayland = true, float = true, fullscreen = false, pin = false },
    no_focus = true,
})
hl.window_rule({
    name  = "move-hyprland-run",
    match = { class = "hyprland-run" },
    move  = "20 monitor_h-120",   -- string form still accepted for expressions
    float = true,
})
```

## Common migration traps (high-priority)

- **`killactive` ≠ kill.** Old `killactive` *gracefully closed* the window. The
  new equivalent is `hl.dsp.window.close()`. There is now a real
  `hl.dsp.window.kill()` that SIGKILLs — don't swap them by accident.
- **`movetoworkspacesilent`** is now `hl.dsp.window.move({ workspace = W, follow = false })`.
  Plain `movetoworkspace` omits `follow` (defaults to following).
- **Booleans must be Lua booleans.** `yes/on/1` no longer work — use `true`.
- **Strings need quotes.** `layout = dwindle` → `layout = "dwindle"`.
- **Gradients** `rgba(a) rgba(b) 45deg` → `{ colors = {"rgba(a)","rgba(b)"}, angle = 45 }`. A single color can stay a string.
- **vec2** `10 -10` → `{ 10, -10 }`.
- **`$VAR` interpolation inside tokens** (`ff$R1111`) doesn't exist; build the whole string in Lua.
- **`source`** is `require` — and each `require`d file is isolated, so split your config.
- **`exec`/`exec-once`** are gone. Top-level `hl.exec_cmd` runs every reload; the `hyprland.start` event runs once.
- **`fakefullscreen`** is gone as a 1:1 concept; use `fullscreen_state({ internal = 2, client = 0 })`.
- **Gestures**: delete `workspace_swipe`, `workspace_swipe_fingers`, `workspace_swipe_numbered`, `workspace_swipe_min_fingers` — add `hl.gesture({ fingers=3, direction="horizontal", action="workspace" })`.
- **`misc.vfr`** moved to **`debug.vfr`**. **`misc.no_direct_scanout`** became **`render.direct_scanout`** and is **inverted** (and now 0/1/2). **`general.no_cursor_warps`** → **`cursor.no_warps`**; **`general.cursor_inactive_timeout`** → **`cursor.inactive_timeout`**.
- **`dwindle.pseudotile`** and **`master.gaps_in/out`** config options were removed (pseudotile is a dispatcher/window rule; gaps live in `general`/workspace rules).
- **`binds:focus_preferred_method`** affects `hl.dsp.focus({direction})` / `hl.dsp.window.move({direction})`.
- **uwsm users**: don't bind `hl.dsp.exit()`; use `hl.dsp.exec_cmd("uwsm stop")`.

## New capabilities with no 0.54 equivalent (mention proactively)

- **Real logic in config**: conditionals, loops (the `for i=1,10` workspace binds), functions.
- **`hl.on` events** (`window.active`, `workspace.created`, `config.reloaded`, …) — replaces most IPC scripting.
- **`hl.get_*` query API** — inspect windows/workspaces/monitors live and react.
- **`hl.timer`** — repeat/oneshot timers in config.
- **`hl.notification.create`** — script the built-in notifications.
- **Custom Lua layouts** via `hl.layout.register("name", { recalculate, layout_msg })`, used as `lua:name`.
- **Live gestures** with `start`/`update`/`finish` callbacks; pinch/zoom; modifier gating.
- **Runtime rule toggling** — named rules return handles with `:set_enabled(bool)`.
- **Per-device keybinds**, click/drag differentiation, `long_press`, `submap_universal`, `auto_consuming`.
- **LSP autocompletions** (`/usr/share/hypr/stubs`) and **`hyprctl repl`** for live exploration.

## References (read the one matching the task)

- **`references/01-core-syntax.md`** — file mechanics, comments, variables, require, env, autostart/exec, sections, type system, error behavior, LSP, REPL. *Read when:* translating top-level structure or a whole file.
- **`references/02-variables.md`** — every `hl.config` option (general/decoration/input/gestures/group/misc/cursor/render/…), old→new names, moved/renamed/inverted/removed markers. *Read when:* converting any section or diagnosing an unknown-key error.
- **`references/03-binds.md`** — bind/flag translation, mouse binds, submaps, global keybinds, switches, per-device binds. *Read when:* converting keybindings.
- **`references/04-dispatchers.md`** — every old dispatcher → `hl.dsp.*` method with params; workspace/window/monitor selectors; fullscreen_state; exec-with-rules. *Read when:* converting binds or dispatcher calls.
- **`references/05-window-layer-rules.md`** — every window-rule prop/effect and layer-rule, old→new; tags; named-rule handles. *Read when:* converting rules.
- **`references/06-monitors-workspaces.md`** — all monitor fields (incl. HDR/ICC/VRR) and workspace-rule fields/selectors. *Read when:* converting monitor/workspace lines.
- **`references/07-animations-gestures.md`** — bezier/spring curves, `hl.animation`, full animation tree+styles, the new `hl.gesture` system. *Read when:* converting animations or gestures.
- **`references/08-events-objects-custom-layouts.md`** — `hl.on` events, `hl.get_*` objects (complete field tables), timers, notifications, permissions, custom Lua layouts, plugins, hyprctl. *Read when:* adding new logic or explaining new features.
- **`references/09-layouts.md`** — full config tables + layout-message tables for Dwindle, Master, Scrolling, Monocle, and custom layouts. *Read when:* converting `dwindle{}`/`master{}`/scrolling blocks or `layoutmsg` binds.

## Scope & non-goals

This guide covers **everything that changed structurally in config syntax**
(0.54 hyprlang → 0.55 Lua): every keyword, section, variable, bind, dispatcher,
rule, monitor field, animation, curve, gesture, event, and object.

It intentionally does **not** re-document environment/hardware setup pages that
are unaffected by the syntax change (those still use `hl.env(...)` and Aquamarine
vars exactly as before): Multi-GPU device selection (`AQ_DRM_DEVICES`),
Virtual-GPU/headless setup (`hyprctl output`), and the Performance tuning page.
Their content is unchanged in form — only wrap any env var in `hl.env(NAME, VAL)`.

## Authoritative provenance

This guide is **fully self-contained** — every option, param, default, and
old→new mapping is reproduced inline in the references below. It was derived and
verified by diffing: Hyprland `example/hyprland.lua` (0.55) vs
`example/hyprland.conf` (0.54); the 0.55 C++ source (`src/config/lua/**`, the
rule/match engines in `src/desktop/rule/**`, and the config-value defaults in
`src/config/values/ConfigValues.cpp`); and the autogenerated LuaLS stubs
(`meta/generateLuaStubs.py`). When two sources disagree, the parameter tables
here follow the source code.
