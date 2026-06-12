# Code Recipes

Use these recipes when the target project does not already have a stronger local pattern. Adapt
names, paths, and action ids to the project. Keep Vue `script`/`template`/`style` order, two-space
indentation, TypeScript semicolons, no `any`, and i18n for user-facing text.

## File Layout

```text
src/
  App.vue
  components/
    DesktopPet.vue
    SettingsPanel.vue
  composables/
    usePetAssets.ts
    usePetSettings.ts
    useRuntimeBridge.ts
  i18n/
    index.ts
    messages.ts
  utils/
    logger.ts
public/
  videos/
  follow-frames/
  props/
  icons/
```

## i18n Messages

```ts
export const messages = {
  zh: {
    app: {
      name: "桌面伙伴"
    },
    settings: {
      title: "设置",
      scale: "画面大小",
      actions: "动作",
      interactions: "互动",
      followPointer: "指针跟随",
      dragging: "拖拽移动",
      close: "关闭"
    }
  }
};

export type Locale = keyof typeof messages;
export type MessageSchema = typeof messages.zh;
```

```ts
import { computed, ref } from "vue";
import { messages, type Locale } from "./messages";

const locale = ref<Locale>("zh");
type NestedMessages = Record<string, string | NestedMessages>;

export const useI18n = () => {
  const t = (path: string): string => {
    const parts = path.split(".");
    let current: NestedMessages = messages[locale.value];

    for (const part of parts) {
      const next = current[part];

      if (typeof next === "string") {
        return next;
      }

      if (!next || typeof next !== "object") {
        return path;
      }

      current = next;
    }

    return path;
  };

  return {
    locale: computed(() => locale.value),
    t
  };
};
```

## Logger

```ts
interface LogPayload {
  message: string;
  scope?: string;
  detail?: string;
}

export const logger = {
  info(payload: LogPayload): void {
    console.info(JSON.stringify(payload));
  },
  warn(payload: LogPayload): void {
    console.warn(JSON.stringify(payload));
  },
  error(payload: LogPayload): void {
    console.error(JSON.stringify(payload));
  }
};
```

If the project already has a logger, use it instead. Keep logger calls to one argument.

## Asset Types

```ts
export type RenderSource = "video" | "follow-frame" | "image";

export interface PetAction {
  id: string;
  labelKey: string;
  source: string;
  sourceType: RenderSource;
  weight: number;
  enabled: boolean;
  canUseWatermarkMask: boolean;
}

export interface LoadedVideoAsset {
  type: "video";
  id: string;
  element: HTMLVideoElement;
}

export interface LoadedImageAsset {
  type: "image";
  id: string;
  element: HTMLImageElement;
}

export type LoadedPetAsset = LoadedVideoAsset | LoadedImageAsset;

export const petActions: PetAction[] = [
  {
    id: "idle-primary",
    labelKey: "settings.actions",
    source: "/videos/idle-primary.mp4",
    sourceType: "video",
    weight: 60,
    enabled: true,
    canUseWatermarkMask: true
  }
];
```

## Asset Loader

