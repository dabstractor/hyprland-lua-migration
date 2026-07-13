# Reference 09 — Layouts (Dwindle / Master / Scrolling / Monocle + Custom)

> Companion to `SKILL.md`. Read this for the exhaustive config tables and layout
> messages for every built-in layout. These options were previously (0.54) set in
> `dwindle {}` / `master {}` blocks; in 0.55 they are `hl.config({ dwindle = {...} })`.

The layout is chosen with `general.layout` (`"dwindle"`/`"master"`/`"scrolling"`/`"monocle"`,
or `"lua:NAME"` for custom). Per-workspace overrides use `hl.workspace_rule({ layout = ... })`.

## Dwindle

BSPWM-like binary-tree layout. Split direction is dynamic (W>H → side-by-side)
unless `preserve_split` is on.

### Config — `hl.config({ dwindle = { ... } })`

| name | type | default | description |
| --- | --- | --- | --- |
| `force_split` | int | `0` | 0=follow mouse, 1=always split left/top, 2=always split right/bottom |
| `preserve_split` | bool | `false` | keep split orientation regardless of container ratio |
| `smart_split` | bool | `false` | split dir from cursor triangle (also enables preserve_split) |
| `smart_resizing` | bool | `true` | resize dir from mouse corner |
| `permanent_direction_override` | bool | `false` | make `preselect` persist |
| `special_scale_factor` | float | `1` | scale of windows on special workspace [0–1] |
| `split_width_multiplier` | float | `1.0` | auto-split width multiplier (wide screens) |
| `use_active_for_splits` | bool | `true` | prefer active window vs mouse for splits |
| `default_split_ratio` | float | `1.0` | default split ratio on open [0.1–1.9] |
| `split_bias` | int | `0` | 0=directional (top/left), 1=current window |
| `precise_mouse_move` | bool | `false` | `window.drag` drops more precisely by cursor pos |

> 0.54 `dwindle.pseudotile` config option was **REMOVED**. Pseudotiling is now a
> dispatcher (`hl.dsp.window.pseudo()`) / window rule (`pseudo = true`).

### Dwindle layout messages — `hl.dsp.layout("<msg>")`

| message | args | description |
| --- | --- | --- |
| `splitratio` | float delta, `+x`/`-x`, or `x exact` | change split ratio (default positive delta; `1.0 exact` centers) |
| `togglesplit` | — | toggle top/side split (needs `preserve_split`) |
| `swapsplit` | — | swap the two halves of the current split |
| `rotatesplit` | `[angle]` (multiple of 90; default 90) | rotate split (+cw, -ccw) |
| `preselect` | direction | one-time split-direction override for next window |
| `movetoroot` | `[window [ "unstable" ]]` | move window to root of its tree (maximize in subtree); `unstable` swaps instead |

```lua
hl.bind("SUPER + A", hl.dsp.layout("togglesplit"))
hl.bind("SUPER + Shift + J", hl.dsp.layout("rotatesplit 90"))
```

### Dwindle dispatcher
`hl.dsp.window.pseudo()` — toggle pseudotiling (was the `pseudo` dispatcher).

## Master

One master window + a slave stack.

### Config — `hl.config({ master = { ... } })`

| name | type | default | description |
| --- | --- | --- | --- |
| `allow_small_split` | bool | `false` | allow extra masters in a horizontal split |
| `special_scale_factor` | float | `1` | scale of special-workspace windows [0–1] |
| `mfact` | float | `0.55` | master size as fraction [0–1] |
| `new_status` | str | `"slave"` | `"master"`/`"slave"`/`"inherit"` — status of new windows |
| `new_on_top` | bool | `false` | new window on top of stack |
| `new_on_active` | str | `"none"` | `"before"`/`"after"`/`"none"` — place vs focused window |
| `orientation` | str | `"left"` | `"left"`/`"right"`/`"top"`/`"bottom"`/`"center"` |
| `slave_count_for_center_master` | int | `2` | min slaves to center master (0 = always) |
| `center_master_fallback` | str | `"left"` | fallback orientation when fewer slaves |
| `center_ignores_reserved` | bool | `false` | ignore reserved area in center mode |
| `smart_resizing` | bool | `true` | resize dir from mouse corner |
| `drop_at_cursor` | bool | `true` | drag-drop places at cursor; else stack top/bottom |
| `always_keep_position` | bool | `false` | keep master position when no slaves |
| `focus_master_on_close` | bool | `false` | closing a window focuses the master |

> 0.54 `master.gaps_in`/`master.gaps_out` config options were **REMOVED**. Gaps
> now live in `general.gaps_in/out` or per-workspace rules.

### Master layout messages — `hl.dsp.layout("<msg>")`

