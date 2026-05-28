# WeRead Insight Skill（阅读助手）

[English](README.md)

WeRead Insight Skill（阅读助手）是一个 Codex skill，用来解释你在微信读书里的划线和批注。

它不是笔记导出工具。它的目标是把一次阅读停顿转化成一张可复用的理解卡片：这句话在说什么，为什么你可能会卡住，原文和上下文是什么，哪些理解容易走偏，最后应该怎样记住它。

```text
微信读书划线 -> 读者问题 -> 原文查证 -> 解释 -> Obsidian 笔记
```

## 概览

当你划下一段话时，你通常不是只在保存一个句子，而是在标记一个问题。这个问题可能很明确，比如你写下了批注；也可能是隐性的，比如你在同一章围绕同一个概念连续划了几处。

WeRead Insight 把每条划线当作证据。它读取相关的阅读信号，推断你可能在追问的问题，在必要时查证原文，并生成一段可以保存到 Obsidian 的简洁解释。

适合这类问题：

```text
这句怎么理解？
罗素原文怎么说？
这个翻译是不是有问题？
看看我最近一周的划线，我可能在困惑什么？
把今天最值得解的一条划线保存成 Obsidian 卡片。
```

## Skill 做什么

WeRead Insight 执行一个五步工作流。

1. **定位文本**

   它通过 companion `微信读书` skill 找到相关书籍、章节、划线、用户批注和最近阅读上下文。如果当前对话里已经有这段文本，则优先使用当前上下文。

2. **推断问题**

   它会区分三件事：用户明确问了什么，用户批注里写了什么，以及只能从连续划线或相邻划线中推断出的内容。低置信推断必须标注为推断。

3. **查证原文**

   对翻译作品、经典文本、引文、术语或可疑译法，它会尝试查找原文和上下文。它不应该从译文反推原文。

4. **解释文本**

   解释重点是这句话在作者论证中的作用：字面意思、上下文、论证功能，以及常见误读。

5. **保存结果**

   如果配置了 Obsidian，它会把解释保存为一条笔记。笔记保存用户问题、必要短引文、链接、来源说明和解释，不保存长段版权文本。

## 依赖

| 依赖 | 是否必需 | 用途 |
| --- | --- | --- |
| Codex local skills | 必需 | 加载 `SKILL.md`，让 Codex 能识别并运行这个 skill。 |
| `微信读书` skill | 必需 | 提供微信读书 API 工作流，用来读取划线、批注、阅读统计和章节数据。 |
| `WEREAD_API_KEY` | 必需 | 调用微信读书 gateway 的鉴权凭证。 |
| 网络访问 | 通常必需 | 调用微信读书 API，以及查证原文或外部来源。 |
| Obsidian vault | 可选但推荐 | 保存解释卡片、每日摘要和未解问题。 |
| Codex automations | 可选 | 做每日定时回顾，例如每天晚上 22:00 运行一次。 |

### Companion Skill 依赖

WeRead Insight 依赖 companion `微信读书` skill。它预期该 skill 提供这些能力：

| 微信读书能力 | 用途 |
| --- | --- |
| `/user/notebooks` | 找到最近有笔记或划线的书。 |
| `/book/bookmarklist` | 获取划线文本、章节 ID、range 和章节元数据。 |
| `/review/list/mine` | 获取用户自己的想法、点评和批注。 |
| `/book/chapterinfo` | 解析章节结构和章节标题。 |
| `/readdata/detail` | 汇总最近阅读活动。 |

如果没有安装 `微信读书` skill，或者没有配置 API key，WeRead Insight 仍然可以解释用户手动贴到对话里的文本，但不能主动读取用户的微信读书数据。

## 安装

把仓库复制到 Codex skills 目录：

```bash
mkdir -p "$HOME/.codex/skills/weread-insight-skill"
rsync -a ./ "$HOME/.codex/skills/weread-insight-skill/"
```

