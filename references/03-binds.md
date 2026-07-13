# Reference 03 — Key/Mouse Binds

> Companion to `SKILL.md`. Read this for the exhaustive bind translation,
> every flag, mouse binds, submaps, global keybinds, switches, and per-device binds.

## The core transformation

```ini
# 0.54
bind[FLAGS] = MODS, KEY, DISPATCHER, PARAMS
```
```lua
-- 0.55
hl.bind("MODS + KEY", hl.dsp.<dispatcher>(<params>), { flag = true, ... })
```

- Modifiers and the key are now a single `"MODS + KEY"` string joined by ` + `.
- `hl.dsp.*` calls **return a dispatcher object** (a table describing the action);
  they do not run immediately. They are meant to be passed to `hl.bind()` or
  `hl.dispatch()`.
- Flags moved from the `[...]` suffix into an options table (3rd argument).

### Basic examples

```ini
# 0.54
bind   = SUPER, Q, exec, kitty
bind   = SUPER, C, killactive,
bind   = $mainMod, V, togglefloating,
bind   = , Print, exec, grim
```
```lua
-- 0.55
local mainMod = "SUPER"
hl.bind(mainMod .. " + Q", hl.dsp.exec_cmd("kitty"))
hl.bind(mainMod .. " + C", hl.dsp.window.close())
hl.bind(mainMod .. " + V", hl.dsp.window.float({ action = "toggle" }))
hl.bind("Print", hl.dsp.exec_cmd("grim"))
```

## Bind flag translation

| 0.54 flag letter | 0.55 options-table key | meaning |
| --- | --- | --- |
| `l` | `{ locked = true }` | works under input inhibitor (lockscreen) |
| `r` | `{ release = true }` | triggers on key release |
| `e` | `{ repeating = true }` | repeats while held |
| `n` | `{ non_consuming = true }` | event also passed to app |
| — | `{ auto_consuming = true }` **NEW** | passed to app if dispatcher fails |
| `m` | `{ mouse = true }` | mouse bind (see Mouse Binds) |
| `t` | `{ transparent = true }` | cannot be shadowed |
| `i` | `{ ignore_mods = true }` | ignore modifiers |
| — | `{ dont_inhibit = true }` **NEW** | bypass app's inhibit requests |
| — | `{ long_press = true }` **NEW** | triggers on long press |
| — | `{ click = true }` **NEW** | triggers on release if cursor stays inside `binds.drag_threshold` |
| — | `{ drag = true }` **NEW** | triggers on release if cursor moved outside threshold |
| — | `{ submap_universal = true }` **NEW** | active in every submap |
| — | `{ description = "..." }` **NEW** | human-readable description (`hyprctl binds`) |
| — | `{ device = { inclusive = true, list = {...} } }` **NEW** | per-device (see below) |

`click` and `drag` are mutually exclusive. `long_press`/`release` are incompatible
with `repeating`. `mouse` is incompatible with `repeat`/`release`/`locked`.

### Old composite flags → new table

```ini
# 0.54
bindrl = SUPER, T, exec, kitty        # release + locked
binde  = , XF86AudioRaiseVolume, exec, wpctl set-volume ...
bindel = , XF86AudioRaiseVolume, exec, wpctl set-volume ...
bindm  = SUPER, mouse:272, movewindow
```
```lua
-- 0.55
hl.bind("SUPER + T", hl.dsp.exec_cmd("kitty"), { release = true, locked = true })
hl.bind("XF86AudioRaiseVolume", hl.dsp.exec_cmd("wpctl set-volume @DEFAULT_AUDIO_SINK@ 5%+"), { repeating = true })
hl.bind("XF86AudioRaiseVolume", hl.dsp.exec_cmd("wpctl set-volume @DEFAULT_AUDIO_SINK@ 5%+"), { repeating = true, locked = true })
hl.bind("SUPER + mouse:272", hl.dsp.window.drag(), { mouse = true })
```

## Special key forms (unchanged concepts, new quoting)

| Concept | 0.54 | 0.55 |
| --- | --- | --- |
| keycode | `bind=SUPER,code:28,exec,...` | `hl.bind("SUPER + code:28", ...)` |
| mouse button | `bind=SUPER,mouse:272,exec,...` | `hl.bind("SUPER + mouse:272", ...)` |
| mouse wheel | `mouse_down`/`mouse_up`/`mouse_left`/`mouse_right` | `"SUPER + mouse_down"` |
| switches | `switch:[name]`, `switch:on:[name]`, `switch:off:[name]` | same strings, just quoted |
| bind modkey alone | `bindr=ALT,Alt_L,...` | `hl.bind("ALT + ALT_L", ..., { release = true })` |
| catch-all (submaps) | `bind=,catchall,...` | `hl.bind("catchall", ...)` (submaps only) |

