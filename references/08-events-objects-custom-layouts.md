# Reference 08 — Events, Objects, Timers, Notifications, Custom Layouts

> Companion to `SKILL.md`. Read this for the parts of the new Lua API that had
> **no equivalent in 0.54** (or only crude equivalents via IPC). These are the
> features that make the Lua config dramatically more powerful than hyprlang.

## Events (`hl.on`)

Register callbacks on compositor events. Any number of listeners per event.

```lua
hl.on("window.active", function(w)
    hl.notification.create({ text = "Focused: " .. w.title, timeout = 3000, icon = "ok" })
end)

hl.on("workspace.move_to_monitor", function(ws, m)
    print(ws.name, "moved to monitor at", m.position.x)
end)
```

### Subscription handle
`hl.on` returns an object with `:is_active()` and `:remove()`.

### Full event list

| event | parameters |
| --- | --- |
| `hyprland.start` | (none) — fires once on start |
| `hyprland.shutdown` | (none) — fires once before exit |
| `window.open` | Window (fully initialized, rules applied) |
| `window.open_early` | Window (mapped, **before** rules) |
| `window.close` | Window (may still be visible during close anim) |
| `window.destroy` | Window (after close anim completes) |
| `window.kill` | Window (force-killed via hyprctl kill) |
| `window.active` | Window, int (focus reason 0/1) |
| `window.urgent` | Window |
| `window.title` | Window |
| `window.class` | Window |
| `window.pin` | Window |
| `window.fullscreen` | Window |
| `window.update_rules` | Window (rules re-evaluated) |
| `window.move_to_workspace` | Window, Workspace |
| `layer.opened` | LayerSurface |
| `layer.closed` | LayerSurface |
| `monitor.added` | Monitor |
| `monitor.removed` | Monitor |
| `monitor.focused` | Monitor |
| `monitor.layout_changed` | (none) |
| `workspace.active` | Workspace |
| `workspace.special_active` | Workspace, Monitor (workspace nil = none open) |
| `workspace.created` | Workspace |
| `workspace.removed` | Workspace |
| `workspace.move_to_monitor` | Workspace, Monitor |
| `config.reloaded` | (none) — fired after reload applied |
| `config.props_refreshed` | bool (was it a scheduled refresh) |
| `keybinds.submap` | string (submap name; "" = default) |
| `screenshare.state` | bool active, int type, string name |
| `input.keyboard.key` | int keycode, int timestamp, int state (0 rel/1 press/2 repeat) |

Unknown event names error and list the known events.

## Object / query API (`hl.get_*`)

These return rich Lua objects (set up the LSP stubs to see fields). All selectors
accept a string/number/object; pass nothing where a "current" variant exists.

| function | returns |
| --- | --- |
| `hl.get_active_window()` | Window\|nil |
| `hl.get_active_workspace(monitor?)` | Workspace\|nil |
| `hl.get_active_special_workspace(monitor?)` | Workspace\|nil |
| `hl.get_active_monitor()` | Monitor |
| `hl.get_monitor_at_cursor()` | Monitor\|nil |
| `hl.get_monitor_at({x=,y=})` / `get_monitor_at(x,y)` | Monitor\|nil |
| `hl.get_cursor_pos()` | `{x=,y=}` |
| `hl.get_current_submap()` | string |
| `hl.get_window(selector)` | Window\|nil |
| `hl.get_windows(filters?)` | Window[] (`filters` = a WindowQueryFilter table) |
| `hl.get_urgent_window()` | Window\|nil |
| `hl.get_last_window()` | Window\|nil |
| `hl.get_workspaces()` | Workspace[] |
| `hl.get_workspace(selector)` | Workspace\|nil |
| `hl.get_last_workspace(monitor?)` | Workspace\|nil |
| `hl.get_monitors()` | Monitor[] |
| `hl.get_monitor(selector)` | Monitor\|nil |
| `hl.get_layers(filters?)` | LayerSurface[] |
| `hl.get_workspace_windows(selector)` | Window[] |
| `hl.get_config(key)` | current value of e.g. `"general.gaps_in"` (returns the typed value) |
| `hl.version()` | Hyprland version string |
| `hl.get_loaded_plugins()` | plugin list |

### Reading & mutating config at runtime
```lua
-- gaps_in comes back as a table {top,left,right,bottom} because it's css_gaps
local g = hl.get_config("general.gaps_in")
hl.config({ general = { gaps_in = (g.top == 3 and 0 or 3) } })
```
`hl.config()` merges; calling it from a bind/event updates live.

### `Window` object — complete field list

