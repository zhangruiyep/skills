---
name: "lvgl-v8-sifli-dev"
description: "LVGL V8 GUI development on SiFli Solution 2.x (SF32LB52x/55x/56x/58x). Invoke when: (1) project uses SiFli Solution 2.x AND LVGL config is V8.x simultaneously, or (2) user explicitly requests LVGL V8 development. Covers generating/modifying LVGL UI code, watch faces, settings pages, or any LVGL-based GUI application. | LVGL V8 在 SiFli Solution 2.x 平台上的 GUI 开发：当项目同时满足 SiFli Solution 2.x 与 LVGL V8，或用户明确要求 LVGL V8 开发时触发；适用于生成/修改 LVGL UI 代码、表盘、设置页或任何 LVGL GUI 应用。"
---

# LVGL V8 Development Guide for SiFli Solution 2.x

This skill provides conventions, patterns, and APIs for developing LVGL V8 GUI applications on SiFli Solution 2.x platform based on RT-Thread RTOS.

## Invocation Conditions / 生效条件

- **Automatic (条件 A)** — both must hold / 同时满足：
  1. SiFli Solution version is 2.x (SF32LB52x/55x/56x/58x series)
  2. Project LVGL config is V8.x (judge via `lv_conf_sifli.h` or the `DISABLE_LVGL_V9` macro)
- **Manual (条件 B)** — user explicitly requests LVGL V8 development / 用户明确要求使用此技能进行 LVGL V8 开发

## Screen Adaptation / 屏幕适配（生成代码前必须确认）

Before generating any LVGL UI code, **confirm the current project's screen parameters**:

### Required info / 需要确认的信息

1. **Resolution / 屏幕分辨率**: width × height (e.g., 466×466, 454×454, 240×280)
2. **Shape / 屏幕形状**: Square or Round

### How to obtain / 获取方式

- **Resolution**: read from the project config — `LV_HOR_RES_MAX` / `LV_VER_RES_MAX` in `lv_conf_sifli.h` (or `LCD_HOR_RES_MAX` / `LCD_VER_RES_MAX` in `rtconfig.h`)
- **Shape**: via the `BSP_USING_ROUND_TYPE_LCD` macro in `rtconfig.h` — defined as `1` → **Round screen**; undefined / `0` → **Square screen**
- If none of the above is determinable, ask the user

### Adaptation requirements / 适配要求

- Size and position of every widget should be computed proportionally from `LV_HOR_RES_MAX` / `LV_VER_RES_MAX`, never hardcode absolute pixels
- On round screens, avoid placing widgets in the corner areas (out of the visible region)
- Font sizes, spacing, etc. should also scale with resolution
- Prefer relative layout APIs: `lv_obj_align` / `lv_obj_align_to`

## Project Structure Overview

```
sdk/middleware/lvgl/
  lv_conf_sifli.h            -- Main LVGL config (memory, tick, GPU, fonts, V8/V9 compat)
  littlevgl2rtt.c/h          -- RT-Thread integration (lv_init, FPS/perf monitor)
  lv_drivers/                 -- V8 display/touch/keypad/wheel drivers
  lvsf/                       -- SiFli custom LVGL extensions (lvsf.h is master include)

solution/framework/
  gui_widget/                 -- Solution-level custom LVGL widgets (50+ widgets)
  gui_fwk/reg_fwk/app_reg.h   -- Application registration macros

solution/examples/watch/application/  -- Watch UI applications reference
```

## Key Configuration

- **Include order**: Always include `lvgl.h` first, then `lvsf.h` for SiFli extensions.
- **Memory**: Uses RT-Thread `rt_malloc`/`rt_free` (LV_MEM_CUSTOM).
- **Tick**: `rt_tick_get_millisecond()` via LV_TICK_CUSTOM.
- **GPU**: SiFli EPIC GPU enabled (LV_USE_GPU_SIFLI_EPIC).
- **Display**: Resolution from `LV_HOR_RES_MAX`/`LV_VER_RES_MAX`.
- **Font sizes**: `FONT_SMALL`, `FONT_NORMAL`, `FONT_SUBTITLE`, `FONT_TITLE`, `FONT_BIG` (defined via `lv_ext_set_local_font`).

