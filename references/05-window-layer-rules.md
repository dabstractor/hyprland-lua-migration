# Reference 05 — Window & Layer Rules

> Companion to `SKILL.md`. Read this for the exhaustive mapping of every window
> rule and layer rule from 0.54 to 0.55.

## Window rules

### Syntax

0.54 had `windowrule=` (V1) and `windowrulev2=` (V2) and the `windowrule {}`
block. **All of these are unified into `hl.window_rule({...})`.**

```ini
# 0.54 V1
windowrule = float, ^(kitty)$
# 0.54 V2
windowrulev2 = float, class:^(kitty)$, title:^(kitty)$
# 0.54 block
windowrule {
    name = my-rule
    match:class = .*
    suppress_event = maximize
}
```
```lua
-- 0.55 (anonymous)
hl.window_rule({ match = { class = "kitty" }, float = true })

-- 0.55 (named — required for runtime enable/disable)
hl.window_rule({
    name  = "suppress-maximize-events",
    match = { class = ".*" },
    suppress_event = "maximize",
})
```

A rule = a `match` table (props) + effect fields. **All** match props must hold.
Effects split into **static** (evaluated once at open) and **dynamic**
(re-evaluated whenever a property changes).

### Match props (`match = { ... }`)

| field | arg | notes |
| --- | --- | --- |
| `class` | regex | |
| `title` | regex | |
| `initial_class` | regex | |
| `initial_title` | regex | |
| `tag` | name | |
| `xwayland` | bool | |
| `float` | bool | (was `floating:0/1`) |
| `fullscreen` | bool | |
| `pin` | bool | (was `pinned:0/1`) |
| `focus` | bool | (was `focus:0/1`) |
| `group` | bool **NEW** | |
| `modal` | bool **NEW** | |
| `fullscreen_state_client` | int **NEW** | 0/1/2/3 |
| `fullscreen_state_internal` | int **NEW** | 0/1/2/3 |
| `workspace` | selector **NEW** | id, `"name:..."`, or selector |
| `content` | string **NEW** | none/photo/video/game |
| `xdg_tag` | regex **NEW** | |

Old `onworkspace` (window count) prop is replaced by workspace selectors in the
`workspace` match field (e.g. `workspace = "w[tv1]"`).

> Regex uses Google RE2. To negate, prefix `negative:`: `match = { class = "negative:kitty" }`.

### Static effects

| 0.54 rule | 0.55 effect | type / notes |
| --- | --- | --- |
| `float` | `float` | bool |
| `tile` | `tile` | bool |
| `fullscreen` | `fullscreen` | bool |
| `fakefullscreen` | **REMOVED** | use `fullscreen_state` |
| `maximize` | `maximize` | bool |
| — | `fullscreen_state` **NEW** | string e.g. `"1 2"` (internal client; 0/1/2/3) |
| `move x y` | `move` | `{x,y}` or expression strings |
| `size x y` | `size` | `{x,y}` or expression strings |
| `center` | `center` | bool |
| `pseudo` | `pseudo` | bool |
| `monitor id` | `monitor` | string e.g. `"1"`/`"DP-1"` (+`" silent"`) |
| `workspace w` | `workspace` | string (`"unset"` or +`" silent"`) |
| `noinitialfocus` | `no_initial_focus` | bool |
| `nofocus` | `no_focus` | bool |
| `pin` | `pin` | bool (floats only) |
| `group [opts]` | `group` | string of space-separated opts |
| `ignoreevent types` | `suppress_event` | string, space-separated (see below) |
| — | `content` **NEW** | `"none"`/`"photo"`/`"video"`/`"game"` |
| — | `no_close_for` **NEW** | int ms |
| — | `scrolling_width` **NEW** | float (scrolling layout column width) |
| `minsize x y` | `min_size` **NEW** | vec2 `{x,y}` |
| `maxsize x y` | `max_size` **NEW** | vec2 `{x,y}` |