```ts
import { onBeforeUnmount, shallowRef } from "vue";
import { logger } from "../utils/logger";
import type { LoadedPetAsset, PetAction } from "../shared/petTypes";

const loadVideo = async (action: PetAction): Promise<LoadedPetAsset> => {
  const video = document.createElement("video");
  video.src = action.source;
  video.muted = true;
  video.loop = true;
  video.playsInline = true;
  video.preload = "auto";

  await new Promise<void>((resolve, reject) => {
    video.oncanplay = () => resolve();
    video.onerror = () => reject(new Error(`视频加载失败: ${action.source}`));
    video.load();
  });

  return {
    type: "video",
    id: action.id,
    element: video
  };
};

const loadImage = async (action: PetAction): Promise<LoadedPetAsset> => {
  const image = new Image();
  image.src = action.source;

  await new Promise<void>((resolve, reject) => {
    image.onload = () => resolve();
    image.onerror = () => reject(new Error(`图片加载失败: ${action.source}`));
  });

  return {
    type: "image",
    id: action.id,
    element: image
  };
};

export const usePetAssets = (actions: PetAction[]) => {
  const assets = shallowRef<Map<string, LoadedPetAsset>>(new Map());

  const loadAssets = async (): Promise<void> => {
    const entries = await Promise.all(
      actions.map(async (action) => {
        const asset = action.sourceType === "video"
          ? await loadVideo(action)
          : await loadImage(action);

        return [action.id, asset] as const;
      })
    );

    assets.value = new Map(entries);
  };

  const playVideo = async (id: string): Promise<void> => {
    const asset = assets.value.get(id);

    if (!asset || asset.type !== "video") {
      return;
    }

    try {
      await asset.element.play();
    } catch (error) {
      logger.warn({
        scope: "assets",
        message: "视频播放失败",
        detail: error instanceof Error ? error.message : String(error)
      });
    }
  };

  onBeforeUnmount(() => {
    assets.value.forEach((asset) => {
      if (asset.type === "video") {
        asset.element.pause();
      }
    });
  });

  return {
    assets,
    loadAssets,
    playVideo
  };
};
```

## Settings Store

```ts
import { ref, watch } from "vue";
import { logger } from "../utils/logger";

export interface PetSettings {
  scale: number;
  enabledActions: Record<string, boolean>;
  actionRatios: Record<string, number>;
  enablePointerFollow: boolean;
  enableDragging: boolean;
}

const storageKey = "desktop-pet-settings";

const defaultSettings: PetSettings = {
  scale: 80,
  enabledActions: {},
  actionRatios: {},
  enablePointerFollow: true,
  enableDragging: true
};

const clampNumber = (value: number, min: number, max: number): number =>
  Math.min(max, Math.max(min, value));

const normalizeSettings = (value: Partial<PetSettings>): PetSettings => ({
  scale: clampNumber(value.scale ?? defaultSettings.scale, 40, 160),
  enabledActions: value.enabledActions ?? {},
  actionRatios: value.actionRatios ?? {},
  enablePointerFollow: value.enablePointerFollow ?? true,
  enableDragging: value.enableDragging ?? true
});

const readSettings = (): PetSettings => {
  const raw = localStorage.getItem(storageKey);

  if (!raw) {
    return defaultSettings;
  }

  try {
    const parsed = JSON.parse(raw) as Partial<PetSettings>;

    return normalizeSettings(parsed);
  } catch (error) {
    logger.warn({
      scope: "settings",
      message: "读取设置失败，使用默认设置",
      detail: error instanceof Error ? error.message : String(error)
    });

    return defaultSettings;
  }
};

export const usePetSettings = () => {
  const settings = ref<PetSettings>(readSettings());

  watch(
    settings,
    (nextSettings) => {
      localStorage.setItem(storageKey, JSON.stringify(nextSettings));
    },
    { deep: true }
  );

  return {
    settings
  };
};
```

## Runtime Bridge

```ts
import { logger } from "../utils/logger";

interface MoveWindowArgs {
  x: number;
  y: number;
}

interface ResizeSettingsArgs {
  open: boolean;
}

type JsonValue = string | number | boolean | null | JsonValue[] | { [key: string]: JsonValue };
type RuntimeArgs = Record<string, JsonValue>;
type InvokeFn = <T>(command: string, args?: RuntimeArgs) => Promise<T>;

const canUseTauri = (): boolean => "__TAURI_INTERNALS__" in window;

const loadInvoke = async (): Promise<InvokeFn | null> => {
  if (!canUseTauri()) {
    return null;
  }

  const module = await import("@tauri-apps/api/core");

  return module.invoke;
};

const callRuntime = async <T>(
  command: string,
  args?: RuntimeArgs
): Promise<T | null> => {
  const invoke = await loadInvoke();

  if (!invoke) {
    return null;
  }

  try {
    return await invoke<T>(command, args);
  } catch (error) {
    logger.warn({
      scope: "runtime",
      message: "桌面运行时调用失败",
      detail: error instanceof Error ? error.message : String(error)
    });

    return null;
  }
};

export const useRuntimeBridge = () => {
  const moveWindow = async (args: MoveWindowArgs): Promise<void> => {
    await callRuntime<void>("move_pet_window", args);
  };

  const resizeSettings = async (args: ResizeSettingsArgs): Promise<void> => {
    await callRuntime<void>("resize_settings_window", args);
  };

  const setPointerFollowing = async (enabled: boolean): Promise<void> => {
    await callRuntime<void>("set_pointer_following", { enabled });
  };

  return {
    canUseTauri,
    moveWindow,
    resizeSettings,
    setPointerFollowing
  };
};
```

