# Reference 07 — Animations, Curves & Gestures

> Companion to `SKILL.md`. Read this for the exhaustive translation of bezier/
> animation declarations and the brand-new gesture system.

## Animations overview

In 0.54 curves and animations were declared inside an `animations {}` block with
positional `bezier=`/`animation=` lines. In 0.55 they are **separate top-level
functions**: `hl.curve()` and `hl.animation()`. The `animations` config section
now only holds the master `enabled` flag (and `workspace_wraparound`).

```ini
# 0.54
animations {
    enabled = yes
    bezier = mycurve, 0.25, 0.1, 0.25, 1
    animation = windows, 1, 8, mycurve, popin 80%
    animation = workspaces, 1, 8, default
}
```
```lua
-- 0.55
hl.config({ animations = { enabled = true } })
hl.curve("mycurve", { type = "bezier", points = { {0.25, 0.1}, {0.25, 1} } })
hl.animation({ leaf = "windows",    enabled = true, speed = 8, bezier = "mycurve", style = "popin 80%" })
hl.animation({ leaf = "workspaces", enabled = true, speed = 8, bezier = "default" })
```

## Bezier curves

```ini
# 0.54 — NAME, X0, Y0, X1, Y1
bezier = easeOutQuint, 0.23, 1, 0.32, 1
```
```lua
-- 0.55
hl.curve("easeOutQuint", { type = "bezier", points = { {0.23, 1}, {0.32, 1} } })
```
The two `points` are the two configurable middle control points of the cubic
Bézier (start/end are fixed at {0,0} and {1,1}). Each point is a `{x, y}` table.

## Spring curves

```lua
hl.curve("easy", { type = "spring", mass = 1, stiffness = 71.2633, dampening = 15.8273644 })
```
All of `mass`, `stiffness`, `dampening` required and must be ≥ 0.5. More
stiffness = more speed; more dampening = less bounce. Keep mass at 1 usually.

## `hl.animation()` fields

```lua
hl.animation({ leaf = STRING, enabled = BOOL, speed = FLOAT, bezier = NAME | spring = NAME [, style = STRING] })
```
- `leaf` — animation scope (see tree below)
- `enabled` — `false` disables (then other args optional)
- `speed` — duration in **deciseconds** (1 ds = 100 ms); must be > 0
- `bezier` **or** `spring` — curve name (exactly one required when enabled)
- `style` — optional style string

If disabled, omit everything else:
```lua
hl.animation({ leaf = "fade", enabled = false })
```

### Animation tree (inheritance from parent)

```
global
 ├ windows        styles: slide, popin, gnomed
 │  ├ windowsIn
 │  ├ windowsOut
 │  └ windowsMove
 ├ layers         styles: slide, popin, fade
 │  ├ layersIn
 │  └ layersOut
 ├ fade
 │  ├ fadeIn
 │  ├ fadeOut
 │  ├ fadeSwitch
 │  ├ fadeShadow
 │  ├ fadeGlow
 │  ├ fadeDim
 │  ├ fadeLayers ├ fadeLayersIn └ fadeLayersOut
 │  ├ fadePopups ├ fadePopupsIn └ fadePopupsOut
 │  └ fadeDpms
 ├ border
 ├ borderangle    styles: once (default), loop
 ├ shadowangle    styles: once, loop
 ├ glowangle      styles: once, loop
 ├ workspaces     styles: slide, slidevert, fade, slidefade, slidefadevert
 │  ├ workspacesIn
 │  ├ workspacesOut
 │  └ specialWorkspace ├ specialWorkspaceIn └ specialWorkspaceOut
 ├ zoomFactor
 └ monitorAdded
```

### Style extras
- `popin` accepts a start percentage: `style = "popin 80%"` (80%→100%).
- workspace `slide`/`slidevert`/`slidefade`/`slidefadevert` accept a movement
  percentage: `style = "slidefade 20%"`.
- `windows`/`layers` `slide` accepts a forced side: `style = "slide left"`.
- `loop` on `*angle` forces constant re-rendering at refresh rate (battery cost).