#### `move` / `size` expressions
```lua
move = {100, 200}
move = {"cursor_x-(window_w*0.5)", "cursor_y-(window_h*0.5)"}
size = {"monitor_w*0.5", "monitor_h*0.5"}
```
Variables: `monitor_w/h`, `window_x/y/w/h`, `cursor_x/y` (monitor-local). The old
`move cursor X% Y%` and `move onscreen ...` forms are replaced by these
expressions (e.g. cursor-centering above).

#### `group` options (unchanged vocabulary)
`"set"`/`"set always"`, `"new"` (= `"barred set"`), `"lock"`/`"lock always"`,
`"barred"`, `"deny"`, `"invade"`, `"override"`, `"unset"`. Bare `group = ""` == `"set"`.

#### `suppress_event` types (expanded)
`"fullscreen"`, `"maximize"`, `"activate"`, `"activatefocus"`, `"fullscreenoutput"`,
`"x11configurerequest"` (NEW) — space-separated.

### Dynamic effects

| 0.54 rule | 0.55 effect | type / notes |
| --- | --- | --- |
| `opacity a` | `opacity` | string: `"0.8"`, `"0.9 0.7"`, `"1.0 0.8 0.9"`, append `" override"` per value |
| `bordercolor c` | `border_color` | gradient/color, or two (active/inactive) |
| `rounding x` | `rounding` | int |
| `bordersize x` | `border_size` | int |
| `animation style` | `animation` | string e.g. `"popin 80%"` |
| `idleinhibit mode` | `idle_inhibit` | `"none"`/`"always"`/`"focus"`/`"fullscreen"` |
| `noblur` | `no_blur` | bool |
| `noborder` | **REMOVED** | use `border_size = 0` |
| `nodim` | `no_dim` | bool |
| `noshadow` | `no_shadow` | bool |
| `noanim` | `no_anim` | bool |
| `nomaxsize` | `no_max_size` | bool |
| `stayfocused` | `stay_focused` | bool |
| `forceinput` | `allows_input` | bool |
| `keepaspectratio` | `keep_aspect_ratio` | bool |
| `nearestneighbor` | `nearest_neighbor` | bool |
| `opaque` | `opaque` | bool |
| `forcergbx` | `force_rgbx` | bool |
| `dimaround` | `dim_around` | bool |
| `xray on` | `xray` | bool |
| `immediate` | `immediate` | bool (tearing) |
| `unset` | (gone) | use handle `:set_enabled(false)` instead |
| — | `tag` **NEW** | `"+x"`/`"-x"`/`"x"` (dynamic tags) |
| — | `persistent_size` **NEW** | bool (remember float size by class+title) |
| — | `rounding_power` **NEW** | float |
| — | `scroll_mouse` **NEW** | float |
| — | `scroll_touchpad` **NEW** | float |
| — | `decorate` **NEW** | bool |
| — | `focus_on_activate` **NEW** | bool |
| — | `no_follow_mouse` **NEW** | bool |
| — | `no_shortcuts_inhibit` **NEW** | bool |
| — | `no_screen_share` **NEW** | bool |
| — | `no_vrr` **NEW** | bool |
| — | `no_auto_hdr` **NEW** | bool |
| — | `sync_fullscreen` **NEW** | bool |
| — | `render_unfocused` **NEW** | bool |
| — | `confine_pointer` **NEW** | bool |
| — | `tonemap` **NEW** | `"on"`/`"off"`/`"clamp"`/`"limited"` |
| — | `max_size`/`min_size` **NEW** | vec2 |

### Dynamic enabling/disabling rules (NEW)

Only **named** rules can be toggled at runtime; `hl.window_rule()` returns a handle:
```lua
local r = hl.window_rule({ name = "kitty-borders", match = { class = "kitty" }, border_size = 5 })
r:set_enabled(false)
r:set_enabled(true)
r:is_enabled()
```

> **Named rules are evaluated before anonymous ones.** Within each group, rules
> run top-to-bottom and the **last** match wins (for overlapping effects).

### Opacity semantics (unchanged)
Opacity is a **product** of all matching opacities (and the global
`active_opacity`/`inactive_opacity`). Use `" override"` to set absolute instead
of multiplied. Any product > 1.0 glitches.

