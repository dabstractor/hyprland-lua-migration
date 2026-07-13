# Reference 04 — Dispatchers (`hl.dsp.*`)

> Companion to `SKILL.md`. Read this for the exhaustive mapping of every old
> dispatcher name to its new `hl.dsp.*` form, with parameters.

## Mental model

In 0.54 a dispatcher was a string + comma-separated params. In 0.55 dispatchers
are **methods on the `hl.dsp` table that return a dispatcher object**. You either
pass that object to `hl.bind()`, or execute it immediately with `hl.dispatch()`.

```ini
# 0.54
bind = SUPER, left, movefocus, l
bind = SUPER, 1, workspace, 1
```
```lua
-- 0.55
hl.bind("SUPER + left",  hl.dsp.focus({ direction = "left" }))
hl.bind("SUPER + 1",     hl.dsp.focus({ workspace = 1 }))
```

The new API groups dispatchers by namespace: `hl.dsp.*` (general),
`hl.dsp.window.*`, `hl.dsp.workspace.*`, `hl.dsp.group.*`, `hl.dsp.cursor.*`.

## General dispatchers (`hl.dsp.*`)

| 0.54 dispatcher | 0.55 method | params (table) |
| --- | --- | --- |
| `exec` | `hl.dsp.exec_cmd(cmd, rules?)` | 2nd arg optional rules table — see Executing-with-rules |
| `execr` | `hl.dsp.exec_raw(cmd)` | runs without shell |
| `exit` | `hl.dsp.exit()` | quit Hyprland (prefer `hyprshutdown`/`uwsm stop`) |
| `submap` | `hl.dsp.submap(name)` | `"reset"` returns to global |
| `pass` | `hl.dsp.pass({ window })` | pass shortcut to a window |
| (new) | `hl.dsp.send_shortcut({ mods, key, window? })` | was `sendshortcut` |
| (new) | `hl.dsp.send_key_state({ mods, key, state, window? })` | `state` = `"down"`/`"up"`/`"repeat"` |
| `layoutmsg` | `hl.dsp.layout(msg)` | layout message string — full list in Reference 09 |
| `dpms` | `hl.dsp.dpms({ action?, monitor? })` | `action` = `"on"`/`"off"`/`"toggle"` |
| (new) | `hl.dsp.event(string)` | send an event to socket2 |
| `global` | `hl.dsp.global(name)` | dbus global shortcut |
| (new) | `hl.dsp.force_idle(seconds)` | set idle timers; **do not bind directly** |
| (new) | `hl.dsp.no_op()` | does nothing (useful for conditional binds) |
| `forcerendererreload` | `hl.dsp.force_renderer_reload()` | |
| `movefocus` | `hl.dsp.focus({ direction })` | direction = `"left"`/`"right"`/`"up"`/`"down"` |
| `focusmonitor` | `hl.dsp.focus({ monitor })` | monitor selector |
| `workspace` | `hl.dsp.focus({ workspace, on_current_monitor? })` | see Workspace selectors |
| `focuswindow` | `hl.dsp.focus({ window })` | window selector |
| `focusurgentorlast` | `hl.dsp.focus({ urgent_or_last = true })` | |
| `focuscurrentorlast` | `hl.dsp.focus({ last = true })` | |
| `focusworkspaceoncurrentmonitor` | `hl.dsp.focus({ workspace = W, on_current_monitor = true })` | XMonad-style switching |

> `force_idle`/`dpms` should not be bound directly (undefined behavior). Wrap in
> a timer: `hl.bind(K, function() hl.timer(function() hl.dispatch(hl.dsp.dpms({action="disable"})) end, {timeout=500, type="oneshot"}) end)`.

## Window dispatchers (`hl.dsp.window.*`)

`window` selectors are strings like `"class:..."`, `"title:..."`, `"pid:..."`,
`"address:0x..."`, `"floating"`, `"tiled"`, `"activewindow"`. Omit to use the
active window. `action` for toggles is `"toggle"` (default), `"set"`/`"on"`, `"unset"`/`"off"`.

