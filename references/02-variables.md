# Reference 02 — All Variables (`hl.config`)

> Companion to `SKILL.md`. Read this for the exhaustive list of every config
> option, its old→new name, type, and default. **Bold** rows are moved, renamed,
> inverted, or removed.

All options are set inside `hl.config({...})`. See Reference 01 for the type
system (bool is now strictly `true`/`false`; strings quoted; vec2 is a table).

Legend: **MOVED** (changed section), **RENAMED**, **INVERTED** (sense flipped),
**REMOVED** (no direct equivalent — see note), **NEW** (added in 0.55).

---

## General

| 0.54 name | 0.55 name | type | default | notes |
| --- | --- | --- | --- | --- |
| `general.border_size` | `general.border_size` | int | `1` | |
| `general.gaps_in` | `general.gaps_in` | css_gaps | `5` | now also table per-side |
| `general.gaps_out` | `general.gaps_out` | css_gaps | `20` | now also table per-side |
| — | `general.float_gaps` **NEW** | css_gaps | `0` | gaps for floating windows |
| `general.gaps_workspaces` | `general.gaps_workspaces` | int | `0` | |
| `general.col.inactive_border` | `general.col.inactive_border` | gradient | `0xff444444` | |
| `general.col.active_border` | `general.col.active_border` | gradient | `0xffffffff` | |
| `general.col.nogroup_border` | `general.col.nogroup_border` | gradient | `0xffffaaff` | |
| `general.col.nogroup_border_active` | `general.col.nogroup_border_active` | gradient | `0xffff00ff` | |
| `general.layout` | `general.layout` | str | `"dwindle"` | now also `"scrolling"`/`"monocle"` |
| `general.no_focus_fallback` | `general.no_focus_fallback` | bool | `false` | |
| `general.resize_on_border` | `general.resize_on_border` | bool | `false` | |
| `general.extend_border_grab_area` | `general.extend_border_grab_area` | int | `15` | |
| `general.hover_icon_on_border` | `general.hover_icon_on_border` | bool | `true` | |
| `general.allow_tearing` | `general.allow_tearing` | bool | `false` | |
| — | `general.resize_corner` **NEW** | int | `0` | force a resize corner for floats (1-4) |
| — | `general.modal_parent_blocking` **NEW** | bool | `true` | |
| — | `general.locale` **NEW** | str | `""` | override system locale |
| `general.sensitivity` | **REMOVED** | — | — | use `input.sensitivity` |
| `general.no_border_on_floating` | **REMOVED** | — | — | use window rules / `border_size` |
| `general.cursor_inactive_timeout` | `cursor.inactive_timeout` **MOVED** | float | `0` | |
| `general.no_cursor_warps` | `cursor.no_warps` **MOVED** | bool | `false` | |
| `general.apply_sens_to_raw` | **REMOVED** | — | — | |

### `general.snap` (NEW subsection)

| name | type | default | description |
| --- | --- | --- | --- |
| `enabled` | bool | `false` | snapping for floating windows |
| `window_gap` | int | `10` | min gap before snapping |
| `monitor_gap` | int | `10` | min gap to monitor edge |
| `border_overlap` | bool | `false` | one border-width between windows |
| `respect_gaps` | bool | `false` | respect `general.gaps_in` |

---

## Decoration

| 0.54 name | 0.55 name | type | default |
| --- | --- | --- | --- |
| `decoration.rounding` | `decoration.rounding` | int | `0` |
| `decoration.rounding_power` | `decoration.rounding_power` | float | `2.0` |
| `decoration.active_opacity` | `decoration.active_opacity` | float | `1.0` |
| `decoration.inactive_opacity` | `decoration.inactive_opacity` | float | `1.0` |
| `decoration.fullscreen_opacity` | `decoration.fullscreen_opacity` | float | `1.0` |
| `decoration.dim_modal` | `decoration.dim_modal` | bool | `true` |
| `decoration.dim_inactive` | `decoration.dim_inactive` | bool | `false` |
| `decoration.dim_strength` | `decoration.dim_strength` | float | `0.5` |
| `decoration.dim_special` | `decoration.dim_special` | float | `0.2` |
| `decoration.dim_around` | `decoration.dim_around` | float | `0.4` |
| `decoration.screen_shader` | `decoration.screen_shader` | str | `""` |
| — | `decoration.border_part_of_window` **NEW** | bool | `true` |