| field | type | notes |
| --- | --- | --- |
| `active` | bool\|nil | currently focused |
| `address` | string | `0x...` |
| `class` / `initial_class` | string | |
| `title` / `initial_title` | string | |
| `pid` | int | process id |
| `stable_id` | int | stable across reloads |
| `floating` | bool | |
| `fullscreen` | int | internal fullscreen mode (0/1/2/3) |
| `fullscreen_client` | int | client-visible mode |
| `fullscreen_handler` | string | which handler (default/layout) |
| `pinned` | bool | |
| `pin_fullscreened` | bool | pinned-then-fullscreened state |
| `xwayland` | bool | |
| `workspace` | Workspace\|nil | |
| `monitor` | Monitor\|nil | |
| `group` | Group\|nil | |
| `size` | table | `{w,h}` |
| `at` | table | `{x,y}` |
| `tags` | table | tags on this window |
| `xdg_tag` | string\|nil | |
| `xdg_description` | string\|nil | |
| `content_type` | string | none/photo/video/game |
| `hidden` / `mapped` / `visible` | bool | visibility states |
| `accepts_input` | bool | |
| `allowed_over_fullscreen` | bool | |
| `focus_history_id` | int | |
| `inhibiting_idle` | bool | |
| `swallowing` | Window\|nil | window this one swallowed |
| `layout` | mixed | layout reference |

### `Workspace` object — complete field/method list

| field/method | type | notes |
| --- | --- | --- |
| `id` | int | 1..2147483647 |
| `name` | string | |
| `config_name` | string | name from rules |
| `monitor` | Monitor\|nil | |
| `active` | bool | active on its monitor |
| `special` | bool | special/scratchpad workspace |
| `is_empty` | bool | |
| `is_persistent` | bool | |
| `has_fullscreen` | bool | |
| `has_urgent` | bool | |
| `fullscreen_mode` | int | |
| `fullscreen_window` | Window\|nil | |
| `windows` | int | window count |
| `groups` | int\|nil | group count |
| `last_window` | Window\|nil | |
| `visible` | bool | |
| `tiled_layout` | string | active tiled layout name |
| `:get_windows()` | | array of Window |
| `:get_groups()` | | groups on the workspace |

### `Monitor` object — complete field/method list

| field/method | type | notes |
| --- | --- | --- |
| `id` | int | |
| `name` | string | e.g. `DP-1` |
| `description` | string | |
| `serial` | string | |
| `width` / `height` | int | current mode |
| `physical_width` / `physical_height` | int | mm |
| `refresh_rate` | number | Hz |
| `scale` | number | |
| `transform` | int | 0..7 |
| `position` | table | `{x,y}` |
| `x` / `y` | int | |
| `size` | table | |
| `reserved` | table | `{top,right,bottom,left}` |
| `focused` | bool | |
| `dpms_status` | bool | |
| `vrr_active` | bool | |
| `cm` | string | color mode preset |
| `available_modes` | table | supported modes |
| `is_mirror` | bool | |
| `mirrors` | table | monitors mirroring this one |
| `active_workspace` | Workspace\|nil | |
| `active_special_workspace` | Workspace\|nil | |
| `:set_workspace(ws)` | | |
| `:set_special_workspace(ws)` | | |

### `Group` object — complete field/method list

| field/method | type | notes |
| --- | --- | --- |
| `current` | Window\|nil | active window in the group |
| `current_index` | int | |
| `size` | int | member count |
| `locked` | bool | |
| `denied` | bool | deny-from-group status |
| `members` | table | member windows |
| `:add(window, index?)` | | |
| `:remove(window\|index)` | | |

### `LayerSurface` object — complete field list

| field | type | notes |
| --- | --- | --- |
| `address` | string | |
| `namespace` | string | match key for layer rules |
| `pid` | int | |
| `layer` | int | z-layer |
| `x` / `y` / `w` / `h` | int | geometry |
| `monitor` | Monitor\|nil | |
| `mapped` | bool | |
| `interactivity` | int | keyboard interactivity mode |
| `above_fullscreen` | bool\|nil | |

### `Keybind` handle — fields
Read-only introspection (besides `:set_enabled(bool)`, `:is_enabled()`, `:unbind()`,
`:remove()`): `key`, `keycode`, `display_key`, `modmask`, `mouse`, `locked`,
`release`, `repeating`, `non_consuming`, `auto_consuming`, `transparent`,
`ignore_mods`, `dont_inhibit`, `long_press`, `click`, `drag`, `catchall`,
`submap`, `submap_universal`, `enabled`, `handler`, `arg`, `description`,
`has_description`, `device_inclusive`, `devices`.

### WindowQueryFilter (for `hl.get_windows`)
All the window-rule `match` props are accepted as filters: `class`, `title`,
`initial_class`, `initial_title`, `tag`, `xwayland`, `float`, `fullscreen`,
`fullscreen_state_client`, `fullscreen_state_internal`, `pin`, `focus`, `group`,
`modal`, `workspace`, `content`, `xdg_tag` (regex / bool / selectors).

### LayerQueryFilter (for `hl.get_layers`)
`namespace` (regex).

## Timers (`hl.timer`)

```lua
local t = hl.timer(function()
    print("tick")
end, { timeout = 1000, type = "repeat" })   -- type: "repeat" or "oneshot"

t:set_enabled(false)
t:set_enabled(true)
t:is_enabled()
t:set_timeout(2000)
```
`timeout` is milliseconds, must be > 0.

## Notifications (`hl.notification`)

