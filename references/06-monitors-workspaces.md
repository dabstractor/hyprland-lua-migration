# Reference 06 — Monitors & Workspace Rules

> Companion to `SKILL.md`. Read this for the exhaustive monitor and
> workspace-rule translations, including all the new HDR/color-management fields.

## Monitors

### Syntax

```ini
# 0.54 — comma-separated positional fields
monitor = name, resolution, position, scale
monitor = , preferred, auto, auto          # fallback rule
monitor = desc:Chimei Innolux 0x150C, preferred, auto, 1.5
```
```lua
-- 0.55 — named fields in a table
hl.monitor({ output = "DP-1", mode = "1920x1080@144", position = "0x0", scale = 1 })
hl.monitor({ output = "", mode = "preferred", position = "auto", scale = "auto" })  -- fallback
hl.monitor({ output = "desc:Chimei Innolux Corporation 0x150C", mode = "preferred", position = "auto", scale = 1.5 })
```

Only `output` is required; all other fields fall back to sensible defaults.

### Field reference

| field | type | default | description |
| --- | --- | --- | --- |
| `output` | string | required | name or `desc:`+description (drop the `(portname)`!) |
| `mode` | string | `preferred` | `WxH@RR`, or `preferred`/`highres`/`highrr`/`maxwidth`, or `modeline ...` |
| `position` | string | `auto` | `XxY`, or `auto`/`auto-right`/`-left`/`-up`/`-down`/`-center-*` |
| `scale` | string/float | `auto` | float, or `"auto"` |
| `disabled` | bool | false | removes from layout (use `dpms` dispatcher for screensaver-style off) |
| `transform` | int | 0 | rotation/flip (0 normal, 1=90°, 2=180°, 3=270°, 4=flip, 5..7 flipped) |
| `mirror` | string | | output name to mirror |
| `bitdepth` | int | 8 | 8 or 10 |
| `cm` | string | `"srgb"` | `auto`/`srgb`/`dcip3`/`dp3`/`adobe`/`wide`/`edid`/`hdr`/`hdredid` |
| `sdr_eotf` | string | `"default"` | `"default"`/`"gamma22"`/`"gamma22force"`/`"srgb"` |
| `sdrbrightness` | float | 1.0 | SDR brightness in HDR mode |
| `sdrsaturation` | float | 1.0 | SDR saturation in HDR mode |
| `vrr` | int | 0 | per-display VRR (0/1/2/3) |
| `icc` | string | | absolute path to ICC profile |
| `reserved_area` | int or table | 0 | int (all sides) or `{top=,right=,bottom=,left=}` (also accepts `reserved`) |
| `supports_wide_color` | int | 0 | -1 off / 0 auto / 1 on |
| `supports_hdr` | int | 0 | -1 off / 0 auto / 1 on |
| `sdr_min_luminance` | float | 0.2 | |
| `sdr_max_luminance` | int | 80 | |
| `min_luminance` | float | -1 | |
| `max_luminance` | int | -1 | |
| `max_avg_luminance` | int | -1 | |

### Special mode/position values

mode: `preferred`, `highres`, `highrr`, `maxwidth`.
position: `auto`, `auto-right/left/up/down`, `auto-center-right/left/up/down`.

Negative coordinates allowed (inverse-Y cartesian: negative Y = higher).

### Custom modeline
```lua
hl.monitor({
    output = "DP-1",
    mode = "modeline 1071.101 3840 3848 3880 3920 2160 2263 2271 2277 +hsync -vsync",
    position = "0x0", scale = 1,
})
```

### Disable
```ini
# 0.54
monitor = eDP-1, disable
```
```lua
-- 0.55
hl.monitor({ output = "eDP-1", disabled = true })
```

### Mirror
```ini
# 0.54
monitor = DP-3, 1920x1080@60, 0x0, 1, mirror, DP-2
```
```lua
-- 0.55
hl.monitor({ output = "DP-3", mode = "1920x1080@60", position = "0x0", scale = 1, mirror = "DP-2" })
```

### 10-bit
```ini
# 0.54
monitor = eDP-1, 2880x1800@90, 0x0, 1, bitdepth, 10
```
```lua
-- 0.55
hl.monitor({ output = "eDP-1", mode = "2880x1800@90", position = "0x0", scale = 1, bitdepth = 10 })
```

### HDR
```lua
hl.monitor({
    output = "eDP-1", mode = "2880x1800@90", position = "0x0", scale = 1,
    bitdepth = 10, cm = "hdr", sdrbrightness = 1.2, sdrsaturation = 0.98,
})
```

### ICC
```lua
hl.monitor({ output = "eDP-1", icc = "/absolute/path.icm" })  -- path must be absolute
```