Keysym names: the segment after `XKB_KEY_` in `xkbcommon-keysyms.h`. Use `wev` to
discover codes. Note: `Enter` is **not** valid — use `Return`.

## Mouse binds

Old `bindm` with `movewindow`/`resizewindow` becomes a `{ mouse = true }` flag
with `hl.dsp.window.drag()` / `hl.dsp.window.resize()`.

```ini
# 0.54
bindm = SUPER, mouse:272, movewindow
bindm = SUPER, mouse:273, resizewindow
bindm = SUPER, mouse:273, resizewindow, 1   # keep aspect ratio
```
```lua
-- 0.55
hl.bind("SUPER + mouse:272", hl.dsp.window.drag(),   { mouse = true })
hl.bind("SUPER + mouse:273", hl.dsp.window.resize(), { mouse = true })
hl.bind("SUPER + mouse:273", hl.dsp.window.resize({ keep_aspect_ratio = true }), { mouse = true })
```

Common mouse codes: LMB `272`, RMB `273`, MMB `274`.

### Click vs drag differentiation (NEW)

Use `binds.drag_threshold` + `click`/`drag` flags to bind different actions to
click vs drag of the same button:
```lua
hl.config({ binds = { drag_threshold = 10 } })
hl.bind("ALT + mouse:272", hl.dsp.window.drag(),  { mouse = true, drag = true })
hl.bind("ALT + mouse:272", hl.dsp.window.float(), { mouse = true, click = true })
```

### Touchpad-friendly: use a keyboard key instead of a mouse button
```lua
hl.bind("SUPER + CTRL_L", hl.dsp.window.drag(),   { mouse = true })
hl.bind("SUPER + ALT_L",  hl.dsp.window.resize(), { mouse = true })
```

## Multiple actions on one key

Old: list multiple `bind=` lines with the same key. New: use a Lua function and
call `hl.dispatch()` for each (they run top-to-bottom):
```lua
hl.bind("SUPER + Tab", function()
    hl.dispatch(hl.dsp.window.cycle_next())
    hl.dispatch(hl.dsp.window.bring_to_top())
end)
```
(You may also still bind the same key string to multiple `hl.bind` calls.)

## Lua function as a dispatcher

Any bind can take a Lua function instead of an `hl.dsp.*` object — essential for
conditional logic:
```lua
hl.bind("SUPER + X", function()
    local w = hl.get_active_window()
    if w ~= nil and w.title == "htop" then
        hl.dispatch(hl.dsp.window.float({ action = "set" }))
    else
        hl.dispatch(hl.dsp.window.float({ action = "toggle" }))
    end
end)
```

## Unbind

```ini
# 0.54
unbind = SUPER, O
```
```lua
-- 0.55
hl.unbind("SUPER + O")        -- must EXACTLY match case of the bind
hl.unbind("all")              -- clear everything (NEW)
```
Case matters: `"SUPER + TAB"` unbinds a bind created with `"SUPER + TAB"`, but
will NOT unbind `"SUPER + Tab"`.

## Bind handles (NEW)

`hl.bind()` returns a handle you can disable/enable/remove at runtime:
```lua
local b = hl.bind("SUPER + C", hl.dsp.window.close())
b:set_enabled(false)   -- disable
b:set_enabled(true)    -- re-enable
b:is_enabled()
b:unbind()
```

## Submaps (modes)

Old: `submap=name` ... binds ... `submap=reset`. New: `hl.define_submap(name, fn)`
runs the binds inside `fn` under that submap.

```ini
# 0.54
bind  = ALT, R, submap, resize
submap = resize
binde = , right, resizeactive, 10 0
binde = , left,  resizeactive, -10 0
bind  = , escape, submap, reset
submap = reset
```
```lua
-- 0.55
hl.bind("ALT + R", hl.dsp.submap("resize"))

hl.define_submap("resize", function()
    hl.bind("right",  hl.dsp.window.resize({ x = 10,  y = 0, relative = true }), { repeating = true })
    hl.bind("left",   hl.dsp.window.resize({ x = -10, y = 0, relative = true }), { repeating = true })
    hl.bind("escape", hl.dsp.submap("reset"))
end)
```

