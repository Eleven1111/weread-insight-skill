# WeRead Insight Skill

[简体中文](README.zh-CN.md)

WeRead Insight Skill is a Codex skill for explaining highlights and notes from WeRead.

It is not a note-export tool. Its job is to turn a moment of reading friction into a reusable understanding card: what the passage says, why it may be confusing, what the source text and context are, where common misreadings come from, and how to remember the point.

```text
WeRead highlight -> reader question -> source check -> explanation -> Obsidian note
```

## Overview

When you highlight a passage, you are usually marking a problem, not just saving a sentence. The problem may be explicit, as in a note you wrote yourself, or implicit, as in several adjacent highlights around the same concept.

WeRead Insight treats each highlight as evidence. It reads the surrounding reading signals, infers the likely question, verifies source text when needed, and writes a concise explanation that can be saved to Obsidian.

Use it for questions like:

```text
How should I understand this sentence?
What does Russell say in the original text?
Is this translation misleading?
Look at my highlights from the last week. What questions am I probably struggling with?
Explain the most important highlight from today and save it as an Obsidian card.
```

## What the Skill Does

WeRead Insight runs a five-step workflow.

1. **Locate the passage**

   It uses the companion `微信读书` skill to find the relevant book, chapter, highlight, user note, and recent reading context. If the current conversation already contains the passage, it uses that first.

2. **Infer the question**

   It distinguishes between what the user explicitly asked, what their annotation says, and what can only be inferred from repeated or adjacent highlights. Low-confidence inferences are labeled as such.

3. **Verify source text**

   For translated works, classics, quoted material, or suspicious wording, it looks for the original wording and surrounding context. It should not reconstruct original text from a translation.

4. **Explain the passage**

   The explanation focuses on the passage's role in the author's argument: literal meaning, context, function in the argument, and likely misreadings.

5. **Persist the result**

   If configured, it saves the result as an Obsidian note. The note stores the user's question, a short quote, links, source notes, and the explanation. It does not store long copyrighted excerpts.

## Requirements

| Requirement | Required | Why it is needed |
| --- | --- | --- |
| Codex local skills | Yes | Loads `SKILL.md` and makes the skill available in Codex. |
| `微信读书` skill | Yes | Provides the WeRead API workflow used to fetch highlights, notes, reading statistics, and chapter data. |
| `WEREAD_API_KEY` | Yes | Authenticates requests to the WeRead gateway. |
| Network access | Usually | Needed for WeRead API calls and for checking original text or external source references. |
| Obsidian vault | Optional | Stores explanation cards, daily summaries, and unresolved questions. |
| Codex automations | Optional | Runs a daily review, for example every night at 22:00. |

### Companion Skill Dependency

WeRead Insight depends on the companion `微信读书` skill. It expects that skill to expose these workflows:

| WeRead capability | Used for |
| --- | --- |
| `/user/notebooks` | Find books with recent notes or highlights. |
| `/book/bookmarklist` | Fetch highlight text, chapter IDs, ranges, and chapter metadata. |
| `/review/list/mine` | Fetch the user's own comments, ideas, and book notes. |
| `/book/chapterinfo` | Resolve chapter structure and chapter titles. |
| `/readdata/detail` | Summarize recent reading activity. |

If `微信读书` is not installed or the API key is not configured, WeRead Insight can still explain text pasted into the conversation, but it cannot fetch the user's reading data.

## Installation

Copy the repository into your Codex skills directory:

```bash
mkdir -p "$HOME/.codex/skills/weread-insight-skill"
rsync -a ./ "$HOME/.codex/skills/weread-insight-skill/"
```

Restart Codex, or open a new session, so the skill list is refreshed.

## Configuration

Set the WeRead API key in the environment that Codex can read:

```bash
export WEREAD_API_KEY="wrk-..."
```

If you want WeRead Insight to write Obsidian notes, also configure a vault path:

```bash
export OBSIDIAN_VAULT="/path/to/your/obsidian/vault"
```

