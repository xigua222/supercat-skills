# Supercat-skills —— Your Desktop Pet Factory

English | [简体中文](README.zh.md)

[![GitHub stars](https://img.shields.io/github/stars/xigua222/supercat-skills?style=social)](https://github.com/xigua222/supercat-skills/stargazers)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![AI Skill](https://img.shields.io/badge/AI%20Skill-Compatible-green)](https://github.com/xigua222/supercat-skills)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-yellow)](https://github.com/xigua222/supercat-skills)
[![OpenAI Codex](https://img.shields.io/badge/OpenAI%20Codex-Compatible-8A2BE2)](https://github.com/xigua222/supercat-skills)
[![Cursor](https://img.shields.io/badge/Cursor-Compatible-black)](https://github.com/xigua222/supercat-skills)

> One sentence to let AI build you a cat that lives inside your screen.

Supercat-skills is an AI Agent Skill that lets you **build a desktop pet from scratch using natural language**. Hand over a few assets and let AI do the rest: inspect media, write code, spin up a preview, and package everything into a desktop app — all without touching a config file.

---

## Why Supercat-skills?

Want a companion that strolls along your screen edges, chases your cursor, and purrs when you drag it around?

The hard way: learn Tauri, wrestle with transparent windows, write frame animation loops, tune chroma keys, handle multi-monitor bounds...

**The Supercat-skills way:** say "make me a desktop cat" and the AI walks through the entire engineering pipeline for you.

---

## What It Can Do

| Capability | Description |
|------------|-------------|
| **Asset Guide** | Tells you what to draw, how to name files, and which formats to export; auto-checks transparency and loop quality |
| **Browser-first Preview** | See your pet in action inside a browser before ever touching a desktop build |
| **Transparent Window** | Borderless, frameless, truly "floating" pets with click-through and drag support |
| **Mouse Interaction** | Cursor follow, click feedback, drag-and-idle, multi-state switching — all supported |
| **One-click Packaging** | Confirmed? The AI builds installables for Windows, macOS, and Linux |
| **Persistent Settings** | Scale, behavior toggles, and position preferences survive app restarts |

---

## Installation

### Option 1: One-liner (Recommended)

```bash
npx skills add https://github.com/xigua222/supercat-skills --skill supercat-skills
```

### Option 2: Paste This to Your AI

Install the skill `supercat-skills` for me. Follow these steps:

1. Detect which AI Agent / IDE you are running in, then find its skills directory (e.g., `~/.claude/skills/` for Claude Code, `~/.agents/skills/` for Trae, or the path configured in your IDE settings)
2. Create the directory if it doesn't exist, then run `git clone https://github.com/xigua222/supercat-skills.git <SKILLS_DIR>/supercat-skills`
3. Verify: `<SKILLS_DIR>/supercat-skills/` should contain `SKILL.md`, `agents/`, and `references/`
4. Tell me it's ready; after that, phrases like "build me a desktop pet" should trigger this skill

Copy and paste the above into Claude Code, Cursor, or any AI Agent with shell access and it will install automatically.

### Option 3: Manual CLI

```bash
# Replace <SKILLS_DIR> with your agent's actual skills path
git clone https://github.com/xigua222/supercat-skills.git <SKILLS_DIR>/supercat-skills
```

> Common skill paths by agent: `~/.claude/skills/` (Claude Code), `~/.agents/skills/` (Trae), or the directory configured in your IDE's skill settings.

---

## Quick Start

Once installed, describe your pet in plain text, for example:

> "I want a black-and-white cow cat that follows my mouse and licks its paws when idle. Tell me what assets I need."

Supercat-skills immediately gives you an **asset checklist + generation prompts** that you can feed to Midjourney, 即梦 (Jimeng), 豆包 (Doubao), or any image/video generator.

After you drop the assets back into the chat, it will:

1. **QA** — right dimensions? clean alpha channel? correct naming?
2. **Develop** — scaffold a complete Vue + Tauri project automatically
3. **Preview** — start a local dev server so you can see it in the browser first
4. **Deliver** — upon your approval, package it into `.exe` / `.dmg` / `.AppImage`

---

## Example: From Zero to Desktop Cat

```
You: I want a desktop pet
AI: Suggest 4 animation sets: idle, walk, lick, sleep.
      8-12 frames each, transparent PNGs, named like idle_01.png ~ idle_12.png.
      Here are ready-to-use prompts for Jimeng...

You: (drops assets)
AI: Inspected. Frame 7 of the walk sequence has white fringing; re-export recommended. The rest pass.
      Now writing code. Preview link in 3 minutes.

You: Preview looks good, but it walks too fast
AI: Walk frame rate reduced. Recompiling...

You: Good to go, package it
AI: Windows installer generated: supercat-pet_0.1.0_x64-setup.exe
```

---

## Tech Stack

- **Frontend**: Vue 3 + TypeScript
- **Desktop**: Tauri (Rust-powered, tiny bundle size, low memory footprint)
- **Preview**: Vite HMR, instant browser preview
- **Windowing**: Transparent background, hit-test ignoring, always-on-top, multi-monitor aware

---

## Project Structure

```
supercat-skills/
├── SKILL.md                      # Core skill logic and workflow definition
├── agents/
│   └── openai.yaml               # Agent interface and trigger config
└── references/
    ├── asset-generation.md       # Asset specs, prompt templates, QA checklist
    ├── from-zero-build.md        # Complete guide from empty folder to runnable app
    ├── implementation-patterns.md # Mouse follow, drag, settings panel patterns
    └── code-recipes.md           # Copy-paste Vue/Tauri snippets
```

---

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=xigua222/supercat-skills&type=Date)](https://star-history.com/#xigua222/supercat-skills&Date)

---

## Contributing

Got ideas? Found a bug? Built an awesome pet and want to show it off?

- Open an [Issue](https://github.com/xigua222/supercat-skills/issues)
- Send a [PR](https://github.com/xigua222/supercat-skills/pulls)
- Leave a star and help more people adopt desktop pets

---

## License

MIT — use freely, modify boldly, and share the pets you create.