## Chroma Key Renderer Helpers

```ts
interface Rgb {
  r: number;
  g: number;
  b: number;
}

const isConfidentGreen = ({ r, g, b }: Rgb): boolean => {
  const greenBias = g - Math.max(r, b);

  return g > 90 && greenBias > 42;
};

const softenGreenEdge = ({ r, g, b }: Rgb): Rgb => {
  const greenBias = g - Math.max(r, b);

  if (greenBias < 16) {
    return { r, g, b };
  }

  const dampen = Math.min(0.45, greenBias / 180);
  const nextGreen = Math.round(g * (1 - dampen) + Math.max(r, b) * dampen);

  return { r, g: nextGreen, b };
};

export const applyChromaKey = (imageData: ImageData): ImageData => {
  const { data } = imageData;

  for (let index = 0; index < data.length; index += 4) {
    const color = {
      r: data[index],
      g: data[index + 1],
      b: data[index + 2]
    };

    if (isConfidentGreen(color)) {
      data[index + 3] = 0;
      continue;
    }

    const softened = softenGreenEdge(color);
    data[index] = softened.r;
    data[index + 1] = softened.g;
    data[index + 2] = softened.b;
  }

  return imageData;
};

export const drawVideoWithChromaKey = (
  targetContext: CanvasRenderingContext2D,
  scratchContext: CanvasRenderingContext2D,
  video: HTMLVideoElement,
  width: number,
  height: number
): void => {
  scratchContext.clearRect(0, 0, width, height);
  scratchContext.drawImage(video, 0, 0, width, height);

  const frame = scratchContext.getImageData(0, 0, width, height);
  targetContext.putImageData(applyChromaKey(frame), 0, 0);
};
```

## DesktopPet.vue