### `decoration.blur`

| name | type | default | notes |
| --- | --- | --- | --- |
| `enabled` | bool | `true` | |
| `size` | int | `8` | ≥1 |
| `passes` | int | `1` | ≥1 |
| `ignore_opacity` | bool | `true` | |
| `new_optimizations` | bool | `true` | |
| `xray` | bool | `false` | only with new_optimizations |
| `noise` | float | `0.0117` | |
| `contrast` | float | `0.8916` | |
| `brightness` | float | `1.0` | |
| `vibrancy` | float | `0.1696` | |
| `vibrancy_darkness` | float | `0.0` | |
| `special` | bool | `false` | blur behind special workspace (expensive) |
| `popups` | bool | `false` | |
| `popups_ignorealpha` | float | `0.2` | |
| `input_methods` | bool | `false` | e.g. fcitx5 |
| `input_methods_ignorealpha` | float | `0.2` | |

### `decoration.shadow` (was already a subsection in 0.54)

| name | type | default |
| --- | --- | --- |
| `enabled` | bool | `true` |
| `range` | int | `4` |
| `render_power` | int | `3` |
| `sharp` | bool | `false` |
| `color` | gradient | `0xee1a1a1a` |
| `color_inactive` | gradient | unset |
| `offset` | vec2 | `{0,0}` |
| `scale` | float | `1.0` |

### `decoration.glow` (NEW subsection)

| name | type | default |
| --- | --- | --- |
| `enabled` | bool | `false` |
| `range` | int | `10` |
| `render_power` | int | `3` |
| `color` | gradient | `0xee33ccff` |
| `color_inactive` | gradient | unset |

### `decoration.motion_blur` (NEW subsection)

| name | type | default |
| --- | --- | --- |
| `enabled` | bool | `false` |
| `samples` | int | `7` |

---

## Animations (enable flag here; curves/animations in `hl.curve`/`hl.animation`)

| name | type | default |
| --- | --- | --- |
| `animations.enabled` | bool | `true` |
| `animations.workspace_wraparound` **NEW** | bool | `false` |

> Defining curves & per-leaf animation settings is no longer done inside
> `animations {}`. Use `hl.curve()` and `hl.animation()` — see Reference 07.

---

## Input

| 0.54 name | 0.55 name | type | default |
| --- | --- | --- | --- |
| `input.kb_model` | `input.kb_model` | str | `""` |
| `input.kb_layout` | `input.kb_layout` | str | `"us"` |
| `input.kb_variant` | `input.kb_variant` | str | `""` |
| `input.kb_options` | `input.kb_options` | str | `""` |
| `input.kb_rules` | `input.kb_rules` | str | `""` |
| `input.kb_file` | `input.kb_file` | str | `""` |
| `input.numlock_by_default` | `input.numlock_by_default` | bool | `false` |
| `input.resolve_binds_by_sym` | `input.resolve_binds_by_sym` | bool | `false` |
| `input.repeat_rate` | `input.repeat_rate` | int | `25` |
| `input.repeat_delay` | `input.repeat_delay` | int | `600` |
| `input.sensitivity` | `input.sensitivity` | float | `0.0` |
| `input.accel_profile` | `input.accel_profile` | str | `""` |
| `input.force_no_accel` | `input.force_no_accel` | bool | `false` |
| `input.rotation` | `input.rotation` | int | `0` |
| `input.left_handed` | `input.left_handed` | bool | `false` |
| `input.scroll_points` | `input.scroll_points` | str | `""` |
| `input.scroll_method` | `input.scroll_method` | str | `""` |
| `input.scroll_button` | `input.scroll_button` | int | `0` |
| `input.scroll_button_lock` | `input.scroll_button_lock` | bool | `false` |
| `input.scroll_factor` | `input.scroll_factor` | float | `1.0` |
| `input.natural_scroll` | `input.natural_scroll` | bool | `false` |
| `input.follow_mouse` | `input.follow_mouse` | int | `1` |
| — | `input.follow_mouse_shrink` **NEW** | int | `0` |
| — | `input.follow_mouse_threshold` **NEW** | float | `0.0` |
| — | `input.focus_on_close` **NEW** | int | `0` |
| `input.mouse_refocus` | `input.mouse_refocus` | bool | `true` |
| `input.float_switch_override_focus` | `input.float_switch_override_focus` | int | `1` |
| `input.special_fallthrough` | `input.special_fallthrough` | bool | `false` |
| — | `input.off_window_axis_events` **NEW** | int | `1` |
| — | `input.emulate_discrete_scroll` **NEW** | int | `1` |