## Application Architecture

### 1. Application Registration

Use `APPLICATION_REGISTER` macro (expands to `BUILTIN_APP_EXPORT`). This is the standard way to register a built-in GUI app.

```c
// flashlight_app.c - simplest example
#include "global.h"
#include "app_module.h"

// ... entry function and pages ...

APPLICATION_REGISTER(app_get_strid(key_myapp, "My App"), img_myapp, MY_APP_ID, 0);
```

Parameters: `(display_name_sid, icon_image, app_id_string, struct_size_for_state)`.

- `APPLICATION_REGISTER_HIDDEN` - register app not shown in main menu
- `APPLICATION_REGISTER_PATH` - for dynamic loading apps

### 2. Subpage Pattern (gui_app_create_page)

Every UI page is a "subpage" with a lifecycle message handler. The entry function calls `gui_app_create_page`:

```c
// Standard subpage pattern (see app_calendar_schedule.c)
static void msg_handler(gui_app_msg_type_t msg, void *param)
{
    switch (msg) {
    case GUI_APP_MSG_ONSTART:
        // Create UI: lv_obj_create, lv_label_create, etc.
        break;
    case GUI_APP_MSG_ONRESUME:
        // Resume timers, refresh data
        break;
    case GUI_APP_MSG_ONPAUSE:
        // Pause timers
        break;
    case GUI_APP_MSG_ONSTOP:
        // Clean up, delete objects (lv_scr_act() children are auto-deleted)
        break;
    default:
        break;
    }
}

void app_my_page_main(void)
{
    gui_app_create_page("my_page_id", msg_handler);
}
```

**Lifecycle flow**: ONSTART → ONRESUME (page active) → ONPAUSE → ONSTOP (page destroyed).

### 3. UI Creation Patterns

Always create UI objects as children of `lv_scr_act()` or a container:

```c
static void page_init(void)
{
    lv_obj_t *bg = lv_obj_create(lv_scr_act());
    lv_obj_set_size(bg, LV_HOR_RES_MAX, LV_VER_RES_MAX);
    lv_obj_align(bg, LV_ALIGN_CENTER, 0, 0);

    // Header (manual or using lvsf components)
    lv_obj_t *header = lv_obj_create(bg);
    lv_obj_set_size(header, LV_HOR_RES_MAX, LV_VER_RES_MAX / 10);

    // Label with font
    lv_obj_t *label = lv_label_create(bg);
    lv_ext_set_local_font(label, FONT_TITLE, LV_COLOR_WHITE);
    lv_label_set_text(label, "Hello");
}
```

**Key APIs**:
- `lv_obj_create(parent)` - create container
- `lv_obj_set_size(obj, w, h)` - set size
- `lv_obj_align(obj, align, x_ofs, y_ofs)` / `lv_obj_align_to(obj, base, align, x, y)` - alignment
- `lv_label_create(parent)` / `lv_label_set_text(label, text)` - label
- `lv_btn_create(parent)` - button
- `lv_img_create(parent)` / `lv_img_set_src(img, &img_dsc)` - image
- `lv_obj_add_event_cb(obj, cb, LV_EVENT_ALL, user_data)` - event callback
- `lv_obj_set_scrollbar_mode(obj, LV_SCROLLBAR_MODE_OFF)` - hide scrollbar

### 4. Event Handling (LVGL V8 style)

```c
static void btn_click_cb(lv_event_t *e)
{
    lv_event_code_t code = lv_event_get_code(e);
    if (LV_EVENT_SHORT_CLICKED == code) {
        // handle click
    }
}

lv_obj_add_event_cb(btn, btn_click_cb, LV_EVENT_ALL, NULL);
```

### 5. Navigation

- `gui_app_goback()` - navigate back to previous page
- `gui_app_switch_app(app_id)` - switch to another application