| message | args | description |
| --- | --- | --- |
| `swapwithmaster` | `master`/`child`/`auto` (+`ignoremaster`) | swap current with master |
| `focusmaster` | `master`/`auto`/`previous` | focus the master |
| `cyclenext` | `loop`/`noloop` | focus next (loop default) |
| `cycleprev` | `loop`/`noloop` | focus previous |
| `swapnext` | `loop`/`noloop` | swap with next |
| `swapprev` | `loop`/`noloop` | swap with previous |
| `addmaster` | — | add a master |
| `removemaster` | — | remove a master |
| `orientationleft`/`right`/`top`/`bottom`/`center` | — | set workspace orientation |
| `orientationnext` / `orientationprev` | — | cycle orientation cw/ccw |
| `orientationcycle` | list (`left top right ...`) | cycle through a list |
| `mfact` | delta (`-0.2`/`+0.2`) or `exact X` | change mfact |
| `rollnext` / `rollprev` | — | rotate next/prev stack window to master, keep focus on master |

```lua
hl.bind("SUPER + J", hl.dsp.layout("cyclenext"))
hl.bind("SUPER + Return", hl.dsp.layout("swapwithmaster master"))   -- xmonad-style promote
hl.bind("SUPER + I", hl.dsp.layout("mfact +0.05"))
```

### Master workspace rule (`layout_opts`)
```lua
hl.workspace_rule({ workspace = "2", layout_opts = { orientation = "top" } })
```

## Scrolling

Windows placed on an infinitely growing tape.

### Config — `hl.config({ scrolling = { ... } })`

| name | type | default | description |
| --- | --- | --- | --- |
| `fullscreen_on_one_column` | bool | `true` | a single column spans the whole screen |
| `column_width` | float | `0.5` | default column width [0.1–1.0] |
| `focus_fit_method` | int | `1` | 0=center, 1=fit when focusing a column |
| `follow_focus` | bool | `true` | move layout to bring focused window into view |
| `follow_min_visible` | float | `0.4` | min fraction visible for focus-follow [0–1] |
| `explicit_column_widths` | str | `"0.333, 0.5, 0.667, 1.0"` | preconfigured widths for `+conf`/`-conf` |
| `direction` | str | `"right"` | `"left"`/`"right"`/`"down"`/`"up"` |
| `wrap_focus` | bool | `true` | wrap `focus l/r` at ends |
| `wrap_swapcol` | bool | `true` | wrap `swapcol l/r` at ends |

### Scrolling layout messages — `hl.dsp.layout("<msg>")`

| message | args | description |
| --- | --- | --- |
| `move` | rel px (`-200`/`+200`) or columns (`+col`/`-col`) | move the tape |
| `colresize` | `0.5`/`+0.2`/`-0.2`/`+conf`/`-conf`/`all (n)` | resize current column (or all) |
| `fit` | `active`/`visible`/`all`/`toend`/`tobeg`/`expand` | fit operation |
| `fit_into_view` | — | fit the active column fully into view |
| `focus` | direction | focus + center layout, wrapping (no monitor jump) |
| `promote` | — | move window to its own new column |
| `swapcol` | `l`/`r` | swap current column with neighbor (wraps) |
| `inhibit_scroll` | `[bool]` | prevent the view from moving for this workspace (toggle/explicit) |
| `expel` | — | move current window to a dedicated column |
| `consume` | — | move current window into the previous column |
| `consume_or_expel` | `prev`/`next` | expel if not alone, consume if alone |

```lua
hl.bind(mainMod .. " + period", hl.dsp.layout("move +col"))
hl.bind(mainMod .. " + comma",  hl.dsp.layout("swapcol l"))
hl.bind(mainMod .. " + bracketleft",  hl.dsp.layout("colresize -0.1"))
```

### Scrolling specifics
- **Layout-handled fullscreen**: scrolling has its own FS handler that lets you
  scroll away from a fullscreened window without un-fullscreening it. Use
  `hl.dsp.window.fullscreen({ layout_aware = true })` (the default).
- Static window rule `scrolling_width = 0.5` sets a starting column width.
- Workspace rule `layout_opts = { direction = "right" }`.

## Monocle

Windows always take the full available space. No config options.

### Monocle layout messages
| message | description |
| --- | --- |
| `cyclenext` | next window |
| `cycleprev` | previous window |

> `hl.dsp.window.cycle_next()` does NOT work in monocle. Use
> `hl.dsp.layout("cyclenext")` or `hl.dsp.window.cycle_next({ tiled = true })`.

## Custom (Lua) layouts — `lua:NAME`

Register with `hl.layout.register(name, { recalculate, layout_msg? })` and select
as `general.layout = "lua:name"` or `hl.workspace_rule({ layout = "lua:name" })`.
See Reference 08 for the full `ctx`/target API and example layouts.

## Layout messages — note on the old `layoutmsg`

The 0.54 dispatcher `layoutmsg` maps directly to `hl.dsp.layout(string)`. The
message strings themselves (e.g. `togglesplit`, `cyclenext`, `mfact +0.1`,
`movetoroot active unstable`) are **unchanged** — only the call site changed:

```ini
# 0.54
bind = SUPER, A, layoutmsg, togglesplit
bind = SUPER, J, layoutmsg, cyclenext
```
```lua
-- 0.55
hl.bind("SUPER + A", hl.dsp.layout("togglesplit"))
hl.bind("SUPER + J", hl.dsp.layout("cyclenext"))
```
`splitratio` (was its own dispatcher) is now also `hl.dsp.layout("splitratio 0.5")`.
