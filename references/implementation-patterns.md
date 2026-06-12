# Implementation Patterns

Use these patterns after the asset plan is clear and the supplied files have been inspected.
Prefer the host project's conventions over the examples below.

## Work Rhythm

1. Inspect project scripts, framework, i18n, logger, and existing asset folders.
2. Implement the smallest browser-previewable loop with real or accepted placeholder assets.
3. Start the dev server and give the user the local URL.
4. Iterate on browser feedback until the user confirms the visual direction and core behavior.
5. Add desktop-runtime behavior.
6. Package only after desktop runtime checks pass.

This rhythm matters because browser preview is faster for visual and interaction feedback, while
desktop runtime is necessary for transparent windows, drag, monitor placement, and packaging.

For fuller copy-adaptable code, read `code-recipes.md`. Prefer those recipes over rewriting common
asset loading, chroma key, settings persistence, and runtime bridge code from scratch.

## Vue/Tauri Structure

Use the host project's naming first. For Vue desktop pets, a common structure is:

- `DesktopPet.vue` or the existing pet component: canvas rendering, interaction state, settings
  state, asset loading.
- `src/i18n/messages.ts`: all user-facing text.
- `src/utils/logger.ts`: project logger, not `console.log`.
- `src-tauri/src/main.rs`: window commands, pointer listener, drag/follow bridge, resize logic.

Prefer small composables or helper functions when logic grows. Keep each function under 100 lines.

## Canvas Rendering Rules

Render by source category:

```ts
type RenderSource = "video" | "follow-frame" | "image";

const shouldUseChromaKey = (source: RenderSource): boolean => source === "video";
const shouldUseFollowFrameDirectly = (source: RenderSource): boolean =>
  source === "follow-frame";
```

For transparent frames:

- Draw PNG or WebP frames directly with `globalAlpha = 1`.
- Do not call green-screen removal.
- Do not normalize alpha or apply soft-edge passes.
- Do not fade the active frame during follow mode.

For solid-color video actions:

- Reuse a canvas buffer instead of allocating per frame.
- Chroma-key only when the source is video.
- Keep edge cleanup color-based where possible.

```ts
const softenGreenEdge = (r: number, g: number, b: number): [number, number, number] => {
  const greenBias = g - Math.max(r, b);

  if (greenBias < 16) {
    return [r, g, b];
  }

  const dampen = Math.min(0.45, greenBias / 180);
  const nextGreen = Math.round(g * (1 - dampen) + Math.max(r, b) * dampen);

  return [r, nextGreen, b];
};
```

The key lesson: fuzzy edges should not become transparent. Only confident background should.

## Watermark and Rectangular Artifact Handling

Avoid fixed masks unless absolutely needed. If a provider watermark must be removed:

```ts
const canUseWatermarkMask = (actionId: string): boolean =>
  !["wide-pose", "lie-down", "full-body-stretch"].includes(actionId);
```

Make mask bounds per asset instead of global when possible. A standard rectangle in a corner is
usually code or watermark masking, not alpha matting.

## Follow Transition Without Ghosting

Use a short release and fade only the previous visual layer:

```ts
const followTransitionMs = 180;

const getPreviousFrameAlpha = (elapsedMs: number): number => {
  if (elapsedMs <= 0) {
    return 1;
  }

  return Math.max(0, 1 - elapsedMs / followTransitionMs);
};
```

Draw order:

1. Previous frame or action at `getPreviousFrameAlpha(elapsedMs)`.
2. Current follow frame at `1`.

If two subjects appear while switching to follow mode, start the previous-layer fade earlier and
keep the active follow layer opaque.

## Settings Persistence

Persist settings immediately after user changes them:

```ts
interface PetSettings {
  scale: number;
  enabledActions: Record<string, boolean>;
  actionRatios: Record<string, number>;
  enablePointerFollow: boolean;
  enableDragging: boolean;
}

const storageKey = "desktop-pet-settings";

const loadSettings = (): PetSettings | null => {
  const raw = localStorage.getItem(storageKey);

  if (!raw) {
    return null;
  }

  return JSON.parse(raw) as PetSettings;
};

const saveSettings = (settings: PetSettings): void => {
  localStorage.setItem(storageKey, JSON.stringify(settings));
};
```