### Reserved area (custom)
```lua
hl.monitor({ output = "name", reserved_area = 10 })                                      -- all sides
hl.monitor({ output = "name", reserved_area = { top = 10, bottom = 10, left = 0, right = 0 } })
```

### VRR per-display
```lua
hl.monitor({ output = "DP-1", vrr = 1 })
```

### Transform / rotation
```lua
hl.monitor({ output = "eDP-1", mode = "2880x1800@90", position = "0x0", scale = 1, transform = 1 })
-- 0 normal, 1 90°, 2 180°, 3 270°, 4 flip, 5 flip+90, 6 flip+180, 7 flip+270
```

> Position math uses the **scaled + transformed** resolution. A 4K@scale-2 to the
> left of a 1080p screen uses `1920x0`. If the 4K is also rotated 90°, use `1080x0`.

## Workspace rules

### Syntax

```ini
# 0.54
workspace = WORKSPACE, rule1:value1, rule2:value2
workspace = name:coding, gapsin:0, gapsout:0, monitor:DP-1
```
```lua
-- 0.55
hl.workspace_rule({ workspace = "...", rule1 = value1, rule2 = value2 })
hl.workspace_rule({ workspace = "name:coding", gaps_in = 0, gaps_out = 0, monitor = "DP-1" })
```

`workspace` is mandatory and may be a workspace selector (but selectors only match
**existing** workspaces).

### Workspace selectors (NEW, more powerful)

Selectors combine props separated by spaces (no spaces inside a prop):
- `r[A-B]` — ID range inclusive
- `s[bool]` — special or not
- `n[bool]` / `n[s:...]` / `n[e:...]` — named actions (is-named / starts-with / ends-with)
- `m[monitor]` — monitor selector
- `w[(flags)A-B]` / `w[(flags)X]` — window counts; flags `t` tiled, `f` floating, `g` groups, `v` visible, `p` pinned
- `f[-1|0|1|2]` — fullscreen state (-1 none, 0 fullscreen, 1 maximized, 2 fs w/o client state)

Example: `"w[tv1]s[false]"` = tiled/visible count 1, not special.

### Rule fields

| 0.54 rule | 0.55 field | type |
| --- | --- | --- |
| `monitor:m` | `monitor` | string |
| `default:b` | `default` | bool |
| `persistent:b` | `persistent` | bool |
| `gapsin:x` | `gaps_in` | css_gaps |
| `gapsout:x` | `gaps_out` | css_gaps |
| — | `float_gaps` **NEW** | css_gaps |
| `bordersize:x` | `border_size` | int |
| `border:b` | `no_border` **INVERTED** | bool (true = no border) |
| `shadow:b` | `no_shadow` **INVERTED** | bool |
| `rounding:b` | `no_rounding` **INVERTED** | bool |
| `decorate:b` | `decorate` | bool |
| `on-created-empty:c` | `on_created_empty` | string |
| — | `default_name` **NEW** | string |
| — | `layout` **NEW** | string (per-workspace layout) |
| — | `animation` **NEW** | string (animation style) |
| — | `layout_opts` **NEW** | table of layout-specific opts |

Note the **inverted** naming: old `border:false` (don't draw) → new `no_border = true`.

### Examples
```ini
# 0.54
workspace = 3, rounding:false, decorate:false
workspace = name:Hello, monitor:DP-1, default:true
workspace = 5, on-created-empty:[float] firefox
workspace = special:scratchpad, on-created-empty:foot
```
```lua
-- 0.55
hl.workspace_rule({ workspace = "3", no_rounding = true, decorate = false })
hl.workspace_rule({ workspace = "name:Hello", monitor = "DP-1", default = true })
hl.workspace_rule({ workspace = "5", on_created_empty = "[float] firefox" })
hl.workspace_rule({ workspace = "special:scratchpad", on_created_empty = "foot" })
```

### Per-workspace layout & layout_opts
```lua
hl.workspace_rule({ workspace = "2", layout = "scrolling" })
hl.workspace_rule({ workspace = "2", layout_opts = { orientation = "top" } })   -- master
hl.workspace_rule({ workspace = "2", layout_opts = { direction = "right" } })  -- scrolling
```

### Named/toggleable workspace rules (NEW)
`hl.workspace_rule()` returns a handle with `:set_enabled()` / `:is_enabled()`.

### Smart gaps ("no gaps when only")
```lua
hl.workspace_rule({ workspace = "w[tv1]", gaps_out = 0, gaps_in = 0 })
hl.workspace_rule({ workspace = "f[1]",   gaps_out = 0, gaps_in = 0 })
hl.window_rule({ match = { float = false, workspace = "w[tv1]" }, border_size = 0, rounding = 0 })
hl.window_rule({ match = { float = false, workspace = "f[1]"   }, border_size = 0, rounding = 0 })
```