```vue
<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from "vue";
import { useI18n } from "../i18n";
import { usePetAssets } from "../composables/usePetAssets";
import { usePetSettings } from "../composables/usePetSettings";
import { useRuntimeBridge } from "../composables/useRuntimeBridge";
import { drawVideoWithChromaKey } from "../composables/useChromaKey";
import { petActions, type LoadedPetAsset } from "../shared/petTypes";
import SettingsPanel from "./SettingsPanel.vue";

type PetMode = "idle" | "follow" | "interact";

const baseSize = 320;
const canvasRef = ref<HTMLCanvasElement | null>(null);
const scratchCanvas = document.createElement("canvas");
const mode = ref<PetMode>("idle");
const activeActionId = ref("idle-primary");
const isSettingsOpen = ref(false);

const { t } = useI18n();
const { settings } = usePetSettings();
const { assets, loadAssets, playVideo } = usePetAssets(petActions);
const { moveWindow, resizeSettings, setPointerFollowing } = useRuntimeBridge();

const canvasSize = computed(() => Math.round(baseSize * settings.value.scale / 100));

let animationFrame = 0;

const getContext = (
  canvas: HTMLCanvasElement
): CanvasRenderingContext2D | null => canvas.getContext("2d", { willReadFrequently: true });

const getActiveAsset = (): LoadedPetAsset | null =>
  assets.value.get(activeActionId.value) ?? null;

const clearCanvas = (
  context: CanvasRenderingContext2D,
  width: number,
  height: number
): void => {
  context.clearRect(0, 0, width, height);
};

const drawAsset = (
  context: CanvasRenderingContext2D,
  asset: LoadedPetAsset,
  width: number,
  height: number
): void => {
  if (asset.type === "video") {
    const scratchContext = getContext(scratchCanvas);

    if (!scratchContext) {
      return;
    }

    drawVideoWithChromaKey(context, scratchContext, asset.element, width, height);
    return;
  }

  context.drawImage(asset.element, 0, 0, width, height);
};

const render = (): void => {
  const canvas = canvasRef.value;

  if (!canvas) {
    animationFrame = requestAnimationFrame(render);
    return;
  }

  const context = getContext(canvas);

  if (!context) {
    animationFrame = requestAnimationFrame(render);
    return;
  }

  const width = canvas.width;
  const height = canvas.height;
  const asset = getActiveAsset();

  clearCanvas(context, width, height);

  if (asset) {
    drawAsset(context, asset, width, height);
  }

  animationFrame = requestAnimationFrame(render);
};

const openSettings = async (): Promise<void> => {
  isSettingsOpen.value = true;
  await resizeSettings({ open: true });
};

const closeSettings = async (): Promise<void> => {
  isSettingsOpen.value = false;
  await resizeSettings({ open: false });
};

const handlePointerMove = async (event: PointerEvent): Promise<void> => {
  if (!settings.value.enablePointerFollow || mode.value !== "follow") {
    return;
  }

  await moveWindow({
    x: Math.round(event.screenX - canvasSize.value / 2),
    y: Math.round(event.screenY - canvasSize.value / 2)
  });
};

const handlePointerEnter = async (): Promise<void> => {
  if (!settings.value.enablePointerFollow) {
    return;
  }

  mode.value = "follow";
  await setPointerFollowing(true);
};

const handlePointerLeave = async (): Promise<void> => {
  mode.value = "idle";
  await setPointerFollowing(false);
};

onMounted(async () => {
  scratchCanvas.width = canvasSize.value;
  scratchCanvas.height = canvasSize.value;
  await loadAssets();
  await playVideo(activeActionId.value);
  render();
});

onBeforeUnmount(() => {
  cancelAnimationFrame(animationFrame);
});
</script>

<template>
  <main
    class="pet-root"
    @pointerenter="handlePointerEnter"
    @pointerleave="handlePointerLeave"
    @pointermove="handlePointerMove"
    @contextmenu.prevent="openSettings"
  >
    <canvas
      ref="canvasRef"
      class="pet-canvas"
      :width="canvasSize"
      :height="canvasSize"
      :aria-label="t('app.name')"
    />

    <SettingsPanel
      v-if="isSettingsOpen"
      :settings="settings"
      @update:settings="settings = $event"
      @close="closeSettings"
    />
  </main>
</template>

<style scoped>
.pet-root {
  width: 100vw;
  height: 100vh;
  display: grid;
  place-items: center;
  background: transparent;
  overflow: hidden;
}

.pet-canvas {
  width: v-bind(canvasSize + "px");
  height: v-bind(canvasSize + "px");
  display: block;
}
</style>
```

## SettingsPanel.vue