### 6. Timers

```c
lv_timer_t *timer = lv_timer_create(timer_cb, interval_ms, user_data);
// Store timer handle to pause/resume/delete later
lv_timer_del(timer);
```

### 7. LVGL V8 Delayed Layout / LVGL V8 延迟布局

LVGL V8 applies widget layout in a **delayed** manner: positions/sizes are recalculated during the next refresh (`lv_refr_now()` or the next `lv_timer_handler` frame). Before that, APIs like `lv_obj_get_width()` / `lv_obj_get_height()` return **stale values**.

LVGL V8 的控件布局是**延时生效**的：在布局刷新（`lv_refr_now()` 或下一帧 `lv_timer_handler`）之前，通过 `lv_obj_get_width()` / `lv_obj_get_height()` 获取到的是旧值，不准确。

**Avoid / 禁止的做法** — getting a widget size first, then computing offsets from it / 先获取控件尺寸再基于尺寸计算偏移：

```c
// ❌ Wrong / 错误：布局刷新前获取到的尺寸可能是旧值
lv_obj_t *label = lv_label_create(parent);
lv_label_set_text(label, "Some text");
lv_coord_t w = lv_obj_get_width(label);             // may be the old value! / 可能是旧值！
lv_obj_t *next = lv_obj_create(parent);
lv_obj_align(next, LV_ALIGN_TOP_LEFT, 0, w + 10);   // unreliable offset / 不可靠的偏移
```

**Correct / 正确的做法** — always use anchor-based alignment (`lv_obj_align` / `lv_obj_align_to`) and let LVGL compute positions automatically / 始终使用锚点相对布局，由 LVGL 自动计算位置：

```c
// ✅ Correct / 正确：LVGL 在下一帧自动计算实际位置
lv_obj_t *header = lv_obj_create(parent);
lv_obj_set_size(header, LV_HOR_RES_MAX, LV_VER_RES_MAX / 10);
lv_obj_align(header, LV_ALIGN_TOP_MID, 0, 0);

lv_obj_t *body = lv_obj_create(parent);
lv_obj_set_size(body, LV_HOR_RES_MAX, LV_VER_RES_MAX - LV_VER_RES_MAX / 10);
lv_obj_align_to(body, header, LV_ALIGN_OUT_BOTTOM_MID, 0, 0);  // auto-sticks below header / 自动紧贴 header 下方
```

**Core principle / 核心原则**: rely on parent/child or sibling anchor relationships so LVGL handles coordinate calculation, avoiding overlap or misplacement when source widget sizes change.
利用控件间的父子/兄弟锚点关系，让 LVGL 自动处理坐标计算，确保源控件大小变化时不会出现位置重叠等非预期效果。

## Widget Selection Priority / 控件选用优先级

When multiple widgets can satisfy the requirement, choose in this order / 当多种控件都能满足需求时，按以下优先级选用：

| Priority / 优先级 | Source / 来源 | Description / 说明 |
|--------|------|------|
| 1 (highest) | **SiFli custom widgets / 思澈自定义控件** — `lvsf_*` in `sdk/middleware/lvgl/lvsf/` and `solution/framework/gui_widget/` | SiFli official docs / 思澈官方文档 |
| 2 | **DAL custom widgets / DAL 前缀自定义控件** — `dal_*` (e.g. `dal_blood_pressure_tlv.h`, `dal_heartrate_tlv.h`), usually page-level widgets with business logic | Project code / 项目代码 |
| 3 | **LVGL standard widgets / LVGL 标准控件** (`lv_btn`, `lv_label`, `lv_img`, `lv_chart`, ...) | LVGL official docs / LVGL 官方文档 |
| 4 (lowest) | **Basic combination / new implementation / 基础控件组合或新增实现** | — |

## SiFli LVSF Extensions (lvsf.h)

After including `lvsf.h`, these custom components are available:

| Component | Header | Usage |
|-----------|--------|-------|
| Header/Title | `lvsf_header.h` | `lv_title_create(parent)` - standardized title bar |
| Popup dialog | `lvsf_popup.h` | Popup dialogs with standard styling |
| Analog clock | `lvsf_analogclk.h` | Analog clock widget |
| Composite | `lvsf_composite.h` | Compound widget base |
| Input dispatch | `lvsf_input.h` | Keypad/wheel input handling |
| Gesture | `lvsf_gesture.h` | Gesture recognition |
| Theme | `lvsf_theme_1.h` | Default SiFli theme |
| Font mgmt | `lvsf_font.h` | FreeType font management |
| Barcode | `lvsf_barcode.h` | Barcode display |
| Curved text | `lvsf_curvetext.h` | Text along curves |
| eZIP image | `lvsf_ezipa.h` | Compressed image format |
| Lottie anim | `lvsf_rlottie.h` | Lottie animation |
| Indexed image | `lvsf_idximg.h` | Indexed color images |
| Switch anim | `lvsf_switchanim.h` | Page transition animations |

Additional solution-level widgets are in `solution/framework/gui_widget/`:
`lvsf_arctext`, `lvsf_basechart`, `lvsf_baseimg`, `lvsf_baselabel`, `lvsf_encoder`, `lvsf_gif`, `lvsf_imgarray`, `lvsf_imgbar`, `lvsf_mulroller`, `lvsf_sector`, `lvsf_scrollbar`, `lvsf_timeline`, `lvsf_video`, etc.

## Font Usage

Use `lv_ext_set_local_font()` with predefined size macros:

```c
lv_ext_set_local_font(obj, FONT_SMALL, LV_COLOR_WHITE);    // 12px
lv_ext_set_local_font(obj, FONT_NORMAL, LV_COLOR_WHITE);   // 16px
lv_ext_set_local_font(obj, FONT_SUBTITLE, LV_COLOR_WHITE); // 20px
lv_ext_set_local_font(obj, FONT_TITLE, LV_COLOR_WHITE);    // 24px
lv_ext_set_local_font(obj, FONT_BIG, LV_COLOR_WHITE);      // 28px+
```

## Color Depth

The platform supports `LV_COLOR_DEPTH` 16 (RGB565) or 24 (RGB888). Use standard `lv_color_hex(0xRRGGBB)` or `lv_color_make(r,g,b)`.

## Common Includes for a New Page

```c
#include "global.h"
#include "app_module.h"    // or specific module header
// Core LVGL + SiFli extensions
#include "lvgl.h"
#include "lvsf.h"
```

## Best Practices

1. **Screen adaptation first / 屏幕适配优先**: Before generating any UI code, confirm the screen resolution and shape (square/round) from the project config, and scale all widgets proportionally — never hardcode absolute pixels. See the "Screen Adaptation" section above.
2. **One screen = one subpage**: Each logical screen should be a subpage managed by `gui_app_create_page`.
3. **ONSTART creates, ONSTOP cleans**: Create all UI objects in ONSTART handler. The framework auto-deletes `lv_scr_act()` children on stop, but clean up timers and resources.
4. **Use `lv_obj_create(lv_scr_act())` as root container**: Always create a full-screen background container as the first child.
5. **Event-driven input**: Use `lv_obj_add_event_cb` with `LV_EVENT_SHORT_CLICKED` / `LV_EVENT_LONG_PRESSED` for button interactions.
6. **Anchor-based layout**: Prefer `lv_obj_align` / `lv_obj_align_to` over absolute coordinates. Do NOT read `lv_obj_get_width()` / `lv_obj_get_height()` before a layout refresh and compute offsets from stale values (see "LVGL V8 Delayed Layout" above).
7. **V8 vs V9**: This is LVGL V8. Do NOT use V9 APIs (e.g., no `lv_<widget>_create` with separate `lv_<widget>_set_*` style — V8 uses positional params in create functions).
8. **SiFli compat macros**: `lv_conf_sifli.h` provides V8↔V9 compatibility macros. Use standard V8 API names.
