# 阅读助手

把“我划了这句话”变成“我到底卡在哪里，以及这段该怎么读”。

`阅读助手` 是一个 Codex skill，工作对象是微信读书里的划线、批注和最近阅读记录。它不做大规模摘抄，也不把书摘搬进仓库；它做的是一条更窄、更有用的链路：

```text
微信读书划线/批注 -> 判断困惑 -> 查原文和上下文 -> 解释误读点 -> 保存理解卡片
```

典型场景：

- 你划了一句翻译腔很重的话，想知道原文到底怎么说。
- 你最近在同一章连续划线，想让 Codex 判断你可能在追问什么问题。
- 你每天读完书后，希望晚上自动生成一个“今天值得解惑的地方”。
- 你想把真正解释清楚的内容沉淀到 Obsidian，而不是留在一次性聊天里。

## 这个 Skill 做什么

**1. 从微信读书拿证据**

它会通过 companion `微信读书` skill 读取书籍、章节、划线、个人想法和最近阅读统计。它不会假设你在读什么，也不会让你重复输入已经能从记录里查到的信息。

**2. 判断你可能卡住的问题**

它会看三类信号：你直接问的问题、你写在划线上的批注、以及同一章节/同一主题下的连续划线。输出时会区分“明确证据”和“低置信推断”。

**3. 查原文和上下文**

遇到翻译书、经典文本、引文、术语或可疑译法时，它会优先找可靠原文来源。找不到时必须说找不到，不能把译文反推成原文。

**4. 解释，而不是复述**

默认解释结构是：

- 结论：先给一句话判断。
- 原文/译法：指出关键词和译法偏移。
- 上下文：说明作者这一段在解决什么问题。
- 怎么理解：拆开论证关系。
- 避免误读：指出哪些理解太强、太弱或方向错了。
- 可以这样记：沉淀成一句可复用理解。

**5. 保存到 Obsidian**

实质解惑、每日摘要和未解问题可以保存为 Obsidian Markdown。保存的是理解、来源和短引文，不是批量书摘。

## 依赖关系

| 依赖 | 是否必需 | 用途 | 配置方式 |
| --- | --- | --- | --- |
| Codex local skills | 必需 | 让 Codex 识别并运行 `SKILL.md` | 把本仓库复制到 `$HOME/.codex/skills/reading-assistant` |
| `微信读书` skill | 必需 | 调用微信读书接口，读取划线、批注、阅读统计、章节信息 | 先安装并验证 companion skill |
| `WEREAD_API_KEY` | 必需 | 微信读书 API 鉴权 | 放入 Codex 可读取的环境变量 |
| 网络访问 | 按任务必需 | 调微信读书接口；查证原文、版本和引用来源 | 运行环境需要允许对应请求 |
| Obsidian vault | 可选但推荐 | 保存解惑卡片、日报和未解问题 | 设置 `OBSIDIAN_VAULT` 或在本地私有配置里指定 vault |
| Codex automations | 可选 | 每天固定时间检查新划线并提醒是否解惑 | 在 Codex app 中创建 heartbeat/cron 自动化 |

### Companion `微信读书` Skill

`阅读助手` 自己不直接定义微信读书 API。它依赖 companion `微信读书` skill 提供这些能力：

- `/user/notebooks`：找到最近有笔记的书。
- `/book/bookmarklist`：获取单本书划线和章节信息。
- `/review/list/mine`：获取个人想法、点评和批注。
- `/book/chapterinfo`：读取章节目录。
- `/readdata/detail`：读取本周、本月、本年等阅读统计。

如果 companion skill 没装好，`阅读助手` 只能解释用户手动贴出的文本，不能主动读取微信读书数据。

### Environment Variables

```bash
export WEREAD_API_KEY="wrk-..."
export OBSIDIAN_VAULT="/path/to/your/obsidian/vault" # optional
```

不要把这些值提交到 GitHub。本仓库只保留占位符和说明。

## 安装

```bash
mkdir -p "$HOME/.codex/skills/reading-assistant"
rsync -a ./ "$HOME/.codex/skills/reading-assistant/"
```

然后重启 Codex，或开启一个新会话，让 skill 列表重新加载。

安装后可以这样试：

```text
用阅读助手看看我最近一周在微信读书划了什么，并判断我最近可能在困惑哪些问题。
```

## Obsidian 保存结构

推荐 vault 内目录：

```text
<OBSIDIAN_VAULT>/微信读书/阅读助手/
  阅读助手索引.md
  未解问题.md
  Daily/
    2026-05-27.md
  Cards/
    2026-05-27-西方哲学史-忏悔与激情.md
```

`Cards/` 保存单条问题的理解卡片。  
`Daily/` 保存每日阅读摘要。  
`未解问题.md` 保存需要继续查证、比较译本或等后文验证的问题。

卡片模板见 [templates/obsidian-card.md](templates/obsidian-card.md)。日报模板见 [templates/obsidian-daily.md](templates/obsidian-daily.md)。

## 定时任务建议

可以在 Codex app 里创建一个每天晚上运行的自动化：

```text
每天 22:00 使用 阅读助手 检查今天微信读书是否有新划线、批注或想法。
如果有，生成 Obsidian 日报，并在当前线程提醒我是否需要解惑。
如果没有，不主动打扰。
```

这类自动化最好只提醒“今天最值得解的 1-2 个问题”，不要每天生成一大堆低价值总结。

## 仓库内容

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

没有放进仓库的内容：

- 个人 API key。
- 个人 Obsidian vault 内容。
- 生成的阅读卡片和日报。
- Codex/OMX runtime 日志。
- 本机绝对路径。
- 微信读书导出的完整书摘。

## 隐私和版权边界

`阅读助手` 的默认策略是“保存理解，不保存书”：

- 可以保存必要短引文、WeRead 位置链接、来源链接、解释和结论。
- 不批量导出整本书划线。
- 不保存长段电子书正文。
- 找不到原文来源时标注“待确认”，不伪造引用。
- 区分作者原意、译者选择、模型推断和用户实际问题。

## 示例

**翻译核对**

```text
这句“忏悔本身就是快事一桩”怎么理解？罗素原文怎么说，翻译有没有问题？
```

期望行为：查到原文，指出关键词的译法偏移，解释上下文，并避免把罗素的意思粗暴概括成“忏悔是不道德的”。

**最近困惑画像**

```text
用阅读助手看看我最近一周的划线，判断我可能在困惑哪些问题。
```

期望行为：先说明数据口径，再按主题聚类，不把低置信猜测说成确定结论。

**保存理解卡片**

```text
用阅读助手解释我今天最值得解的一条划线，并保存到 Obsidian。
```

期望行为：生成一张短卡片，写入 configured vault 的 `微信读书/阅读助手/Cards/`。

## 测试提示

[evals/evals.json](evals/evals.json) 里放了几条 smoke eval prompts，覆盖：

- 最近划线分析。
- 原文和译法查证。
- Obsidian 卡片生成。
- 每日自动摘要。

这些不是自动化单元测试，而是用来检查 skill 是否保持正确工作边界的行为样例。