The built-in notification system is now scriptable (the old way was `hyprctl notify`).

```lua
local n = hl.notification.create({ text = "Hello", timeout = 5000, icon = "ok", color = "rgb(ff1ea3)", font_size = 13 })
n:set_text("updated")
n:pause() / n:resume() / n:set_paused(bool)
n:is_alive() / n:is_paused()
n:get_elapsed() / n:get_elapsed_since_creation() / n:get_color() / n:get_font_size() / n:get_icon() / n:get_text() / n:get_timeout()
n:set_color(...) / n:set_font_size(...) / n:set_icon(...) / n:set_timeout(...)
n:dismiss()
hl.notification.get()    -- all live notifications
```
`icon` is an int (0 WARNING, 1 INFO, 2 HINT, 3 ERROR, 4 CONFUSED, 5 OK) or a
string name. `timeout` ms (0 = sticky).

> From the shell you can still use `hyprctl notify [ICON] [TIME_MS] [COLOR] [MSG]`
> and `hyprctl dismissnotify [N]`.

## Permissions (`hl.permission`)

```ini
# 0.54
permission = /usr/(bin|local/bin)/grim, screencopy, allow
ecosystem { enforce_permissions = 1 }
```
```lua
-- 0.55
hl.config({ ecosystem = { enforce_permissions = true } })
hl.permission({ binary = "/usr/(bin|local/bin)/grim", type = "screencopy", mode = "allow" })
-- also accepts positional form: hl.permission("/usr/.../grim", "screencopy", "allow")
```
Types: `screencopy`, `cursorpos`, `plugin`, `keyboard`. Modes: `allow`, `ask`, `deny`.
Permissions are **not** reloaded on-the-fly (restart required). Requires `hyprland-guiutils`.

## Custom layouts (`hl.layout.register`) — NEW

Write a layout entirely in Lua. Register it, then select it as `lua:name`.

```lua
hl.layout.register("columns", {
    recalculate = function(ctx)
        local n = #ctx.targets
        if n == 0 then return end
        for i, target in ipairs(ctx.targets) do
            target:place(ctx:column(i, n))
        end
    end,
})

-- use it
hl.workspace_rule({ workspace = "5", layout = "lua:columns" })
```

### `ctx` (LayoutContext)
- `ctx.area` — work area `Box`
- `ctx.targets` — array of `LayoutTarget`
- `ctx:grid_cell(i, cols, rows?)` — Box for a grid cell
- `ctx:column(i, n)` / `ctx:row(i, n)` — Box for the i-th of n columns/rows
- `ctx:split(box, side, ratio)` — split a Box (`side` = left/right/top/bottom/up/down)

### `LayoutTarget`
- `.window` — the main window (may be nil; groups expose the main one)
- `.index`
- `:place(box)` — **preferred**; computes gaps/pseudotile/reserved for you
- `:set_box(box)` — raw positioning only if absolutely necessary

### Optional `layout_msg`
```lua
hl.layout.register("spiral", {
    recalculate = function(ctx) ... end,
    layout_msg  = function(ctx, msg)
        -- parse msg; return true on success, a string error, or nil
        return true
    end,
})
```
Send messages via `hl.dsp.layout("ratio 0.5")` etc. Example layouts ship in
`example/layouts/` (`columns`, `grid`, `spiral`, `manual`).

## Plugins (`hl.plugin.load`)

```ini
# 0.54
plugin = /path/to/plugin.so
```
```lua
-- 0.55
hl.plugin.load("/path/to/plugin.so")
hl.get_loaded_plugins()    -- { {name, author, version, description}, ... }
```

## Misc utilities

| function | purpose |
| --- | --- |
| `hl.is_key_down(key)` | is a key (keysym string or int keycode) currently held |
| `hl.unbind("all")` | remove every keybind |
| `hl.exec_scheduled_prop_refresh_immediately()` | run a pending prop refresh now |
| `hl.clear_crashed_lockscreen()` | dismiss a crashed lockscreen |
| `print(...)` | logs to Hyprland's log at INFO (Hyprland overrides Lua's `print`) |

## Hyprctl still works

`hyprctl` is unchanged in spirit but now speaks Lua too:
```sh
hyprctl eval     'hl.dispatch(hl.dsp.focus({ workspace = "3" }))'
hyprctl dispatch 'hl.dsp.window.close()'
hyprctl repl                       # interactive Lua
hyprctl repl 'hl.get_active_window().class'
hyprctl reload            # reload config
hyprctl reload full-reset # recreate context (switch lua/hyprlang)
```
Info commands unchanged: `version`, `monitors [all]`, `workspaces`,
`workspacerules`, `clients`, `devices`, `decorations [win]`, `binds`,
`activewindow`, `layers`, `splash`, `getoption sec.opt`, `cursorpos`,
`animations`, `instances`, `layouts`, `configerrors`, `rollinglog [-f]`,
`locked`, `descriptions`, `submap`. Flags: `-j` JSON, `-i INST`, `-r` refresh,
`--batch 'a; b'`.