### Full default set (from the example config)
```lua
hl.curve("easeOutQuint",   { type = "bezier", points = { {0.23, 1},    {0.32, 1} } })
hl.curve("easeInOutCubic", { type = "bezier", points = { {0.65, 0.05}, {0.36, 1} } })
hl.curve("linear",         { type = "bezier", points = { {0, 0},       {1, 1} } })
hl.curve("almostLinear",   { type = "bezier", points = { {0.5, 0.5},   {0.75, 1} } })
hl.curve("quick",          { type = "bezier", points = { {0.15, 0},    {0.1, 1} } })
hl.curve("easy",           { type = "spring", mass = 1, stiffness = 71.2633, dampening = 15.8273644 })

hl.animation({ leaf = "global",       enabled = true, speed = 10,   bezier = "default" })
hl.animation({ leaf = "border",       enabled = true, speed = 5.39, bezier = "easeOutQuint" })
hl.animation({ leaf = "windows",      enabled = true, speed = 4.79, spring = "easy" })
hl.animation({ leaf = "windowsIn",    enabled = true, speed = 4.1,  spring = "easy", style = "popin 87%" })
hl.animation({ leaf = "windowsOut",   enabled = true, speed = 1.49, bezier = "linear", style = "popin 87%" })
hl.animation({ leaf = "fadeIn",       enabled = true, speed = 1.73, bezier = "almostLinear" })
hl.animation({ leaf = "fadeOut",      enabled = true, speed = 1.46, bezier = "almostLinear" })
hl.animation({ leaf = "fade",         enabled = true, speed = 3.03, bezier = "quick" })
hl.animation({ leaf = "layers",       enabled = true, speed = 3.81, bezier = "easeOutQuint" })
hl.animation({ leaf = "layersIn",     enabled = true, speed = 4,    bezier = "easeOutQuint", style = "fade" })
hl.animation({ leaf = "layersOut",    enabled = true, speed = 1.5,  bezier = "linear", style = "fade" })
hl.animation({ leaf = "fadeLayersIn", enabled = true, speed = 1.79, bezier = "almostLinear" })
hl.animation({ leaf = "fadeLayersOut",enabled = true, speed = 1.39, bezier = "almostLinear" })
hl.animation({ leaf = "workspaces",   enabled = true, speed = 1.94, bezier = "almostLinear", style = "fade" })
hl.animation({ leaf = "workspacesIn", enabled = true, speed = 1.21, bezier = "almostLinear", style = "fade" })
hl.animation({ leaf = "workspacesOut",enabled = true, speed = 1.94, bezier = "almostLinear", style = "fade" })
hl.animation({ leaf = "zoomFactor",   enabled = true, speed = 7,    bezier = "quick" })
```

## Gestures (NEW system)

**0.54** gestures were limited to a single `gesture = fingers, direction, action`
keyword plus a pile of `workspace_swipe*` config vars. **0.55** replaces all of
that with `hl.gesture({...})`, which supports many actions, modifiers, scaling,
live lua callbacks, and pinch/zoom.

### Basic syntax
```lua
hl.gesture({
    fingers   = 3,
    direction = "horizontal",
    action    = "workspace",   -- or a lua function / callback table
    mods      = "SUPER",       -- optional
    scale     = 1.5,           -- optional delta multiplier
    disable_inhibit = false,   -- optional
})
```

### Directions
`swipe` (any), `horizontal`, `vertical`, `left`, `right`, `up`, `down`,
`pinch` (any), `pinchin`, `pinchout`.

### Built-in actions

| action | description | extra args |
| --- | --- | --- |
| `workspace` | switch workspaces | — |
| `move` | move active window | — |
| `resize` | resize active window | — |
| `special` | toggle a special workspace | `workspace_name` |
| `close` | close active window | — |
| `fullscreen` | fullscreen active window | `mode = "maximize"` for maximize |
| `float` | float active window | `mode = "float"`/`"tile"` to force direction |
| `cursorZoom` / `cursorZoom` | zoom into cursor | `zoom_level`, `mode = "mult"`/`"live"` |
| `scroll_move` | scroll the tape (scrolling layout) | — |
| `unset` | remove a previously-set gesture | must match all original fields |

### Lua-function actions
```lua
-- simple lambda
hl.gesture({ fingers = 4, direction = "up", action = function()
    hl.notification.create({ text = "swiped!", timeout = 2000 })
end })

-- LIVE gesture: a table with start/update/end(finish)
hl.gesture({
    fingers = 3, direction = "up",
    action = {
        start  = function(e) ... end,
        update = function(e) ... end,   -- e.delta, e.type, ...
        finish = function(e) ... end,
    },
})
```
`e` carries gesture details (type, delta, etc.); print it to a notification to
inspect fields.

### Restoring the old 3-finger workspace swipe
```lua
hl.gesture({ fingers = 3, direction = "horizontal", action = "workspace" })
```

### Examples
```lua
-- 3-finger horizontal → switch workspaces
hl.gesture({ fingers = 3, direction = "horizontal", action = "workspace" })

-- 3-finger down + SUPER → toggle scratchpad, bypassing inhibitors
hl.gesture({ fingers = 4, direction = "down", mods = "SUPER",
             action = "special", workspace_name = "scratchpad", disable_inhibit = true })

-- pinch to zoom cursor
hl.gesture({ fingers = 2, direction = "pinch", action = "cursorZoom", zoom_level = 1.2, mode = "mult" })

-- 3-finger up + ALT → close
hl.gesture({ fingers = 3, direction = "down", mods = "ALT", action = "close" })

-- 3-finger up + SUPER → fullscreen
hl.gesture({ fingers = 3, direction = "up", mods = "SUPER", scale = 1.5, action = "fullscreen" })
```

### `cursorZoom` modes
- default: toggles to `zoom_level`.
- `"mult"`: multiplies the current zoom by `zoom_level`.
- `"live"`: zoom tracks the pinch continuously, anchored to the cursor at gesture
  start. `zoom_level` is unused in live mode (use `1`).

### Gesture field reference

| field | type | notes |
| --- | --- | --- |
| `fingers` | int | 2–9 |
| `direction` | string | see directions |
| `action` | string \| function \| table | see above |
| `mods` | string | e.g. `"SUPER"` or `"ALT SHIFT"` |
| `scale` | float | gesture delta multiplier |
| `disable_inhibit` | bool | bypass shortcut inhibitors |
| `workspace_name` | string | for `special` |
| `mode` | string | `float`/`fullscreen`/`cursorZoom` modes |
| `zoom_level` | number | for `cursorZoom` |