Notes:
- Always provide an escape bind; if stuck, `hyprctl dispatch 'hl.dsp.submap("reset")'`.
- Multiple actions per key inside a submap: same Lua-function trick.
- **Nesting** is supported (define a submap that enters another).
- **Auto-close on dispatch**: `hl.define_submap("A", "B", function() ... end)` —
  the 2nd arg (`"B"`) is the submap to switch to after any bind in `A` fires;
  use `"reset"` to return to global.
- **Catch-all** in a submap: `hl.bind("catchall", ...)`.

## Global keybinds

### Classic (`pass` / `send_shortcut`)

```ini
# 0.54 — pass SUPER+F10 to OBS
bind = SUPER, F10, pass, ^(com\.obsproject\.Studio)$
```
```lua
-- 0.55
hl.bind("SUPER + F10", hl.dsp.pass({ window = "class:^(com\\.obsproject\\.Studio)$" }))
```
`pass` handles press+release itself (no `release` flag needed).

Send a *different* shortcut to a window:
```ini
# 0.54
bind = SUPER, F10, sendshortcut, SUPER, F4, ^(TeamSpeak 3)$
```
```lua
-- 0.55
hl.bind("SUPER + F10", hl.dsp.send_shortcut({ mods = "SUPER", key = "F4", window = "class:^(TeamSpeak 3)$" }))
```
(New `send_key_state` lets you control down/up/repeat explicitly.)

### DBus Global Shortcuts

```ini
# 0.54
bind = SUPERSHIFT, A, global, coolApp:myToggle
```
```lua
-- 0.55
hl.bind("SUPER + SHIFT + A", hl.dsp.global("coolApp:myToggle"))
```
(Works only with xdg-desktop-portal-hyprland. Discover with `hyprctl globalshortcuts`.)

## Per-device binds (NEW)

Bind a key only on specific devices (names from `hyprctl devices`, or device tags):
```lua
-- Only these keyboards can trigger it
hl.bind("SUPER + Q", hl.dsp.exec_cmd("kitty"),
        { device = { inclusive = true,  list = { "kb-1", "kb-2" } } })

-- All keyboards EXCEPT these
hl.bind("SUPER + Q", hl.dsp.exec_cmd("kitty"),
        { device = { inclusive = false, list = { "razer-kb" } } })
```

## Switchable keyboard layouts

Unchanged concept; config just moves into a table:
```ini
# 0.54
input { kb_layout = us,cz ; kb_variant = ,qwerty ; kb_options = grp:alt_shift_toggle }
```
```lua
-- 0.55
hl.config({
    input = {
        kb_layout   = "us,cz",
        kb_variant  = ",qwerty",
        kb_options  = "grp:alt_shift_toggle",
    },
})
```
Switch at runtime: `hyprctl switchxkblayout [DEVICE] next|prev|ID`. The first
layout defines bind keymap unless `input.resolve_binds_by_sym = true`.

## Non-QWERTY layout binds

Keys used in binds must be reachable without modifiers in your layout. On AZERTY
the digit row needs SHIFT, so bind the unmodified key name instead:
```lua
hl.bind(mainMod .. " + ampersand", hl.dsp.focus({ workspace = 1 }))   -- was "... + 1"
```

## Description (NEW, for `hyprctl binds` / GUIs)
```lua
hl.bind("SUPER + Q", hl.dsp.exec_cmd("kitty"), { description = "Open terminal" })
```

## Bind-options reference table (all keys)

| key | type | meaning |
| --- | --- | --- |
| `locked` | bool | active under lockscreen |
| `release` | bool | on key release |
| `repeating` | bool | repeat while held |
| `non_consuming` | bool | event also reaches app |
| `auto_consuming` | bool | reaches app if dispatcher fails |
| `mouse` | bool | is a mouse bind |
| `transparent` | bool | cannot be shadowed |
| `ignore_mods` | bool | ignore modifiers |
| `dont_inhibit` | bool | bypass app keybind-inhibit |
| `long_press` | bool | on long press |
| `click` | bool | release inside drag_threshold |
| `drag` | bool | release outside drag_threshold |
| `submap_universal` | bool | active in every submap |
| `description` / `desc` | string | human label |
| `device` | table `{inclusive?, list?}` | per-device scoping |