### `input.touchpad`

| name | type | default |
| --- | --- | --- |
| `disable_while_typing` | bool | `true` |
| `natural_scroll` | bool | `false` |
| `scroll_factor` | float | `1.0` |
| `middle_button_emulation` | bool | `false` |
| `tap_button_map` | str | `""` (`"lrm"`/`"lmr"`) |
| `clickfinger_behavior` | bool | `false` |
| `tap_to_click` | bool | `true` |
| `drag_lock` | int | `0` (0/1/2) |
| `tap_and_drag` | bool | `true` |
| `flip_x` **NEW** | bool | `false` |
| `flip_y` **NEW** | bool | `false` |
| `drag_3fg` **NEW** | int | `0` (0/1/2) |

### `input.touchdevice`

| name | type | default |
| --- | --- | --- |
| `transform` | int | `0` |
| `output` | str | auto |
| `enabled` | bool | `true` |

### `input.virtualkeyboard` (NEW)

| name | type | default |
| --- | --- | --- |
| `share_states` | int | `2` |
| `release_pressed_on_close` | bool | `false` |

### `input.tablet`

| name | type | default |
| --- | --- | --- |
| `transform` | int | `0` |
| `output` | str | `""` |
| `region_position` | vec2 | `{0,0}` |
| `absolute_region_position` | bool | `false` |
| `region_size` | vec2 | `{0,0}` |
| `relative_input` | bool | `false` |
| `left_handed` | bool | `false` |
| `active_area_size` | vec2 | `{0,0}` |
| `active_area_position` | vec2 | `{0,0}` |

### `input.tablettool` (NEW)

| name | type | default |
| --- | --- | --- |
| `eraser_button_mode` | int | `0` |
| `eraser_button_override` | int | `0` |
| `pressure_range_min` | float | `-1.0` |
| `pressure_range_max` | float | `-1.0` |

---

## Gestures

**MAJOR CHANGE:** The old `workspace_swipe*` enable/finger/count options are
**REMOVED**. Workspace swiping is now configured declaratively with
`hl.gesture({...})` (see Reference 07). The remaining tuning knobs:

| 0.54 name | 0.55 name | type | default |
| --- | --- | --- | --- |
| `gestures.workspace_swipe_distance` | `gestures.workspace_swipe_distance` | int | `300` |
| `gestures.workspace_swipe_touch` | `gestures.workspace_swipe_touch` | bool | `false` |
| `gestures.workspace_swipe_invert` | `gestures.workspace_swipe_invert` | bool | `true` |
| — | `gestures.workspace_swipe_touch_invert` **NEW** | bool | `false` |
| `gestures.workspace_swipe_min_speed_to_force` | `gestures.workspace_swipe_min_speed_to_force` | int | `30` |
| `gestures.workspace_swipe_cancel_ratio` | `gestures.workspace_swipe_cancel_ratio` | float | `0.5` |
| `gestures.workspace_swipe_create_new` | `gestures.workspace_swipe_create_new` | bool | `true` |
| `gestures.workspace_swipe_direction_lock` | `gestures.workspace_swipe_direction_lock` | bool | `true` |
| `gestures.workspace_swipe_direction_lock_threshold` | `gestures.workspace_swipe_direction_lock_threshold` | int | `10` |
| `gestures.workspace_swipe_forever` | `gestures.workspace_swipe_forever` | bool | `false` |
| `gestures.workspace_swipe_use_r` | `gestures.workspace_swipe_use_r` | bool | `false` |
| — | `gestures.close_max_timeout` **NEW** | int | `1000` |
| `gestures.workspace_swipe` | **REMOVED** | — | — | use `hl.gesture({action="workspace"})` |
| `gestures.workspace_swipe_fingers` | **REMOVED** | — | — | use `hl.gesture({fingers=3,...})` |
| `gestures.workspace_swipe_min_fingers` | **REMOVED** | — | — | |
| `gestures.workspace_swipe_numbered` | **REMOVED** | — | — | |

### `gestures.scrolling` (NEW)

| name | type | default |
| --- | --- | --- |
| `move_snap_to_grid` | bool | `true` |
| `move_snap_cursor` | bool | `true` |

> To restore the classic 3-finger workspace swipe:
> ```lua
> hl.gesture({ fingers = 3, direction = "horizontal", action = "workspace" })
> ```

---

## Group

| name | type | default |
| --- | --- | --- |
| `group.auto_group` | bool | `true` |
| `group.insert_after_current` | bool | `true` |
| `group.focus_removed_window` | bool | `true` |
| `group.drag_into_group` | int | `1` (0/1/2) |
| `group.merge_groups_on_drag` | bool | `true` |
| `group.merge_groups_on_groupbar` | bool | `true` |
| `group.merge_floated_into_tiled_on_groupbar` | bool | `false` |
| `group.group_on_movetoworkspace` | bool | `false` |
| `group.col.border_active` | gradient | `0x66ffff00` |
| `group.col.border_inactive` | gradient | `0x66777700` |
| `group.col.border_locked_active` | gradient | `0x66ff5500` |
| `group.col.border_locked_inactive` | gradient | `0x66775500` |

### `group.groupbar`

| name | type | default |
| --- | --- | --- |
| `enabled` | bool | `true` |
| `font_family` | str | `""` |
| `font_size` | int | `8` |
| `font_weight_active`/`font_weight_inactive` | font_weight | `"normal"` |
| `gradients` | bool | `false` |
| `height` | int | `14` |
| `indicator_gap` | int | `0` |
| `indicator_height` | int | `3` |
| `stacked` | bool | `false` |
| `priority` | int | `3` |
| `render_titles` | bool | `true` |
| `text_offset` | int | `0` |
| `text_padding` **NEW** | int | `0` |
| `scrolling` | bool | `true` |
| `rounding` | int | `1` |
| `rounding_power` | float | `2.0` |
| `gradient_rounding` | int | `2` |
| `gradient_rounding_power` | float | `2.0` |
| `round_only_edges` | bool | `true` |
| `gradient_round_only_edges` | bool | `true` |
| `text_color` | color | `0xffffffff` |
| `text_color_inactive` **NEW** | color | unset |
| `text_color_locked_active` **NEW** | color | unset |
| `text_color_locked_inactive` **NEW** | color | unset |
| `col.active` | gradient | `0x66ffff00` |
| `col.inactive` | gradient | `0x66777700` |
| `col.locked_active` | gradient | `0x66ff5500` |
| `col.locked_inactive` | gradient | `0x66775500` |
| `gaps_in` | int | `2` |
| `gaps_out` | int | `2` |
| `keep_upper_gap` | bool | `true` |
| `middle_click_close` | bool | `true` |
| `blur` **NEW** | bool | `false` |

`font_weight` presets: `"thin"` `"ultralight"` `"light"` `"semilight"` `"book"`
`"normal"` `"medium"` `"semibold"` `"bold"` `"ultrabold"` `"heavy"` `"ultraheavy"`
or an int 100–1000.

