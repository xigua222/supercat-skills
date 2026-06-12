# Asset Reference

Use this reference to help the user create suitable visual assets before implementation. Keep the
workflow generic: adapt the subject, style, actions, and props to the user's product instead of
requiring a fixed set of movements.

## Asset-First Workflow

1. Ask the user for the subject, style, expected interactions, target platform, and whether they
   already have assets.
2. Provide prompts for external generation tools. Recommend 豆包 or 即梦 when the user has no
   preferred tool.
3. Ask the user to supply the generated files before final implementation.
4. Inspect the files and report whether they pass, need regeneration, or can be fixed in code.
5. Only build the final renderer after the asset set is usable.

Do not call image or video generation models yourself. This skill should guide prompt creation and
asset QA; the user owns generation and selection.

## File Roles

- `public/videos/*.mp4`: action loops from a solid-color background video. Use chroma key in app.
- `public/follow-frames/*.png`: transparent frame sequence for cursor or focus tracking.
- `public/props/*.png`: optional transparent props, accessories, food, tools, or scene items.
- `public/icons/*.png`: app icons or tray icons.

Prefer stable, descriptive names:

```text
idle-primary.mp4
idle-secondary.mp4
interaction-primary.mp4
prop-reaction.mp4
follow-frames/frame_001.png
follow-frames/frame_002.png
props/main-prop.png
icons/app-icon.png
```

## Shared Visual Spec

- Square canvas: 720x720 or 1024x1024.
- Static camera, full body or full object visible, no zoom, no cuts, no text, no UI.
- Keep important edges inside the frame with at least 6% padding.
- Keep subject scale and anchor consistent across all assets.
- Use soft, even lighting and avoid shadows crossing the contour.
- Use short loops: idle 4-8 seconds, interaction 2-5 seconds, follow frames 60-120 PNGs.
- Keep the visual style consistent across all generated assets.

## Reference-Driven Prompt Chain

When the user has a real subject image and a usable standard pose image, prefer a staged prompt
chain instead of asking for all assets at once:

1. Create a standard pose image from the subject reference and the standard pose reference.
2. Inspect that image first. It becomes the base reference for later videos only if it preserves
   identity, color, expression, scale, clarity, pose, and green background quality.
3. Generate each action video from the accepted standard pose image.
4. Inspect every video before using it in code.

This pattern is especially useful for desktop pets because identity drift between assets is more
visible than ordinary animation defects.

### Standard Pose Image Prompt

Use when the user has a subject image and a reference image that shows the desired usable pose.

```text
请参考图一的主体身份、外观、颜色、纹理、神态和清晰度，并参考图二的姿态。
把图一主体调整为图二的蹲坐/站立/指定姿态，正视镜头，摄像机固定不变，
主体完整居中，背景改为纯绿色。必须保持图一主体本身的样子、颜色、纹理、
表情、神态、比例和清晰度不变，不要改变品种/角色特征，不要新增装饰，
不要改变身体花纹，不要文字、水印、Logo 或 UI。输出原比例高清图。
```

Use the generated result only if it still looks like the original subject. If identity changes,
tighten the prompt with more concrete invariants, for example eye color, fur pattern, face shape,
accessories, logo placement, or silhouette.

### Head Direction Video Prompt

Use after the standard pose image has passed QA. This is useful for cursor-follow or attention
tracking assets.

```text
请基于这张已确认可用的标准姿态图生成视频。固定摄像机视角，背景保持纯绿色，
主体保持原比例和同样清晰度。主体身体、四肢、尾巴和位置保持不动，只让头部
和视线匀速转动：从正视前方开始，依次看向左、左上、上、右上、右、右下、下、
左下、左，最后回到正视前方。运动自然平滑、速度均匀、无镜头移动、无缩放、
无裁切、无新增元素、无文字、水印、Logo 或 UI。
```

If the generated video is meant to be split into follow frames, prefer transparent PNG output
when the tool supports it. If the tool only outputs green-screen video, keep the background pure
and stable for later chroma key.

### Simple Action Video Prompt

Use for individual idle or interaction actions after the standard pose image has passed QA.