然后重启 Codex，或打开一个新会话，让 skill 列表刷新。

## 配置

把微信读书 API key 放到 Codex 能读取的环境变量里：

```bash
export WEREAD_API_KEY="wrk-..."
```

如果希望 WeRead Insight 写入 Obsidian，再配置 vault 路径：

```bash
export OBSIDIAN_VAULT="/path/to/your/obsidian/vault"
```

不要把这些值提交到仓库。本仓库只使用占位符。

## Obsidian 输出

推荐 vault 结构：

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

Cards 用于保存单条文本或单个问题。结构见 [templates/obsidian-card.md](templates/obsidian-card.md)，包括：

- 书名、作者、章节和日期
- 用户问题
- 必要短引文
- 原文和来源说明，如果可用
- 解释和常见误读
- 一句记忆句
- 后续问题

### Daily Notes

Daily notes 用于轻量回顾。结构见 [templates/obsidian-daily.md](templates/obsidian-daily.md)，包括：

- 今天读了什么
- 新划线或新批注
- 值得展开的问题
- 当天生成的卡片链接

## 自动化

一个实用的自动化是每日夜间回顾：

```text
每天 22:00：
  使用阅读助手检查今天微信读书是否有新划线、批注或想法。
  如果有，写入或追加 Obsidian 日报。
  然后询问用户是否需要展开最重要的问题。
  如果没有，不发送实质消息，除非自动化运行环境要求必须回应。
```

保持自动化克制。好的每日运行应该识别一两个高价值问题，而不是生成一长串划线摘要。

## 数据和隐私边界

阅读助手的设计目标是保存理解，而不是保存版权书籍内容。

它可以保存：

- 必要短引文
- WeRead 位置链接
- 来源链接
- 用户问题
- 生成的解释
- 未解决的后续问题

它不应该保存：

- 完整章节
- 长段版权书籍正文
- 全量划线批量导出
- API key
- 本地运行日志
- 私人 Obsidian 笔记
- 本机绝对路径

如果原文无法查证，笔记中应该直接标明。

## 仓库结构

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

仓库有意排除了生成的笔记、个人 vault 内容、运行状态和本地配置文件。

## 示例

### 翻译核对

```text
这句“忏悔本身就是快事一桩”怎么理解？罗素原文怎么说，翻译有没有问题？
```

预期行为：

- 定位书籍和段落
- 查证原文
- 解释译法问题
- 使用上下文论证
- 避免过度概括作者立场

### 最近困惑画像

```text
用阅读助手看看我最近一周的划线，判断我可能在困惑哪些问题。
```

预期行为：

- 说明日期窗口和数据限制
- 按主题聚类划线
- 区分证据充分的结论和推测
- 建议下一步值得展开的问题

### 保存卡片

```text
用阅读助手解释我今天最值得解的一条划线，并保存到 Obsidian。
```

预期行为：

- 选择一条高信号划线
- 简洁解释
- 保存到 `Cards/`
- 返回保存路径

## 故障排查

**Skill 没有触发**

把 skill 复制到 `$HOME/.codex/skills/weread-insight-skill` 后，重启 Codex 或打开新会话。

**无法读取微信读书数据**

检查 companion `微信读书` skill 是否已安装，以及 `WEREAD_API_KEY` 是否对 Codex 运行环境可见。

**能解释粘贴文本，但不能保存笔记**

设置 `OBSIDIAN_VAULT`，或在本地私有环境中配置 vault 路径。不要把真实 vault 路径提交到仓库。

**无法查证原文**

Skill 仍然可以解释已有译文，但回答和保存的笔记都应标注原文来源未确认。

## 评估提示

[evals/evals.json](evals/evals.json) 包含几条 smoke prompts，覆盖：

- 最近划线分析
- 原文查证
- Obsidian 卡片生成
- 每日自动化行为

这些提示不是完整测试，只是定义 skill 迭代时需要保持的行为边界。