---

## Misc

| 0.54 name | 0.55 name | type | default |
| --- | --- | --- | --- |
| `misc.disable_hyprland_logo` | `misc.disable_hyprland_logo` | bool | `false` |
| `misc.disable_splash_rendering` | `misc.disable_splash_rendering` | bool | `false` |
| — | `misc.disable_scale_notification` **NEW** | bool | `false` |
| `misc.col.splash` | `misc.col.splash` | color | `0x55ffffff` |
| `misc.font_family` | `misc.font_family` | str | `"Sans"` |
| `misc.splash_font_family` | `misc.splash_font_family` | str | `""` |
| `misc.force_default_wallpaper` | `misc.force_default_wallpaper` | int | `-1` |
| `misc.vrr` | `misc.vrr` | int | `0` (0/1/2/3) |
| `misc.mouse_move_enables_dpms` | `misc.mouse_move_enables_dpms` | bool | `false` |
| `misc.key_press_enables_dpms` | `misc.key_press_enables_dpms` | bool | `false` |
| — | `misc.name_vk_after_proc` **NEW** | bool | `true` |
| `misc.always_follow_on_dnd` | `misc.always_follow_on_dnd` | bool | `true` |
| `misc.layers_hog_keyboard_focus` | `misc.layers_hog_keyboard_focus` | bool | `true` |
| `misc.animate_manual_resizes` | `misc.animate_manual_resizes` | bool | `false` |
| `misc.animate_mouse_windowdragging` | `misc.animate_mouse_windowdragging` | bool | `false` |
| `misc.disable_autoreload` | `misc.disable_autoreload` | bool | `false` |
| `misc.enable_swallow` | `misc.enable_swallow` | bool | `false` |
| `misc.swallow_regex` | `misc.swallow_regex` | str | `""` |
| `misc.swallow_exception_regex` | `misc.swallow_exception_regex` | str | `""` |
| `misc.focus_on_activate` | `misc.focus_on_activate` | bool | `false` |
| `misc.mouse_move_focuses_monitor` | `misc.mouse_move_focuses_monitor` | bool | `true` |
| `misc.allow_session_lock_restore` | `misc.allow_session_lock_restore` | bool | `false` |
| `misc.session_lock_xray` | `misc.session_lock_xray` | bool | `false` |
| `misc.background_color` | `misc.background_color` | color | `0x111111` |
| `misc.close_special_on_empty` | `misc.close_special_on_empty` | bool | `true` |
| — | `misc.on_focus_under_fullscreen` **NEW** | int | `2` |
| — | `misc.exit_window_retains_fullscreen` **NEW** | bool | `false` |
| `misc.initial_workspace_tracking` | `misc.initial_workspace_tracking` | int | `1` |
| `misc.middle_click_paste` | `misc.middle_click_paste` | bool | `true` |
| — | `misc.render_unfocused_fps` **NEW** | int | `15` |
| — | `misc.disable_xdg_env_checks` **NEW** | bool | `false` |
| — | `misc.disable_hyprland_guiutils_check` **NEW** | bool | `false` |
| — | `misc.lockdead_screen_delay` **NEW** | int | `1000` |
| — | `misc.enable_anr_dialog` **NEW** | bool | `true` |
| — | `misc.anr_missed_pings` **NEW** | int | `5` |
| — | `misc.size_limits_tiled` **NEW** | bool | `false` |
| — | `misc.screencopy_force_8b` **NEW** | bool | `true` |
| — | `misc.disable_watchdog_warning` **NEW** | bool | `false` |
| `misc.vfr` | `debug.vfr` **MOVED** | bool | `true` |
| `misc.no_direct_scanout` | `render.direct_scanout` **INVERTED+MOVED** | int | `0` (0=off,1=on,2=auto) |

---

## Layout (NEW top-level section)

