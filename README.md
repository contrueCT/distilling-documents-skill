# distilling-documents

[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-compatible-2E8B57)](https://agentskills.io/)

`distilling-documents` 是一个面向中文长文的高保真信息蒸馏 Skill。它把文章、技术文档、研究报告和资料整理成一份更短、更容易理解、以后还能回来查的替代稿，目标不是追求尽可能高的压缩率，而是尽量减少「读精炼稿的成本 + 以后回看原文的成本」。

默认情况下，用户不需要先说明阅读目的、目标长度或压缩比例。Skill 会自己判断材料类型、保留值得长期留下的信息，并只交付最终精炼稿。

## 为什么要用它？

普通摘要很容易把一篇有用的材料压成几个抽象结论，真正需要数字、条件、原因或边界时还是得回原文找。

| 常见做法 | 容易出现的问题 | distilling-documents 的处理方式 |
| --- | --- | --- |
| 只保留“核心观点” | 数字、版本、阈值、License 条件等未来可能要查的信息被删掉 | 默认按“通用替代稿”处理，未来查询价值也算价值 |
| 为了短，直接删解释 | 最后只剩“门槛高”“能力强”这类无法重新推理的结论 | 尽量保留能重新推出结论的事实、数字、条件和关系 |
| 一段对应一个 bullet | 信息仍然分散，读者需要自己拼因果关系 | 打破原文顺序，把相关内容合并成可独立理解的信息包 |
| 所有材料套同一种摘要模板 | 技术文档、分享文章和资料整理读起来都像同一份 AI 摘要 | 先判断内容功能，再决定结构、语体和信息密度 |
| 把“说人话”理解成口语化 | 正式文档被改得像聊天，专业结构和术语被破坏 | 根据文体保持合适的正式程度，只处理生硬、空泛和模型腔表达 |
| 为保证质量生成大量中间步骤 | 处理一篇文档反而更费事，还需要用户不断确认 | 默认零交互，两遍完成，中间账本和检查过程不对用户展示 |

## 核心设计

- **替代原文优先**：没有指定用途时，默认假设原文以后不会再读，输出要同时承担理解、回忆和查询作用。
- **MERGE → COMPRESS → DROP**：优先通过重组和降低解释粒度提高密度，真正删除信息保持保守。
- **保留可重新推理的信息**：事实、数字、条件和因果通常比二次概括更有长期价值。
- **认识状态保真**：官方确认、来源声称、估算、作者判断、推测和预测不能在精炼后互相升级。
- **内容类型路由**：技术/设计文档保留结构和术语；分享文章保留来龙去脉；信息资料强调密度和可查询性；教程保留步骤和必要解释。
- **自然中文，但不统一口语化**：表达层参考 `human-writing` 的中文改稿思路，并针对文档精炼做了适配。
- **默认零交互**：普通任务不追问压缩比例，不展示信息账本、删除清单、评分或 Probe 结果。

```mermaid
flowchart LR
    A["原始材料"] --> B["Pass 1：高保真蒸馏"]
    B --> C["识别事实 / 数字 / 条件 / 因果 / 判断"]
    C --> D["MERGE / COMPRESS / DROP"]
    D --> E["按语义重组信息包"]
    E --> F["Pass 2：文体改写 + 对照原文校验"]
    F --> G["最终精炼稿"]
```

## 信息怎么保留？

Skill 不会简单给每句话打“重要 / 不重要”标签，而是看一条信息在替代稿里还承担什么作用。

| 信息类型 | 默认处理 |
| --- | --- |
| 关键数字、阈值、版本、价格、License 条件 | 保留精度，KEEP 或 MERGE |
| 支撑重要结论的原因、证据和条件 | 与结论一起保留 |
| 很长的背景或机制解释 | 保留必要含义，COMPRESS |
| 多处重复同一个事实 | MERGE |
| 铺垫、过渡、同义重复 | 没有独立价值时 DROP |
| 作者推测、估算、预测 | 保留原来的不确定性 |
| 反例、限制和负面证据 | 只要会影响整体判断，就不能为了简短而删掉 |

一个好的精炼单元通常会把主信息、必要数据、关系和边界放在一起。例如，与其只留下“模型部署门槛较高”，更倾向于保留“权重有多大、现有硬件为什么装不下、因此得出什么结论”。

## 两遍处理

### Pass 1：高保真蒸馏

1. 自动判断材料类型和原本承担的功能。
2. 找出关键事实、数字、时间/版本、定义、机制、因果、条件、边界、不确定性、证据、反例和重要判断。
3. 打破原文顺序，按语义关系聚类。
4. 优先合并相关信息，再压缩解释，最后才删除确认没有替代价值的内容。
5. 对关键内容保留内部来源锚点。默认不显示，用户需要追溯时再提供。

### Pass 2：改写与原文校验

1. 根据材料类型选择最终结构和语体。
2. 使用包内的自然中文规则处理生硬翻译腔、抽象套话、机械连接和模型腔，同时保留技术文档需要的结构、术语和正式程度。
3. 对照原文检查关键数字、条件、反例和不确定性有没有丢失，检查是否出现原文没有的新事实。
4. 发现问题直接修正，不把审核工作重新交给用户。

普通材料默认到这里结束。超长或高密度材料可以自动分块处理，但分块结果只作为临时状态，最终仍只交付一份全局精炼稿。

## 适合处理的材料

- 技术设计文档、方案文档、架构说明
- AI / 科技 / 行业分享文章
- 新闻、产品发布、信息资料
- 研究报告、论文解读、调研材料
- 教程、说明文档、长篇知识资料
- 任何“内容有价值，但完整回看成本太高”的中文材料

如果用户已经给出明确目标，例如“只关心部署部分”“压到 500 字”“给管理层看”，这些目标会覆盖默认的通用替代稿策略。

## Install

安装 Skill：

```bash
npx skills add contrueCT/distilling-documents-skill
```

也可以全局安装到 Codex 和 Claude Code：

```bash
npx skills add contrueCT/distilling-documents-skill -g -a codex -a claude-code
```

## Use

直接给材料并要求精炼即可：

```text
Use $distilling-documents to distill this report into a compact Chinese replacement I can rely on later without rereading the source.
```

也可以明确指定用途：

```text
Use $distilling-documents to distill this technical design doc. Preserve implementation constraints, important trade-offs, and anything I may need to look up later.
```

支持自动选择 Skill 的 Agent，也可以在匹配到长文精炼、文档压缩、资料提炼等请求时自动加载它。

## 中文表达层

仓库内置了 `references/human-writing-adapted.md`，用于精炼后的中文改写。它参考了 [KKKKhazix/human-writing](https://github.com/KKKKhazix/human-writing) 中适合现实材料、自然中文和改稿的规则，并针对文档精炼做了裁剪和适配，因此不要求运行环境额外安装 `human-writing`。

适配层只负责“怎么写得自然”，不重新决定“哪些信息应该保留”。第三方来源和 MIT 许可说明见 [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md)。

## Compatibility

仓库遵循 [Agent Skills](https://agentskills.io/) 的标准布局。核心流程和 references 都是 Markdown，可用于 Codex、Claude Code 以及其他支持 Agent Skills 或自定义 Skill 的 Agent。

正常使用只需要 `SKILL.md` 与 `references/`。`evals/cases.md` 用于测试和回归检查，不参与正常输出。

## Repository structure

```text
distilling-documents-skill/
├── SKILL.md
├── THIRD_PARTY_NOTICES.md
├── references/
│   ├── distillation-rules.md
│   ├── human-writing-adapted.md
│   └── routing-and-rewrite.md
├── evals/
│   └── cases.md
└── README.md
```
