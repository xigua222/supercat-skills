# Supercat-skills —— 你的桌面宠物制造机

[English](README.md) | 简体中文

[![GitHub stars](https://img.shields.io/github/stars/xigua222/supercat-skills?style=social)](https://github.com/xigua222/supercat-skills/stargazers)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![AI Skill](https://img.shields.io/badge/AI%20Skill-Compatible-green)](https://github.com/xigua222/supercat-skills)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-yellow)](https://github.com/xigua222/supercat-skills)
[![OpenAI Codex](https://img.shields.io/badge/OpenAI%20Codex-Compatible-8A2BE2)](https://github.com/xigua222/supercat-skills)
[![Cursor](https://img.shields.io/badge/Cursor-Compatible-black)](https://github.com/xigua222/supercat-skills)

> 一句话让 AI 帮你造一只住在屏幕里的猫。

Supercat-skills 是一个专为 AI Agent 设计的 Skill，让你**用自然语言就能从零打造一只桌面宠物**。提供几张素材，剩下的交给 AI：它会帮你检查素材、写代码、跑预览、打包成桌面应用——全程你只需要动动嘴（或者键盘）。

---

## 为什么需要 Supercat-skills？

想做一只会在屏幕边缘散步、追着鼠标跑、被你拖拽还撒娇的桌面宠物？

传统方式：你要学 Tauri、折腾透明窗口、写帧动画循环、调绿幕抠图、处理多显示器适配……

**有了 Supercat-skills**：你发一句"帮我做一只桌面猫咪"，AI 自动走完整个工程化流程。

---

## 它能做什么

| 能力 | 说明 |
|------|------|
| **素材管家** | 告诉你该画什么、怎么命名、导出成什么格式，自动检查透明度和循环质量 |
| **浏览器预览** | 不用等打包，先在网页里看到宠物跑起来的样子，不满意就改 |
| **透明窗口** | 无边框、去标题栏、真正"浮"在桌面上的宠物，支持鼠标穿透和拖拽 |
| **鼠标交互** | 跟随光标、点击反馈、拖拽停留、多状态切换，全都能实现 |
| **一键打包** | 确认效果后自动构建为 Windows / macOS / Linux 安装包 |
| **设置记忆** | 宠物大小、行为开关、位置偏好，重启后照样记得 |

---

## 安装

### 方式一：一行命令安装（推荐）

```bash
npx skills add https://github.com/xigua222/supercat-skills --skill supercat-skills
```

### 方式二：把下面这段话直接发给 AI

帮我安装 supercat-skills 这个 skill。请按下面步骤做：

1. 确保 `~/.claude/skills/` 目录存在（不存在就创建）
2. 执行 `git clone https://github.com/xigua222/supercat-skills.git ~/.claude/skills/supercat-skills`
3. 验证：`ls ~/.claude/skills/supercat-skills/` 应该看到 `SKILL.md`、`agents/`、`references/` 三项
4. 告诉我安装好了，之后我说"帮我做一只桌面宠物"之类的话就会触发这个 skill

把这段话复制粘贴给 Claude Code / Cursor / 任何有 shell 权限的 AI Agent，它会自动完成安装。

### 方式三：手动命令行

```bash
git clone https://github.com/xigua222/supercat-skills.git ~/.claude/skills/supercat-skills
```

> 不同 IDE 的 Skill 目录可能不同，常见路径：`~/.claude/skills/`、`~/.agents/skills/` 或 `~/.cursor/skills/`，请根据你使用的工具调整。

---

## 快速开始

安装完成后，在 Agent 对话中直接描述你的桌面宠物需求，例如：

> "我要一只黑白奶牛猫，能跟着鼠标跑，空闲的时候舔爪子，你告诉我需要什么素材。"

Supercat-skills 会立刻给你一份**素材清单 + 生成提示词**，你可以直接丢给豆包、即梦、Midjourney 等工具生成图片。

把素材丢回对话框后，它会：

1. **质检** —— 尺寸对不对？透明通道干净吗？命名规范吗？
2. **开发** —— 自动写出 Vue + Tauri 的完整项目代码
3. **预览** —— 启动本地服务器，你在浏览器里先看效果
4. **交付** —— 你点个头，它就打包成 `.exe` / `.dmg` / `.AppImage`

---

## 示例：从 0 到一只桌面猫

```
你：想做桌面宠物
AI：建议做 4 组动画：idle（待机）、walk（行走）、lick（舔毛）、sleep（睡觉）。
      每组 8-12 帧，透明 PNG，命名格式：idle_01.png ~ idle_12.png。
      以下是即梦可用的提示词……

你：（丢素材）
AI：已检查，walk 序列第 7 帧边缘有白边，建议重导。其余通过。
      现在开始写代码，3 分钟后给预览链接。

你：预览不错，就是走得太快
AI：已降低 walk 动画帧率，重新编译中……

你：可以了，打包吧
AI：Windows 安装包已生成：supercat-pet_0.1.0_x64-setup.exe
```

---

## 技术栈

- **前端**：Vue 3 + TypeScript
- **桌面端**：Tauri（Rust 驱动，包体极小，内存占用低）
- **预览**：Vite 热更新，浏览器秒开
- **窗口**：透明背景、忽略鼠标事件、始终置顶、多显示器感知

---

## 项目结构

```
supercat-skills/
├── SKILL.md                      # Skill 核心逻辑与工作流定义
├── agents/
│   └── openai.yaml               # Agent 接口与触发配置
└── references/
    ├── asset-generation.md       # 素材规范、提示词模板、QA checklist
    ├── from-zero-build.md        # 从空文件夹到可运行项目的完整指南
    ├── implementation-patterns.md # 鼠标跟随/拖拽/设置面板等交互模式
    └── code-recipes.md           # 可直接复制的 Vue/Tauri 代码片段
```

---

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=xigua222/supercat-skills&type=Date)](https://star-history.com/#xigua222/supercat-skills&Date)

---

## 贡献与支持

有想法？发现 bug？或者造了一只超酷的宠物想秀一下？

- 提交 [Issue](https://github.com/xigua222/supercat-skills/issues) 反馈问题
- 提交 [PR](https://github.com/xigua222/supercat-skills/pulls) 完善 Skill
- 点个 Star，让更多人养上桌面宠物

---

## License

MIT —— 随便用，随便改，造出有趣的宠物记得分享。