Use i18n keys for labels. Do not hardcode panel copy in Vue templates.

## Runtime Bridge

Keep browser preview working, then add runtime behavior behind feature detection. The Rust or
Electron side should own OS window movement and monitor clamping.

```ts
interface MovePetWindowArgs {
  x: number;
  y: number;
}

const canUseDesktopRuntime = (): boolean => "__TAURI_INTERNALS__" in window;

const movePetWindow = async (args: MovePetWindowArgs): Promise<void> => {
  if (!canUseDesktopRuntime()) {
    return;
  }

  const { invoke } = await import("@tauri-apps/api/core");
  await invoke<void>("move_pet_window", args);
};
```

Avoid `any`; define typed argument interfaces for every runtime command.

## Tauri Window Commands

Mouse and drag commands usually include:

```rust
#[tauri::command]
fn set_pointer_following(app: tauri::AppHandle, enabled: bool) -> Result<(), String> {
  let window = app.get_webview_window("main").ok_or("找不到主窗口")?;

  window
    .set_ignore_cursor_events(false)
    .map_err(|err| format!("设置鼠标事件失败: {err}"))?;

  if enabled {
    // 启动指针监听并向前端发送位置事件。
  }

  Ok(())
}
```

For settings, resize and clamp instead of letting the panel depend on the pet's current position:

```rust
#[tauri::command]
fn resize_settings_window(app: tauri::AppHandle, open: bool) -> Result<(), String> {
  let window = app.get_webview_window("main").ok_or("找不到主窗口")?;

  let size = if open {
    tauri::PhysicalSize::new(420, 520)
  } else {
    tauri::PhysicalSize::new(320, 320)
  };

  window
    .set_size(tauri::Size::Physical(size))
    .map_err(|err| format!("调整窗口大小失败: {err}"))?;

  if open {
    if let Some(monitor) = window.current_monitor().map_err(|err| err.to_string())? {
      let area = monitor.work_area();
      let position = window.outer_position().map_err(|err| err.to_string())?;
      let max_x = area.position.x + area.size.width as i32 - size.width as i32;
      let max_y = area.position.y + area.size.height as i32 - size.height as i32;
      let next_x = position.x.clamp(area.position.x, max_x);
      let next_y = position.y.clamp(area.position.y, max_y);

      window
        .set_position(tauri::Position::Physical(tauri::PhysicalPosition::new(next_x, next_y)))
        .map_err(|err| format!("移动窗口失败: {err}"))?;
    }
  }

  Ok(())
}
```

Remember the previous pet window position before opening settings and restore it when closing.

## Browser Preview Checks

After implementation, run the local dev server before desktop packaging:

```powershell
npm run dev
```

Tell the user the local URL and ask them to check:

- Subject size, style, anchor, and cropping.
- Basic interactions and mode switching.
- Settings panel layout.
- Transparent area behavior on a normal web background.
- Whether any asset should be regenerated before desktop work continues.

Do not proceed to desktop packaging until the user confirms or gives specific changes to make.

## Performance Guardrails

- Target 30 fps for canvas animation unless the asset truly needs 60 fps.
- Decode/load assets once and cache them by action id.
- Reuse `ImageData`, `CanvasRenderingContext2D`, and scratch canvases where practical.
- Pause non-visible or inactive videos.
- Avoid per-frame object churn in pointer-follow loops.
- Keep desktop global listeners stopped when follow is disabled.

## Packaging Notes

Recommended order after user confirmation:

```powershell
npm run build
cargo check
npm run tauri:build
```

On Windows machines with limited memory or locked release files:

```powershell
$env:CARGO_BUILD_JOBS = "1"
$env:TEMP = "D:\Dl\cat\tmp\tauri-temp"
npm run tauri:build
```

For mac distribution, provide a minimal source package that excludes heavy build outputs and
includes `src`, `src-tauri`, `public`, lockfiles, and config files. Let the Mac build machine run
the Tauri build locally so native bundles are produced on the target platform.