```vue
<script setup lang="ts">
import { useI18n } from "../i18n";
import type { PetSettings } from "../composables/usePetSettings";

interface Props {
  settings: PetSettings;
}

interface Emits {
  "update:settings": [settings: PetSettings];
  close: [];
}

const props = defineProps<Props>();
const emit = defineEmits<Emits>();
const { t } = useI18n();

const updateSettings = (patch: Partial<PetSettings>): void => {
  emit("update:settings", {
    ...props.settings,
    ...patch
  });
};

const close = (): void => {
  emit("close");
};
</script>

<template>
  <section class="settings-panel">
    <header class="settings-header">
      <h2>{{ t("settings.title") }}</h2>
      <button
        class="settings-close"
        type="button"
        @click="close"
      >
        {{ t("settings.close") }}
      </button>
    </header>

    <label class="settings-row">
      <span>{{ t("settings.scale") }}</span>
      <input
        type="range"
        min="40"
        max="160"
        step="5"
        :value="props.settings.scale"
        @input="updateSettings({
          scale: Number(($event.target as HTMLInputElement).value)
        })"
      />
    </label>

    <label class="settings-row">
      <span>{{ t("settings.followPointer") }}</span>
      <input
        type="checkbox"
        :checked="props.settings.enablePointerFollow"
        @change="updateSettings({
          enablePointerFollow: ($event.target as HTMLInputElement).checked
        })"
      />
    </label>

    <label class="settings-row">
      <span>{{ t("settings.dragging") }}</span>
      <input
        type="checkbox"
        :checked="props.settings.enableDragging"
        @change="updateSettings({
          enableDragging: ($event.target as HTMLInputElement).checked
        })"
      />
    </label>
  </section>
</template>

<style scoped>
.settings-panel {
  width: min(420px, 100vw);
  max-height: 100vh;
  padding: 16px;
  color: #1d1d1f;
  background: rgba(255, 255, 255, 0.94);
  border: 1px solid rgba(0, 0, 0, 0.08);
  overflow: auto;
}

.settings-header,
.settings-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.settings-header h2 {
  margin: 0;
  font-size: 18px;
}

.settings-close {
  border: 0;
  padding: 6px 10px;
  color: #ffffff;
  background: #1d1d1f;
  cursor: pointer;
}

.settings-row {
  margin-top: 14px;
  font-size: 14px;
}
</style>
```

## Tauri Commands

```rust
use tauri::Manager;

#[tauri::command]
fn move_pet_window(app: tauri::AppHandle, x: i32, y: i32) -> Result<(), String> {
  let window = app.get_webview_window("main").ok_or("找不到主窗口")?;

  window
    .set_position(tauri::Position::Physical(tauri::PhysicalPosition::new(x, y)))
    .map_err(|err| format!("移动窗口失败: {err}"))?;

  Ok(())
}

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
    clamp_to_current_monitor(&window, size)?;
  }

  Ok(())
}

#[tauri::command]
fn set_pointer_following(app: tauri::AppHandle, enabled: bool) -> Result<(), String> {
  let window = app.get_webview_window("main").ok_or("找不到主窗口")?;

  window
    .set_ignore_cursor_events(false)
    .map_err(|err| format!("设置鼠标事件失败: {err}"))?;

  if enabled {
    // 指针监听可按项目需要接入全局鼠标库。
  }

  Ok(())
}

fn clamp_to_current_monitor(
  window: &tauri::WebviewWindow,
  size: tauri::PhysicalSize<u32>
) -> Result<(), String> {
  let Some(monitor) = window.current_monitor().map_err(|err| err.to_string())? else {
    return Ok(());
  };

  let area = monitor.work_area();
  let position = window.outer_position().map_err(|err| err.to_string())?;
  let max_x = area.position.x + area.size.width as i32 - size.width as i32;
  let max_y = area.position.y + area.size.height as i32 - size.height as i32;
  let next_x = position.x.clamp(area.position.x, max_x);
  let next_y = position.y.clamp(area.position.y, max_y);

  window
    .set_position(tauri::Position::Physical(tauri::PhysicalPosition::new(next_x, next_y)))
    .map_err(|err| format!("移动窗口失败: {err}"))?;

  Ok(())
}

pub fn run() {
  tauri::Builder::default()
    .invoke_handler(tauri::generate_handler![
      move_pet_window,
      resize_settings_window,
      set_pointer_following
    ])
    .run(tauri::generate_context!())
    .expect("运行桌宠失败");
}
```

## Browser-First Verification Commands

```powershell
npm run dev
```

After the user confirms the browser preview:

```powershell
npm run tauri:dev
npm run build
npm run tauri:build
```
