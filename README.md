# Supercat

Supercat 是一个用于构建桌面宠物（Desktop Pet）应用的 AI Skill。它帮助你通过用户提供的视觉素材，快速构建、调试、预览和打包基于 Tauri/Vue 的桌面宠物应用。

## 功能特性

- **素材驱动**：围绕用户提供的 PNG/WebP 序列帧或绿幕视频构建宠物动画
- **浏览器优先预览**：在打包桌面应用前，先在浏览器中预览交互效果
- **素材质量检查**：自动检查素材的透明度、循环质量、命名规范等
- **透明窗口支持**：支持无边框透明窗口、鼠标跟随、拖拽交互
- **桌面打包**：一键打包为 Windows/macOS/Linux 桌面应用
- **设置持久化**：用户设置自动保存，重启后保留偏好

## 适用场景

- 桌面伴侣 / 虚拟猫咪 / 吉祥物宠物
- 透明窗口交互应用
- 鼠标跟随与点击反馈动画
- 需要素材 QA 和浏览器预览的桌面宠物工作流

## 安装使用

### 前提条件

- 支持 AI Agent / Skill 系统的 IDE（如 Trae、Cursor 等）
- Node.js 18+ 和 Rust 工具链（用于 Tauri 打包）

### 安装步骤

1. 克隆本仓库到本地：

   ```bash
   git clone https://github.com/YOUR_USERNAME/supercat.git
   ```

2. 将 `supercat` 文件夹复制到你的 AI Agent Skills 目录：

   - **Trae**: `~/.agents/skills/` 或 IDE 设置中指定的 Skills 路径
   - **其他 IDE**: 请参考对应 IDE 的 Skill/Plugin 安装文档

3. 重启 IDE，Agent 将自动识别并加载 Supercat Skill。

### 使用方法

在 Agent 对话中直接描述你的桌面宠物需求，例如：

> "帮我做一个会跟着鼠标走的桌面猫咪，我有素材图片。"

Supercat 会引导你完成以下流程：

1. **规划素材**：提供可复用的素材生成提示词（支持豆包、即梦等外部工具）
2. **素材检查**：检查你提供的素材是否符合透明度、帧率、命名等规范
3. **浏览器预览**：先构建可在浏览器中预览的版本，确认效果
4. **桌面打包**：确认无误后，打包为 Tauri 桌面应用

### 项目结构

```
supercat/
├── SKILL.md                      # Skill 核心定义与工作流程
├── agents/
│   └── openai.yaml               # Agent 接口配置
├── references/
│   ├── asset-generation.md       # 素材生成规范与 QA 检查
│   ├── from-zero-build.md        # 从零构建完整项目指南
│   ├── implementation-patterns.md # 交互与桌面端实现模式
│   └── code-recipes.md           # Vue/Tauri 可复用代码片段
```

## 核心原则

- **不重复生成素材**：Skill 不会调用图像/视频生成模型，而是给你提示词，让你在外部工具生成
- **素材未检查不实现**：最终渲染代码只有在素材通过检查后才编写
- **未预览不打包**：浏览器预览通过并得到你确认后，才进入桌面打包阶段
- **保持透明素材纯净**：透明 PNG/WebP 直接绘制，不做额外的绿幕处理或边缘清理

## 贡献

欢迎提交 Issue 和 PR 来改进 Supercat。

## License

MIT