| 0.54 dispatcher | 0.55 method | notes |
| --- | --- | --- |
| `killactive` | `hl.dsp.window.close()` | graceful close of active window |
| `closewindow` | `hl.dsp.window.close({ window })` | |
| (new) | `hl.dsp.window.kill({ window? })` | SIGKILL the owning process |
| (new) | `hl.dsp.window.signal({ signal, window? })` | send a POSIX signal |
| `togglefloating` | `hl.dsp.window.float({ action?, window? })` | |
| `fullscreen` | `hl.dsp.window.fullscreen({ mode?, action?, layout_aware?, window? })` | `mode` `"maximized"`/`"fullscreen"`; `action` `"toggle"`/`"set"`/`"unset"` |
| `fakefullscreen` | `hl.dsp.window.fullscreen_state({ internal = 2, client = 0 })` | decoupled internal/ client state |
| (new) | `hl.dsp.window.fullscreen_state({ internal, client, action?, layout_aware?, window? })` | values -1..3; see Fullscreenstate |
| `pseudo` | `hl.dsp.window.pseudo({ action?, window? })` | |
| `movewindow` (direction) | `hl.dsp.window.move({ direction, group_aware?, window? })` | direction string |
| `movewindow` (to monitor) | `hl.dsp.window.move({ monitor, follow?, window? })` | |
| `movetoworkspace` | `hl.dsp.window.move({ workspace, follow?, window? })` | `follow=false` == old `movetoworkspacesilent` |
| `movetoworkspacesilent` | `hl.dsp.window.move({ workspace, follow = false, window? })` | |
| `moveactive` / `movewindowpixel` | `hl.dsp.window.move({ x, y, relative?, window? })` | `relative=true` for delta |
| `moveintogroup` | `hl.dsp.window.move({ into_group = direction, window? })` | |
| `moveoutofgroup` | `hl.dsp.window.move({ out_of_group = true })` or `{ out_of_group = direction }` | |
| `movewindoworgroup` | `hl.dsp.window.move({ direction, group_aware = true, window? })` | behaves as moveinto/outof/move |
| (new) | `hl.dsp.window.move({ into_or_create_group = direction, window? })` | create group if none |
| `swapwindow` | `hl.dsp.window.swap({ direction })` | |
| (new) | `hl.dsp.window.swap({ target })` / `{ with }` / `{ other }` | swap with a specific window selector |
| `swapnext` | `hl.dsp.window.swap({ next = true })` | |
| (new) | `hl.dsp.window.swap({ prev = true })` | |
| `centerwindow` | `hl.dsp.window.center({ window? })` | floats only |
| `cyclenext` | `hl.dsp.window.cycle_next({ next?, tiled?, floating?, window? })` | `next` default true |
| `pin` | `hl.dsp.window.pin({ action?, window? })` | floats only |
| `bringactivetotop` | `hl.dsp.window.bring_to_top()` | deprecated alias; prefer alter_zorder |
| `alterzorder` | `hl.dsp.window.alter_zorder({ mode, window? })` | mode `"top"`/`"bottom"` |
| `resizeactive` | `hl.dsp.window.resize({ x, y, relative?, window? })` | |
| `resizewindowpixel` | `hl.dsp.window.resize({ x, y, relative?, window? })` | include `window` |
| `movewindoworgroup` resize | `hl.dsp.window.resize({ keep_aspect_ratio })` | mouse bind |
| (new) | `hl.dsp.window.tag({ tag, window? })` | tag a window (`"+x"`/`"-x"`/`"x"`) |
| (new) | `hl.dsp.window.clear_tags({ window? })` | |
| (new) | `hl.dsp.window.toggle_swallow()` | toggle swallowed windows visible |
| (new) | `hl.dsp.window.set_prop({ prop, value, window? })` | set any dynamic window-rule prop |
| `denywindowfromgroup` | `hl.dsp.window.deny_from_group({ action? })` | |
| (mouse) `movewindow` | `hl.dsp.window.drag()` | for mouse binds |
| (mouse) `resizewindow` | `hl.dsp.window.resize()` | for mouse binds; `{keep_aspect_ratio=true}` |

> `set_prop` props = any dynamic window-rule effect (e.g. `no_anim`, `opacity`,
> `border_color` which expands to `active_border_color`/`inactive_border_color`,
> `opacity` → `opacity`/`opacity_inactive`/`opacity_fullscreen`/`*_override`).
> Example: `hl.dsp.window.set_prop({ prop = "no_anim", value = "1" })`.

## Workspace dispatchers (`hl.dsp.workspace.*`)

| 0.54 dispatcher | 0.55 method | notes |
| --- | --- | --- |
| `renameworkspace` | `hl.dsp.workspace.rename({ workspace, name? })` | |
| (new) | `hl.dsp.workspace.change_id({ workspace, id })` | change a workspace's ID |
| `moveworkspacetomonitor` | `hl.dsp.workspace.move({ workspace, monitor })` | |
| `movecurrentworkspacetomonitor` | `hl.dsp.workspace.move({ monitor })` | moves the active workspace |
| `swapactiveworkspaces` | `hl.dsp.workspace.swap_monitors({ monitor1, monitor2 })` | |
| `togglespecialworkspace` | `hl.dsp.workspace.toggle_special(name)` | `""` = first/default special |