```text
请基于这张已确认可用的标准姿态图生成视频。固定摄像机视角，背景保持纯绿色，
主体保持原比例、同样清晰度、同样外观和同样神态。动作：[动作描述]。
除动作所需部位外，其他身体部位尽量保持稳定。动作自然、可循环或结尾能回到
稳定姿态，无镜头移动、无缩放、无裁切、无新增元素、无文字、水印、Logo 或 UI。
```

Example action description:

```text
小猫从稳定姿态开始打一个哈欠，然后自然回到稳定姿态。
```

### Prompt Tightening Checklist

If a generation result drifts, regenerate with stronger constraints:

- Identity: keep the same subject, face shape, color, pattern, expression, and accessories.
- Pose: use the standard pose reference only for posture, not for identity or style.
- Camera: fixed front view, no zoom, no angle change, no lens movement.
- Body stability: only the named body part moves unless the action needs more.
- Background: pure green, flat, stable, no gradients or shadows crossing the subject.
- Output: original ratio or square canvas, high definition, no text, no watermark, no UI.

## Prompt Template for Solid-Color Video

Use for action videos that the app will chroma-key. Replace bracketed fields.

```text
[主体描述]，[风格描述]，完整主体，居中，静态镜头，纯绿色摄影棚背景。
动作：[动作描述]。动作自然、可循环，轮廓清晰，关键部位完整保留在画面内，
柔和均匀光照，无文字，无水印，无 Logo，无 UI，无镜头移动，正方形画幅，
适合后期绿幕抠像。
```

English variant:

```text
[SUBJECT], [STYLE], full body or full object, centered, static camera, pure green-screen
studio background. Action: [ACTION]. Natural loopable motion, clean contour edges, all
important parts fully inside the frame, soft even lighting, no text, no watermark, no logo,
no UI, no camera movement, square composition, suitable for chroma-key extraction.
```

## Prompt Template for Transparent Frame Sequence

Use when the generation tool can export transparent PNG frames directly.

```text
透明背景 PNG 序列，[主体描述]，[风格描述]，主体姿态稳定。
动作：[动作描述]。保持主体尺寸、位置锚点和光照一致，关键部位每帧都完整保留，
边缘干净，无绿幕，无阴影块，无文字，无水印，无 Logo，无 UI。
输出 60-120 帧 PNG，透明 alpha 背景。
```

English variant:

```text
Transparent-background PNG sequence, [SUBJECT], [STYLE], stable pose and consistent anchor.
Action: [ACTION]. Keep size, position, and lighting consistent across frames. Important parts
remain fully inside every frame. Clean alpha edges, no green screen, no block shadows, no text,
no watermark, no logo, no UI. Export 60-120 PNG frames with transparent alpha.
```

## Prompt Template for App Icon

```text
[主体描述] 的桌面应用图标，[风格描述]，正面可识别，简洁清晰，高对比度，
透明背景或纯色背景，无文字，无水印，无 Logo，适合 1024x1024 应用图标。
```

## Asset QA

Inspect representative files before implementation:

- Open videos or frames at actual size and check whether the subject is cropped.
- Place transparent frames over checkerboard, light, and dark backgrounds.
- Confirm transparent frames have no semi-transparent full-frame wash.
- Confirm solid-color videos use a stable background that does not appear inside the subject.
- Confirm no watermark overlaps the subject or expected mask area.
- Confirm loop start and end do not jump sharply unless the interaction intentionally snaps.
- Confirm frame sequence names sort correctly, for example `frame_001.png` to `frame_120.png`.
- Confirm multiple actions share a consistent subject scale and anchor.

## Acceptance Result Format

When the user supplies assets, summarize with:

```text
素材检查结果：
- 可直接使用：
- 建议重新生成：
- 可由代码修正：
- 需要用户确认：
```

Keep the feedback practical. If regeneration is better than complex code cleanup, say so and give
an improved prompt.

## Conversion Notes

- Extract PNGs from video only once. After a follow sequence has clean alpha, store it as PNGs
  and never chroma-key it again in app code.
- If a source has a provider watermark, prefer regenerating without watermark. If masking is
  unavoidable, make the mask per asset and disable it for poses that overlap the mask.
- For fuzzy green edge cleanup, reduce green channel intensity near edges. Do not lower alpha
  unless the pixel is confidently background.