### Example translations

```ini
# 0.54
windowrule = move 100 100, ^(kitty)$
windowrule = animation popin, ^(kitty)$
windowrule = noblur, ^(firefox)$
windowrulev2 = bordercolor rgb(FF0000) rgb(880808), fullscreen:1
windowrule = opacity 1.0 override 0.5 override, ^(kitty)$
windowrule = rounding 10, ^(kitty)$
```
```lua
-- 0.55
hl.window_rule({ match = { class = "kitty" }, move = {100, 100}, animation = "popin" })
hl.window_rule({ match = { class = "firefox" }, no_blur = true })
hl.window_rule({ match = { fullscreen = true }, border_color = "rgb(FF0000) rgb(880808)" })
hl.window_rule({ match = { class = "kitty" }, opacity = "1.0 override 0.5 override" })
hl.window_rule({ match = { class = "kitty" }, rounding = 10 })
```

### Tags (NEW, two kinds)

**Static tags** — applied via dispatcher:
```lua
hl.bind("SUPER + G", hl.dsp.window.tag({ tag = "+code" }))          -- add to active window
hl.bind("SUPER + G", hl.dsp.window.tag({ tag = "+code", window = "class:Celluloid" }))
```
**Dynamic tags** (suffix `*`) — applied via the `tag` effect, re-evaluated:
```lua
hl.window_rule({ match = { class = "footclient" }, tag = "+term" })   -- dynamic tag term*
hl.window_rule({ match = { tag = "code" }, opacity = "0.8" })         -- matches code or code*
hl.window_rule({ match = { tag = "term*" }, opacity = "0.6" })        -- term* only
```
Toggle opacity tiers with keybinds:
```lua
hl.bind("SUPER + CTRL + 2", hl.dsp.window.tag({ tag = "alpha_0.2" }))
hl.window_rule({ match = { tag = "alpha_0.2" }, opacity = "0.2 override" })
```

## Layer rules

Layers (bars, launchers, wallpapers) are not windows. Old `layerrule = rule, ns`
→ `hl.layer_rule({...})`.

```ini
# 0.54
layerrule = blur, waybar
layerrule = ignorealpha 0.5, rofi
layerrule = unset, waybar
```
```lua
-- 0.55
hl.layer_rule({ match = { namespace = "waybar" }, blur = true })
hl.layer_rule({ match = { namespace = "rofi" }, blur = true, ignore_alpha = 0.5 })

-- named (toggleable)
local r = hl.layer_rule({ name = "no-anim-selection", match = { namespace = "selection" }, no_anim = true })
r:set_enabled(false)
```

### Match
| field | arg |
| --- | --- |
| `namespace` | regex (find with `hyprctl layers`) |

### Effects

| 0.54 rule | 0.55 effect | type |
| --- | --- | --- |
| `noanim` | `no_anim` | bool |
| `blur` | `blur` | bool |
| — | `blur_popups` **NEW** | bool |
| `ignorealpha a` | `ignore_alpha` | float 0..1 |
| `ignorezero` | `ignore_alpha = 0` | (merged) |
| `xray on` | `xray` | bool |
| — | `dim_around` **NEW** | bool |
| — | `animation` **NEW** | string |
| — | `order` **NEW** | int (relative layer order; can be negative) |
| — | `above_lock` **NEW** | int (0/1/2; 2 = interactive on lockscreen) |
| — | `no_screen_share` **NEW** | bool |
| `unset` | (gone) | use `:set_enabled(false)` |

There is no `address:` form anymore — match by `namespace`.

## Summary: block-style `windowrule {}` → `hl.window_rule`

The multi-match lines of the old block (`match:class = X`, `match:title = Y`)
collapse into **one** `match` table:
```ini
# 0.54
windowrule {
    name = fix-xwayland
    match:class = ^$
    match:title = ^$
    match:xwayland = true
    no_focus = true
}
```
```lua
-- 0.55
hl.window_rule({
    name  = "fix-xwayland",
    match = { class = "^$", title = "^$", xwayland = true },
    no_focus = true,
})
```