## Group dispatchers (`hl.dsp.group.*`)

| 0.54 dispatcher | 0.55 method | notes |
| --- | --- | --- |
| `togglegroup` | `hl.dsp.group.toggle({ window? })` | |
| `changegroupactive` | `hl.dsp.group.next({ window? })` / `group.prev({ window? })` / `group.active({ index, window? })` | `index` from 1 |
| `movegroupwindow` | `hl.dsp.group.move_window({ forward? })` | `forward` default true |
| `lockgroups` | `hl.dsp.group.lock({ action? })` | global group lock |
| `lockactivegroup` | `hl.dsp.group.lock_active({ action? })` | lock the focused group |

`setignoregrouplock` (old) — there is no dedicated dispatcher; set
`binds.ignore_group_lock` in config instead (the into/out-of group moves respect it).

## Cursor dispatchers (`hl.dsp.cursor.*`)

| 0.54 dispatcher | 0.55 method | notes |
| --- | --- | --- |
| `movecursortocorner` | `hl.dsp.cursor.move_to_corner({ corner, window? })` | corner 0-3 |
| `movecursor` | `hl.dsp.cursor.move({ x, y })` | |

## Workspace selectors

Used wherever a workspace is accepted (as a string or number):

- ID: `1`, `2`
- Relative ID: `+1`, `-3`
- On-monitor relative/absolute: `m+1`, `m-2`, `m~3`
- On-monitor incl. empty: `r+1`, `r~3`
- Open workspace relative/absolute: `e+1`, `e-10`, `e~2`
- Name: `name:Web`
- Previous: `previous`, `previous_per_monitor`
- First empty: `empty` (suffix `m` monitor-only, `n` next) e.g. `emptynm`
- Special: `special`, `special:name`

Numeric IDs must be 1..2147483647. `0` and negatives are not allowed.

## Window selectors

`class:...`, `initialclass:...`, `title:...`, `initialtitle:...`, `tag:...`,
`pid:...`, `stableid:...`, `address:0x...`, `activewindow`, `floating`, `tiled`.

## Monitor selectors

monitor object, ID, direction, name, `desc:`+description, `current`, relative (`+1`/`-2`).

## Fullscreenstate (decoupled internal/ client)

`fullscreen_state({ internal, client, action?, layout_aware? })`. Values: `-1`
current, `0` none, `1` maximized, `2` fullscreen.

Examples:
- `{ internal = 2, client = 0 }` — fullscreen the window but keep the client
  thinking it's windowed (prevents Chromium presentation mode).
- `{ internal = 0, client = 2 }` — window stays windowed but client goes fullscreen.

Some layouts (scrolling) have their own fullscreen handler; use `layout_aware = true`
(the default) to opt into it.

## Executing with rules

`hl.dsp.exec_cmd(cmd, rules)` — the 2nd arg is a table of window-rule effects
applied to the spawned window (records the PID):
```ini
# 0.54
bind = SUPER, E, exec, [workspace 2 silent; float; noanim] kitty
```
```lua
-- 0.55
hl.bind("SUPER + E", hl.dsp.exec_cmd("kitty", { workspace = "2 silent", float = true, no_anim = true }))
```
(Works by tracking the spawned PID; forking processes that open the window later won't match.)

## Layout messages (`hl.dsp.layout(string)`)

Layout-specific messages (dwindle/master/scrolling/monocle) are passed as a
string, exactly as before in spirit:
```lua
hl.bind("SUPER + A", hl.dsp.layout("togglesplit"))     -- dwindle
hl.bind("SUPER + J", hl.dsp.layout("cyclenext"))       -- master/monocle
hl.bind(mainMod .. " + period", hl.dsp.layout("move +col"))  -- scrolling
```
See **Reference 09 — Layouts** for the complete per-layout message tables
(dwindle, master, scrolling, monocle).

## Running a dispatcher from a script/REPL

```sh
hyprctl dispatch 'hl.dsp.focus({ workspace = "3" })'
hyprctl eval     'hl.dispatch(hl.dsp.window.close())'
```
Inside Lua itself use `hl.dispatch(obj)`:
```lua
hl.dispatch(hl.dsp.window.float({ action = "toggle" }))
```