Do not commit these values. This repository uses placeholders only.

## Obsidian Output

The recommended vault layout is:

```text
<OBSIDIAN_VAULT>/微信读书/阅读助手/
  阅读助手索引.md
  未解问题.md
  Daily/
    YYYY-MM-DD.md
  Cards/
    YYYY-MM-DD-书名-主题.md
```

### Cards

Cards are for a single passage or a single question. They follow [templates/obsidian-card.md](templates/obsidian-card.md) and include:

- book, author, chapter, and date
- the user's question
- a short highlight quote
- original wording and source notes, when available
- explanation and likely misreadings
- a short memory sentence
- follow-up questions
- a backlink to the matching daily note

### Daily Notes

Daily notes are for lightweight review. They follow [templates/obsidian-daily.md](templates/obsidian-daily.md) and include:

- what was read today
- new highlights or comments
- likely questions worth unpacking
- links to cards generated that day

Daily notes and cards are intentionally bidirectional: a daily note links to each card created from that day's reading, and every card links back to the daily note. This keeps review, triage, and later cleanup navigable inside Obsidian.

## Automation

A useful automation is a nightly review:

```text
Every day at 22:00:
  Use Reading Assistant to check whether today's WeRead activity contains new highlights, notes, or comments.
  If yes, write or append an Obsidian daily note.
  Then ask whether the user wants help unpacking the most important question.
  If no, do not send a substantive message unless the automation runtime requires one.
```

Keep the automation selective. A good nightly run should identify one or two high-value questions, not produce a long digest of every highlight.

## Data and Privacy Boundaries

Reading Assistant is designed to save understanding, not copyrighted book content.

It should save:

- short necessary quotes
- WeRead position links
- source links
- the user's question
- the generated explanation
- unresolved follow-up questions

It should not save:

- full chapters
- long passages from copyrighted books
- bulk exports of all highlights
- API keys
- local runtime logs
- private Obsidian notes
- machine-specific absolute paths

When the original text cannot be verified, the note should say so directly.

## Repository Layout

```text
.
├── SKILL.md
├── README.md
├── README.zh-CN.md
├── evals/
│   └── evals.json
└── templates/
    ├── obsidian-card.md
    └── obsidian-daily.md
```

The repository intentionally excludes generated notes, personal vault content, runtime state, and local setup files.

## Example Sessions

### Translation Check

```text
How should I understand this sentence: "repentance was itself a form of passion"?
```

Expected behavior:

- identify the book and passage
- verify the original wording
- explain the translation issue, if any
- use the surrounding argument
- avoid overclaiming the author's position

### Recent Confusion Map

```text
Look at my WeRead highlights from the last week. What questions am I probably struggling with?
```

Expected behavior:

- state the date range and data limitations
- group highlights by theme
- separate evidence-backed conclusions from guesses
- suggest the next questions to unpack

### Save a Card

```text
Explain the most important highlight from today and save it to Obsidian.
```

Expected behavior:

- choose a high-signal highlight
- explain it concisely
- save a card under `Cards/`
- report the saved path

## Troubleshooting

**The skill does not trigger**

Restart Codex or open a new session after copying the skill into `$HOME/.codex/skills/weread-insight-skill`.

**It cannot fetch WeRead data**

Check that the companion `微信读书` skill is installed and that `WEREAD_API_KEY` is visible to the Codex runtime.

**It explains pasted text but does not save notes**

Set `OBSIDIAN_VAULT`, or configure a local private path in your own environment. Do not commit the actual vault path to this repository.

**It cannot verify the original wording**

The skill should still explain the available translation, but the answer and any saved note should mark the original source as unverified.

## Evaluation Prompts

[evals/evals.json](evals/evals.json) contains smoke prompts for:

- recent highlight analysis
- original-text verification
- Obsidian card generation
- daily automation behavior

These prompts are not exhaustive tests. They define the behavior boundaries the skill should preserve as it evolves.