| name | type | default |
| --- | --- | --- |
| `layout.single_window_aspect_ratio` | vec2 | `{0,0}` |
| `layout.single_window_aspect_ratio_tolerance` | int | `0.1` |

---

## Binds

| name | type | default |
| --- | --- | --- |
| `binds.pass_mouse_when_bound` | bool | `false` |
| `binds.scroll_event_delay` | int | `300` |
| `binds.workspace_back_and_forth` | bool | `false` |
| `binds.hide_special_on_workspace_change` | bool | `false` |
| `binds.allow_workspace_cycles` | bool | `false` |
| `binds.workspace_center_on` | int | `1` |
| `binds.focus_preferred_method` | int | `0` |
| `binds.ignore_group_lock` | bool | `false` |
| `binds.movefocus_cycles_fullscreen` | bool | `false` |
| — | `binds.movefocus_cycles_groupfirst` **NEW** | bool | `false` |
| `binds.window_direction_monitor_fallback` | bool | `true` |
| `binds.disable_keybind_grabbing` | bool | `false` |
| `binds.allow_pin_fullscreen` | bool | `false` |
| — | `binds.drag_threshold` **NEW** | int | `0` |

---

## XWayland / OpenGL / Render / Cursor / Ecosystem / Quirks / Debug / Experimental

These were mostly `misc.*` or didn't exist before. Full 0.55 list:

### xwayland
`enabled` (bool, `true`), `use_nearest_neighbor` (bool, `true`),
`force_zero_scaling` (bool, `false`), `create_abstract_socket` **NEW** (bool, `false`).

### opengl
`nvidia_anti_flicker` (bool, `true`).

### render
`direct_scanout` (int, `0`), `expand_undersized_textures` (bool, `true`),
`xp_mode` **NEW** (bool, `false`), `ctm_animation` (int, `2`), `cm_enabled`
(bool, `true`), `send_content_type` (bool, `true`), `cm_auto_hdr` (int, `1`),
`new_render_scheduling` **NEW** (bool, `false`), `non_shader_cm` (int, `3`),
`non_shader_cm_interop` (int, `2`), `cm_sdr_eotf` (str, `"default"`),
`commit_timing_enabled` (bool, `true`), `use_fp16` (int, `2`),
`keep_unmodified_copy` (int, `2`), `use_shader_blur_blend` (bool, `false`),
`icc_vcgt_enabled` (bool, `true`), `fp16_sdr_tf` (int, `0`).

### cursor (was `general:*` in 0.54)
`invisible`, `sync_gsettings_theme`, `no_hardware_cursors` (int 0/1/2),
`no_break_fs_vrr` (int 0/1/2), `min_refresh_rate`, `hotspot_padding`,
`inactive_timeout`, `no_warps` (was `general.no_cursor_warps`),
`persistent_warps`, `warp_on_change_workspace`, `warp_on_toggle_special`,
`default_monitor`, `zoom_factor`, `zoom_rigid`, `zoom_detached_camera`,
`enable_hyprcursor`, `hide_on_key_press`, `hide_on_touch`, `hide_on_tablet`,
`use_cpu_buffer`, `warp_back_after_non_mouse_input`, `zoom_disable_aa`.

### ecosystem
`no_update_news`, `no_donation_nag`, `enforce_permissions`.

### quirks (NEW)
`prefer_hdr` (int 0/1/2), `skip_non_kms_dmabuf_formats` (bool).

### debug (developer only)
`overlay`, `damage_blink`, `gl_debugging`, `vfr` (moved from misc), `disable_logs`,
`disable_time`, `damage_tracking`, `enable_stdout_logs`, `manual_crash`,
`suppress_errors`, `log_damage`, `disable_scale_checks`, `error_limit`,
`error_position`, `colored_stdout_logs`, `pass`, `full_cm_proto`,
`ds_handle_same_buffer`, `ds_handle_same_buffer_fifo`, `fifo_pending_workaround`,
`render_solitary_wo_damage`, `invalidate_fp16`.

### experimental
`wp_cm_1_2` (bool, `false`).
