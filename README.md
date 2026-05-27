# 阅读助手 Skill

阅读助手是一个 Codex skill，用来把微信读书里的划线、批注和最近阅读状态转化成可追踪的理解卡片。它的重点不是导出笔记，而是帮助用户定位当前困惑、查证原文和上下文、解释容易误读的地方，并把解惑结果保存到 Obsidian。

## What It Does

- 从微信读书抓取最近阅读、划线和个人想法。
- 根据连续划线、批注和阅读主题推断用户可能卡住的问题。
- 对翻译书或经典文本查找原文和上下文，避免凭记忆反推原文。
- 输出结构化解释：结论、原文/译法、上下文、怎么理解、避免误读、可以这样记。
- 将实质解惑内容保存为 Obsidian Markdown 卡片或每日摘要。

## Repository Contents

```text
.
├── SKILL.md
├── README.md
├── evals/
│   └── evals.json
└── templates/
    ├── obsidian-card.md
    └── obsidian-daily.md
```

Only the reusable skill files are included. Runtime logs, local automation state, personal Obsidian notes, API keys, and generated reading cards are intentionally excluded.

## Requirements

- Codex with local skill support.
- The companion `微信读书` skill installed and configured.
- `WEREAD_API_KEY` available in the runtime environment for WeRead API access.
- Optional: an Obsidian vault for saving cards and daily notes.

## Installation

Copy this directory into your Codex skills folder, for example:

```bash
mkdir -p "$HOME/.codex/skills/reading-assistant"
rsync -a ./ "$HOME/.codex/skills/reading-assistant/"
```

Restart Codex or open a new session if the skill list does not refresh automatically.

## Obsidian Setup

By default, the skill expects an Obsidian vault path to be configured by the user or calling environment. Recommended convention:

```text
<OBSIDIAN_VAULT>/微信读书/阅读助手/
  阅读助手索引.md
  未解问题.md
  Daily/
  Cards/
```

Do not hardcode personal machine paths in published copies of the skill. Use a local environment variable, local wrapper, or private fork if a fixed vault path is needed.

## Output Policy

The skill is designed to avoid publishing or storing excessive copyrighted text:

- Save only necessary short quotes, location links, source links, and original analysis.
- Do not bulk export book text or entire highlight collections.
- Mark original-source gaps honestly when the source cannot be verified.
- Separate author intent, translation judgment, model inference, and the user's actual question.

## Example Prompts

- `用阅读助手看看我最近一周在微信读书划了什么，并判断我最近可能在困惑哪些问题。`
- `这句“忏悔本身就是快事一桩”怎么理解？罗素原文怎么说，翻译有没有问题？`
- `把我在《西方哲学史》最近的划线整理成一份理解卡片，重点解释容易误读的地方。`
- `用阅读助手解释我今天最值得解的一条划线，并保存到 Obsidian。`

## Evaluation Prompts

See [evals/evals.json](evals/evals.json) for smoke tests covering:

- Recent WeRead highlight analysis.
- Original-text and translation verification.
- Obsidian card generation.
- Daily automation summary behavior.

## Privacy Notes

This repository should not contain:

- Personal API keys.
- Personal Obsidian vault contents.
- Generated reading cards or daily notes.
- Local Codex/OMX runtime logs.
- Machine-specific absolute paths.
