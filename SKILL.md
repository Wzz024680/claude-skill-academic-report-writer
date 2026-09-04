---
name: academic-report-writer
description: |
  当用户想把研究笔记、阶段性发现、读书心得整理成格式规整的学术报告 Word 文档时使用本技能。
  触发词包括："写报告""生成报告""阶段性报告""读书报告""研究笔记整理成文""导出为Word"，
  或者用户提供研究材料并要求生成格式化的文档。本技能强制执行中文学术写作规范，包括标点
  符号使用、语言风格控制、引用格式、通过 OfficeMCP 写入 Word 时的样式参考、脚注尾注
  格式、参考文献表和附录的编写。
---

# 学术报告写作助手

把研究中的阶段性发现快速转化为格式规整、可直接阅读的 Word 文档。

## 工作流程

```
确认需求 → 撰写/接收 MD → 子智能体审核(多轮至14/14通过) → 用户确认 → 写入 Word → 完成确认
```

### 流程规则

1. **确认需求**：向用户确认报告类型、主题、素材、字数、参考文献/附录、项目名、作者信息、保存路径。若用户直接提供已有 MD 且声明"已审核通过"，跳过审核直接写入 Word。
2. **审核循环**：子智能体逐行检查 14 项规范（见审核清单），审核→修改→再审核至全部通过。每轮独立启动子智能体。
3. **Word 修改**：优先增量修改（打开→修改→保存）。分节符删除、脚注重新编号等结构性改动允许重建。
4. **完成确认**：任务结束前向用户汇报已保存文档的完整路径与写入结果，并确认是否需要进一步修改。

### MD 稿写作要求

MD 稿结构需一一对应 Word 文档各部分，不得缺项。缺少摘要或关键词时须根据正文补写。

| Word 部分 | MD 对应 | 要求 |
|-----------|---------|------|
| 论文大标题 | `# 标题` | 必填 |
| 作者信息 | 无对应标记 | 有则填，无则跳过 |
| 摘要 | 正文段落（150-300字） | 必填 |
| 关键词 | 3-5个（分号分隔） | 必填 |
| 引言 | 不加标题的2-3段 | 必填 |
| 各章正文 | `## 一、……` / `### （一）……` | 按需 |
| 结语 | 不加标题的收束段落 | 必填 |
| 参考文献 | `## 参考文献` + 条目列表 | 有引用则必填 |
| 脚注 | `## 脚注` + `[^1]: ……` 格式 | 有脚注则必填 |
| 附录 | `## 附录：……` | 按需 |

**脚注与参考文献分离规则：** MD 正文中用 `[^1]`、`[^2]` 标记脚注位置，用 `(作者, 年份 [N])` 标记参考文献引用。脚注内容（URL、代码路径、术语解释等补充说明）统一放在文末 `## 脚注` 区。参考文献（正式引用的论文、著作）统一放在 `## 参考文献` 区。写法示范见下方「三、引用与参考格式」及「脚注与尾注区分」两节。

**字数原则：** 用户有明确字数要求时按要求执行；无要求时以说清楚为准，不硬凑、不灌水。删掉不损失实质内容的句子就删掉。

### Word 写入顺序（不可颠倒）

正文全部段落（含脚注） → 参考文献表（含书签） → 正文中插入尾注交叉引用(REF \r 字段) → 页脚页码 → Fields.Update() → Save()

尾注交叉引用依赖参考文献条目和书签已存在，必须后于参考文献表写入。章节交叉引用同理。

### 子智能体审核清单（14 项）

| 编号 | 检查项 | 依据 |
|------|--------|------|
| A1 | 全文无 `——`（破折号，直接引文内除外） | 标点 |
| A2 | 全文无 `-` 作列表标记 | 标点 |
| A3 | 全文无 `_` 下划线 | 标点 |
| A4 | `：` 仅用于书名和直接引文前 | 标点 |
| A5 | `""` 仅用于直接引文 | 标点 |
| A6 | 无连续标点（如"……。"） | 标点 |
| B1 | 无 AI 套话（不仅…更…、标志着、彰显了、不是…而是…、集大成、奠定了、值得注意的是、毋庸置疑、众所周知、总而言之等） | 语言 |
| B2 | 无空泛赞美（"具有重要的学术价值""值得深入研读""填补了空白"等） | 语言 |
| B3 | 无绝对化词语（最、永远、任何、唯一、彻底、完全、核心） | 语言 |
| B4 | 无空洞修辞（删掉不损失实质的句子）和无意义排比（如"既…也…同时还…"式修辞性对仗） | 语言 |
| B5 | 主语自然交替，不每句同一主语开头 | 语言 |
| B6 | 结尾无自我指涉（"通过以上分析""对本篇报告来说"等） | 语言 |
| C1 | 引文自然嵌入，无单一"某某说"句式 | 引用 |
| C2 | 引用真实准确，无编造 | 引用 |
| D1 | 结构完整（引言→各章→结语→参考文献→脚注±附录） | 结构 |
| D2 | 字数合理（用户有要求时按目标；无要求时说清楚即可，不灌水） | 结构 |

审核报告格式：

```
## 审核报告
### 通过项（X/14）
- A1 ✓ ……
### 需修改项（X/14）
- A4 ✗ 第N段："……" → 改为"……"
### 修改建议汇总
1. ……
```

---

## 一、标点符号规范

### 绝对禁止的符号

- `——`（破折号的一半）— 直接引文内部的除外
- `-`（英文连字符用作列表标记）— 街名等专有名词翻译除外
- `_`（下划线）
- `————`（破折号）— 直接引文内部的除外

### 严格限制的符号

- `：`（冒号）：正文中尽量不用。例如不用"原因在于：社会结构的变化"（改为句号断句）。仅允许在书名中（如《唐人街：共生与同化》）和直接引文前的引导句中使用。
- `""`（引号）：**仅用于直接引用原文**。禁止用于概念术语的强调、比喻性表达的装饰、普通词语的着重。书名用《》，术语不加任何引号。
- `''`（单引号）：同理，仅用于引文内部的次级引述。

### 其他要求

- 中文正文使用全角标点
- 不出现连续标点（如"……。"）
- 分号（；）仅在并列分句间使用，不乱用

---

## 二、语言风格规范

自然流畅的学术散文，有自己的判断和分析，**不是内容摘要**。用自然段落呈现，不列点、不编号（正文中）。

**AI 套话（严格禁止）：** 不仅…更… / 标志着 / 彰显了 / 集大成 / 奠定了 / 不是…而是… / 值得注意的是 / 毋庸置疑 / 众所周知 / 总而言之 / 一言以蔽之 / 精准地概括

**空泛赞美（严格禁止）：** "具有重要的学术价值" / "值得深入研读" / "不可多得的佳作" / "填补了空白"

**绝对化词语（严格禁止）：** 最 / 绝迹 / 永远 / 任何 / 唯一 / 永久 / 彻底 / 完全 / 核心 / 关键 —— 除非有明确学术共识或原文出处支撑

**其他禁止：** 空洞修辞（删掉不损失实质就删）、无意义排比（如"既……也……同时还……"式的修辞性对仗）、结尾自我指涉（"对本篇读书报告来说""通过以上分析可以看出"）、每句同一主语开头

**正向要求：** 每句话提供可验证信息；比喻有节制，传递具体判断而非修辞快感；结尾收束于内容本身自然结束。

---

## 三、引用与参考格式

### MD 层面的脚注与参考文献分离

正文中同时存在两类标注时，按以下规则区分：

- **脚注标记** `[^1]`、`[^2]`：用于 URL、代码路径、术语解释、补充说明等非正式引用内容。脚注正文统一放在文末 `## 脚注` 区，每条以 `[^1]:` 开头。
- **参考文献引用** `(作者, 年份 [N])`：用于正式引用的论文、著作。参考文献条目统一放在 `## 参考文献` 区，使用 `[1]`、`[2]` 编号。

**调研报告中的源码路径脚注：** 介绍系统架构时，每个核心模块的描述段落末尾应添加脚注标注其源码文件的相对路径。格式示例：

```markdown
Manager 是一个基于 GPT-4o 的 CodeAgent，定义在 `run_hist.py` 中[^6]。

Text WebBrowser Agent 负责网络搜索和浏览，实现在 `scripts/text_web_browser.py`[^9]。
```

对应脚注区写法：

```markdown
[^6]: Manager 的定义和配置集中在 `run_hist.py` 中，其 prompt 模板包含调度策略指引和子 Agent 调用规范。

[^9]: Text WebBrowser Agent 的完整实现在 `scripts/text_web_browser.py`，包含 SearchInformationTool、VisitTool 等工具类。
```

**正文中的引用格式示例：**

```markdown
ReAct 范式由 Yao 等人在 2022 年提出（Yao et al., 2023 [2]）。

项目的技术栈以 HuggingFace 开发的 smolagents 框架[^4]为基础。

2025 年，普林斯顿大学与复旦大学的研究团队联合提出了 HistAgent 系统[^1]。
```

对应的参考文献和脚注区：

```markdown
# 参考文献
[1] Qiu J, et al. On path to multimodal historical reasoning... [J]. arXiv, 2025.
[2] Yao S, et al. ReAct: Synergizing reasoning and acting... [C]. ICLR, 2023.

# 脚注
[^1]: 论文全称 On Path to Multimodal Historical Reasoning: HistBench and HistAgent。
[^4]: smolagents 是 HuggingFace 于 2024 年发布的 Python 智能体框架。
```

**参考模板说明：** 本节上方列出的标注语法与分区写法即为完整示范，可按同一规则直接套用。

### 其他引用规则

- 直接引文自然嵌入论述，确认真实准确。不编造、不装饰。
- 参考文献仅列实际引用文献。中文条目用中文标点，外文条目用外文标点。中文作者（拼音序）在前，外文在后。格式符合 GB/T 7714。
- 常用格式：专著`[M]` 期刊`[J]` 学位论文`[D]` 在线资源`[EB/OL]`

---

## 四、报告结构

```
标题
摘要（150-300 字）
关键词（3-5 个，用分号分隔）
引言（不加标题，2-3 段自然引入）

一、……（一级标题）
  （一）……（二级标题，如需要）
    1. ……（三级标题，如需要）
二、……
……

结语（不加标题，自然收束的段落）
参考文献
脚注（如有）
附录（如适用）
```

**MD 文件中各区顺序（不可颠倒）：** 正文 → `## 参考文献` → `## 脚注`。脚注区和参考文献区各自独立，脚注内容不混入参考文献列表。

---

## 脚注与尾注区分

### MD 撰写层面（写作时）

正文中混用两类标记，各司其职：

| 标记类型 | 语法 | 用途 | 汇集位置 |
|----------|------|------|----------|
| 脚注 | `[^1]`、`[^2]` | URL、源码路径、术语解释、API 说明等补充信息 | 文末 `## 脚注` |
| 参考文献引用 | `(作者, 年份 [N])` | 正式引用的论文、著作 | 文末 `## 参考文献` |

**MD 正文示例：**

```markdown
2025 年，研究团队提出了 HistAgent 系统[^1]。Manager 定义在 `run_hist.py` 中[^6]。
ReAct 范式由 Yao 等人提出（Yao et al., 2023 [2]）。

# 参考文献
[1] Qiu J, et al. On path to multimodal historical reasoning... [J]. arXiv, 2025.
[2] Yao S, et al. ReAct... [C]. ICLR, 2023.

# 脚注
[^1]: 论文全称 On Path to Multimodal Historical Reasoning: HistBench and HistAgent。
[^6]: Manager 的定义集中在 `run_hist.py`，prompt 模板包含调度策略指引。
```

**调研报告源码路径规则：** 介绍每个核心模块时，在描述段落末尾以脚注标注其源码文件在项目中的相对路径。格式：`实现在 \`scripts/xxx.py\`[^N]`。脚注正文补全文件功能说明。

### Word 写入层面（写入时）

**规则：** 对论文/著作的正式引用 → 尾注（交叉引用至参考文献表，用 `REF 书签名 \r \h` 显示编号）；URL、代码路径、术语解释等补充说明 → 脚注（页面底部，用 `doc.Footnotes.Add()` 插入）。

**脚注标引位置：** 标在需说明的词语或句末。同一处引用多条文献时用逗号或短横连接（如 `[1,2]` 或 `[1-3]`）。

**尾注实现：** 见 5.8 节代码。正文中插入 `REF bmName \r \h` 字段（`\r` 显示参考文献列表自动编号），参考文献条目用 `ListTemplate` 自动编号并创建书签。

---

## 五、Word 排版规范（通过 OfficeMCP 写入）

以下排版参数为作者在 Windows + Microsoft Word（中文版）环境中实测并固化的推荐值，以中文学术报告版式为基准。不同系统若缺少所列字体，请按 5.4 节末尾的字体说明调整。

---

### 5.1 页面设置

| 参数 | 值 |
|------|-----|
| 纸张 | A4（595.3 × 841.9 pt） |
| 上边距 | 72pt（2.54cm） |
| 下边距 | 72pt（2.54cm） |
| 左边距 | 90pt（3.18cm） |
| 右边距 | 90pt（3.18cm） |
| 页眉 | 左侧顶格两行：第一行日期时间，第二行标题（仿宋小五 9pt + Times New Roman） |
| 页脚 | 居中页码，底端距离 1cm（28.35pt），Times New Roman 9pt |

日期时间格式示例：`2026年8月8日15时30分`

**页眉写入代码（紧接 `doc = word.Documents.Add()` 之后执行）：**

```python
from datetime import datetime

now = datetime.now()
time_str = f"{now.year}年{now.month}月{now.day}日{now.hour}时{now.minute}分"
title_str = "论文主标题——副标题（如有）"

sec = doc.Sections(1)
hdr = sec.Headers(1)  # wdHeaderFooterPrimary

# 第一行：日期时间
hdr.Range.Text = time_str + "\r"
hdr.Range.Paragraphs(1).Alignment = 0   # 左对齐
hdr.Range.Paragraphs(1).Range.Font.Name = "Times New Roman"
hdr.Range.Paragraphs(1).Range.Font.NameFarEast = "仿宋"
hdr.Range.Paragraphs(1).Range.Font.Size = 9

# 第二行：论文标题
hdr.Range.InsertAfter(title_str)
hdr.Range.Paragraphs(2).Alignment = 0
hdr.Range.Paragraphs(2).Range.Font.Name = "Times New Roman"
hdr.Range.Paragraphs(2).Range.Font.NameFarEast = "仿宋"
hdr.Range.Paragraphs(2).Range.Font.Size = 9
```

---

### 5.2 文档段落结构（写入顺序）

```
[论文大标题]    论文主标题
[正文]          （空行）
[单位作者]      （学院，作者姓名，学号）
[正文]          （空行）
[摘要关键词]    摘要：本文……（"摘要："二字加粗，首行缩进两个汉字，单倍行距）
[摘要关键词]    关键词：A；B；C（"关键词："三字加粗，首行缩进两个汉字，单倍行距）
[正文]          （空行）
[一级标题]      一、问题提出
[正文]          （空行）
[正文]          正文段落……
[正文]          ……
[正文]          （空行）
[一级标题]      二、文献综述
[正文]          （空行）
[正文]          （二级标题前必须空一行）
[二级标题]      （一）子标题
[正文]          正文段落……
[三级标题]      1. 子子标题（如需使用）
[正文]          正文段落……
[正文]          （二级标题前必须空一行）
[二级标题]      （二）子标题
……
[正文]          （空行）
[一级标题]      参考文献
[正文]          （空行）
[列表段落]      参考文献条目（顶格，不缩进，自动编号 [1]）
[正文]          （空行）
[一级标题]      附录：原始素材
[正文]          （空行）
[二级标题]      附录A：标题
[三级标题]      A.1 子节
[正文]          附录正文……
```

> 关键规则：
> - 空行用 `[正文]` 样式写入 `""`。
> - 一级标题前是否插入**分节符（下一页）**由用户决定，默认不插入。
> - **一级标题前必须空一行、后也必须空一行**（即 `add_para("", "正文")` → `add_para("一、...", "一级标题")` → `add_para("", "正文")`）。
> - **二级标题前必须空一行、后直接接正文或三级标题，不空行**（即 `add_para("", "正文")` → `add_para("（一）...", "二级标题")` → `add_para("正文...", "正文")`）。
> - **一级标题后紧接二级标题时，中间只空一行**（同时满足前两条规则，不额外堆叠空行）。
> - 摘要与关键词：首行缩进两个汉字（18pt，即 9pt 字号×2），单倍行距，"摘要："和"关键词："需加粗。
> - 参考文献条目：使用 Word 自动编号格式 `[1]`，条目文本中不含手动编号前缀。
> - 无作者信息时跳过"单位作者"行，不添加占位符。

---

### 5.3 样式参数表（含拉丁 + 东亚双字体）

| 样式名 | 拉丁字体 | 东亚字体 | 字号 | 加粗 | 对齐 | 行距规则 | 行距值 | 首行缩进 | 左缩进 | 段前 | 段后 |
|--------|----------|----------|------|------|------|----------|--------|----------|--------|------|------|
| `论文大标题` | 等线 Light | **方正小标宋简体** | 22pt | **是** | 居中 | 固定(5) | 13.8pt | 0 | 0 | 24pt | 4pt |
| `单位作者` | 等线 Light | **楷体** | 14pt | 否 | 居中 | 固定(5) | 13.8pt | 0 | 0 | 8pt | 4pt |
| `摘要关键词` | Times New Roman | **宋体** | 9pt | 否(标签加粗) | 两端(3) | 单倍(0) | — | 18pt | 0 | 0 | 4pt |
| `一级标题` | Times New Roman | **宋体** | 15pt | **是** | **居中(1)** | 多倍(4) | 22pt | 0 | 0 | 0 | 0 |
| `二级标题` | Times New Roman | **宋体** | 16pt | 否 | 左(0) | 多倍(4) | 22pt | 0 | 0 | 0 | 0 |
| `三级标题` | Times New Roman | **宋体** | 12pt | **是** | 左(0) | 多倍(4) | 22pt | 8.75pt | 0 | 0 | 0 |
| `正文` | Times New Roman | **宋体** | 12pt | 否 | 两端(3) | 固定(5) | 13.8pt | 24pt | 0 | 0 | 0 |
| `列表段落` | Times New Roman | **宋体** | 12pt | 否 | 两端(3) | 固定(5) | 13.8pt | 0 | 0 | 0 | 0 |
| `脚注文本` | Times New Roman | **宋体** | 9pt | 否 | 左(0) | 固定(5) | 13.8pt | 0 | 0 | 0 | 0 |

> **行距规则常量**：`SINGLE = 0`（单倍行距），`MULTI = 4`（多倍行距），`FIXED = 5`（固定值）。`FIXED = 5` 不是 3！对齐 `0`=左 `1`=居中 `2`=右 `3`=两端。
>
> **首行缩进**：正文和摘要关键词的缩进值为**两个汉字宽度**（2 × 字号）。正文 12pt → 24pt；摘要关键词 9pt → 18pt。
>
> **摘要关键词加粗**：样式本身不加粗（`bold=False`）。写入段落后，通过 Range 操作将 `摘要：` 或 `关键词：` 前缀部分单独设为加粗（见 5.5 节代码）。
>
> **列表段落**：参考文献条目顶格书写，不缩进、无悬挂缩进。编号使用 Word 自动编号格式 `[1]`（见 5.5 节代码），而非手动输入。
>
> **字体**：方正小标宋简体多数系统未预装，论文大标题东亚字体用"宋体"（如系统有此字体可改回）。等线 Light 英文名为 "DengXian"，楷体/宋体/Times New Roman 均预装。

---

### 5.4 完整 Python 代码模板（样式创建）

```python
import pythoncom
pythoncom.CoInitialize()

word = Officer.Word
word.Visible = True
doc = word.Documents.Add()

# ======================== 页面设置 ========================
page = doc.PageSetup
page.PageWidth = 595.3          # A4
page.PageHeight = 841.9
page.TopMargin = 72             # 2.54cm
page.BottomMargin = 72
page.LeftMargin = 90            # 3.18cm
page.RightMargin = 90

# ======================== 行距 + 对齐常量 ========================
SINGLE = 0         # wdLineSpaceSingle（单倍行距）
MULTI = 4          # wdLineSpaceMultiple（多倍行距）
FIXED = 5          # wdLineSpaceExactly（固定值）—— 不是 3！
LEFT = 0; CENTER = 1; RIGHT = 2; JUSTIFY = 3

# ======================== 样式工具函数 ========================
def get_style(doc, name):
    """按 NameLocal 查找样式；找不到则新建（1=wdStyleTypeParagraph）"""
    for i in range(1, doc.Styles.Count + 1):
        s = doc.Styles(i)
        if s.NameLocal == name:
            return s
    return doc.Styles.Add(name, 1)

def set_style_fmt(s, latin, ea, size, bold, align,
                  line_rule=None, line_value=None,
                  first_indent=0, left_indent=0,
                  space_before=0, space_after=0):
    """
    设置样式的字体和段落格式。
    first_indent 默认 0（不缩进），需缩进的样式显式传入具体值。
    """
    s.Font.Name = latin
    s.Font.NameFarEast = ea
    s.Font.Size = size
    s.Font.Bold = bold
    s.ParagraphFormat.Alignment = align
    if line_rule is not None:
        s.ParagraphFormat.LineSpacingRule = line_rule
        if line_rule != SINGLE and line_value is not None:
            s.ParagraphFormat.LineSpacing = line_value
    s.ParagraphFormat.FirstLineIndent = first_indent
    s.ParagraphFormat.LeftIndent = left_indent
    s.ParagraphFormat.SpaceBefore = space_before
    s.ParagraphFormat.SpaceAfter = space_after

# ======================== 创建全部样式 ========================
# ★ 顺序要求：必须先修改正文样式，再创建其他样式。
# Word 新建样式默认继承正文；若后改正文，其首行缩进会覆盖已设好的子样式。
normal = get_style(doc, "正文")
set_style_fmt(normal, "Times New Roman", "宋体", 12, False, JUSTIFY,
              FIXED, 13.8, first_indent=24)  # 两个汉字宽度 = 2×12pt

set_style_fmt(get_style(doc, "论文大标题"), "DengXian", "宋体", 22, True, CENTER,
              FIXED, 13.8, space_before=24, space_after=4)
set_style_fmt(get_style(doc, "单位作者"), "DengXian", "楷体", 14, False, CENTER,
              FIXED, 13.8, space_before=8, space_after=4)
set_style_fmt(get_style(doc, "摘要关键词"), "Times New Roman", "宋体", 9, False, JUSTIFY,
              SINGLE, first_indent=18, space_after=4)  # 两个汉字宽度 = 2×9pt
set_style_fmt(get_style(doc, "一级标题"), "Times New Roman", "宋体", 15, True, CENTER,
              MULTI, 22)
set_style_fmt(get_style(doc, "二级标题"), "Times New Roman", "宋体", 16, False, LEFT,
              MULTI, 22)
set_style_fmt(get_style(doc, "三级标题"), "Times New Roman", "宋体", 12, True, LEFT,
              MULTI, 22, first_indent=8.75)
set_style_fmt(get_style(doc, "列表段落"), "Times New Roman", "宋体", 12, False, JUSTIFY,
              FIXED, 13.8)
set_style_fmt(get_style(doc, "脚注文本"), "Times New Roman", "宋体", 9, False, LEFT,
              FIXED, 13.8)

# 字体说明：
# - 方正小标宋简体多数系统未预装，此处论文大标题东亚字体用"宋体"，如系统有此字体可改回。
# - 等线 Light 英文名为 "DengXian"，中文 Windows 预装。
# - 楷体/宋体/Times New Roman 均预装。
```

---

### 5.5 写入段落与分节

```python
# ======================== 写入工具函数 ========================
def add_para(text, style_name="正文"):
    """在文档末尾新建一段，应用指定样式。
    文本前插入 \\r 确保独立成段（空字符串也不例外）。
    样式通过对象而非字符串赋值，避免名称查找失败。"""
    style_obj = get_style(doc, style_name)
    rng = doc.Content
    rng.Collapse(0)               # 移到文档末尾
    rng.InsertAfter("\r" + text)  # \\r 确保段落分隔
    doc.Paragraphs(doc.Paragraphs.Count).Style = style_obj

def add_section_break():
    """插入分节符（下一页），使后续内容另起一页。默认不调用，仅用户明确要求时使用。"""
    rng = doc.Content
    rng.Collapse(0)
    rng.InsertBreak(2)            # 2 = wdSectionBreakNextPage

def add_abstract_keywords_para(text):
    """添加摘要或关键词段落，将冒号前的标签部分加粗"""
    add_para(text, "摘要关键词")
    para = doc.Paragraphs(doc.Paragraphs.Count)
    colon_pos = text.find("：")
    if colon_pos >= 0:
        # 将"摘要："或"关键词："部分加粗
        rng = para.Range
        rng.SetRange(rng.Start, rng.Start + colon_pos + 1)
        rng.Font.Bold = True

# ======================== 写入标题区 ========================
add_para("论文主标题", "论文大标题")
add_para("", "正文")
add_para("（学院，作者姓名，学号）", "单位作者")
add_para("", "正文")
add_abstract_keywords_para("摘要：本文……")        # 自动将"摘要："加粗
add_abstract_keywords_para("关键词：A；B；C")      # 自动将"关键词："加粗
add_para("", "正文")

# ======================== 正文第一章 ========================
add_para("", "正文")                                   # 一级标题前空一行
add_para("一、问题提出", "一级标题")
add_para("", "正文")                                   # 一级标题后空一行
add_para("正文第一段内容……", "正文")
add_para("正文第二段内容……", "正文")

# ======================== 正文第二章 ========================
add_para("", "正文")                                   # 一级标题前空一行
add_para("二、文献综述", "一级标题")
add_para("", "正文")                                   # 一级标题后、二级标题前：只空一行（同时满足两规则）
add_para("（一）大语言模型的知识生成与局限", "二级标题")
add_para("正文段落……", "正文")                         # 二级标题后直接正文
add_para("1. 细分要点", "三级标题")
add_para("正文段落……", "正文")
add_para("", "正文")                                   # 二级标题前空一行
add_para("（二）检索增强生成与互联网辅助写作", "二级标题")
add_para("正文段落……", "正文")

# ======================== 结语 ========================
add_para("", "正文")                                   # 结语前空一行
add_para("综上所述，……", "正文")

# ======================== 参考文献 ========================
add_para("", "正文")
add_para("参考文献", "一级标题")
add_para("", "正文")

# ——参考文献自动编号 [1] [2] [3]……——
list_template = doc.ListTemplates.Add(OutlineNumbered=False)
level1 = list_template.ListLevels(1)
level1.NumberFormat = "[%1]"
level1.TrailingCharacter = 0   # wdTrailingNone，编号后不加空格或制表符

def add_reference(text):
    """写入一条参考文献，文本中不含手动编号前缀。编号由 Word 自动生成。"""
    rng = doc.Content; rng.Collapse(0)
    rng.InsertAfter("\r" + text)
    para = doc.Paragraphs(doc.Paragraphs.Count)
    para.Style = get_style(doc, "列表段落")
    para.Range.ListFormat.ApplyListTemplate(list_template, True)

refs = [
    "Vaswani A, Shazeer N, Parmar N, et al. Attention is all you need...",
    "Devlin J, Chang M W, Lee K, et al. BERT: Pre-training of deep bidirectional transformers...",
]
for ref in refs:
    add_reference(ref)

# ======================== 附录 ========================
add_para("附录：原始素材", "一级标题")
add_para("", "正文")
add_para("附录A：核心提示词模板", "二级标题")
add_para("A.1 素材整理阶段", "三级标题")
add_para("以下提示词提取自……", "正文")

# ======================== 保存 ========================
# 全文写入完成后刷新所有字段
doc.Fields.Update()
output_path = r"用户指定的路径.docx"
doc.SaveAs(output_path)
output = f"已保存至：{output_path}"

# ======================== 页脚页码（保存后执行） ========================
# 放在保存之后以避免分节符影响
for s in range(1, doc.Sections.Count + 1):
    sec = doc.Sections(s)
    sec.PageSetup.FooterDistance = 28.35  # 底端距离 1cm
    footer = sec.Footers(1)               # wdHeaderFooterPrimary
    # 清除可能残留的旧页码框架
    for sh in list(footer.Shapes):
        try: sh.Delete()
        except: pass
    footer.Range.Text = ""
    footer.PageNumbers.Add(1, True)        # 1=wdAlignPageNumberCenter, 居中
    # 页码字体
    for f in footer.Range.Fields:
        if f.Type == 33:                   # wdFieldPage
            f.Result.Font.Name = "Times New Roman"
            f.Result.Font.Size = 9         # 小五

doc.Save()  # 再次保存以持久化页码
```

**关键规则：**

| 规则 | 说明 |
|------|------|
| 段落分隔 | `add_para` 在文本前插入 `\r` 确保独立成段 |
| 样式赋值 | 使用样式对象（`get_style`）而非字符串名 |
| 摘要/关键词 | 用 `add_abstract_keywords_para`，自动将 `摘要：` / `关键词：` 加粗 |
| 二级标题前空行 | 必须在二级标题前 `add_para("", "正文")` |
| 参考文献条目 | 用 `列表段落` 样式，顶格、不缩进 |
| 字段刷新 | 全文写入后 `doc.Fields.Update()` |
| 分节符 | 默认不插入，仅用户明确要求时使用 `add_section_break()` |

---

### 5.6 脚注的添加

默认版式使用页面底部脚注，编号由 Word 按页自动重新开始。脚注文本样式为 `脚注文本`（Times New Roman + 宋体，9pt，固定行距 13.8pt）。

```python
# 在正文某处添加脚注
rng = doc.Content
rng.Collapse(0)
rng.InsertAfter("这是需要加注的正文内容。")
ft = doc.Footnotes.Add(rng, "", "脚注补充说明……")
ft.Range.Font.Name = "Times New Roman"
ft.Range.Font.NameFarEast = "宋体"
ft.Range.Font.Size = 9
```

> 脚注内容从第一个脚注开始顺序编号。正文中的引用标注（如 `[1][2]`）是手动写入的标记文字，不是 Word 交叉引用域。

---

### 5.7 交叉引用（章节/图表/附录）

Word COM 的 `InsertCrossReference` 方法在中文版 Word 中不可用（返回"命令失败"）。通过**书签 + Field 字段**实现。

**原理：**

1. 创建书签：`doc.Bookmarks.Add("标签名", heading_range)`
2. 插入 REF 字段（`\h`）：显示被引用标题的文本（相当于"详见X节"）
3. 插入 PAGEREF 字段（`\h`）：显示被引用标题所在的页码（相当于"见第X页"）
4. 全文写入后 `doc.Fields.Update()` 刷新所有字段

**完整示例：**

```python
# === 1. 在构建文档时，为每个需要被引用的标题创建书签 ===
def add_heading_with_bookmark(text, style_name, bookmark_name):
    """插入带书签的标题段落"""
    add_para(text, style_name)
    para = doc.Paragraphs(doc.Paragraphs.Count)
    doc.Bookmarks.Add(bookmark_name, para.Range)

# 写入各章标题时使用
add_heading_with_bookmark("一、问题提出", "一级标题", "sec_problem")
add_heading_with_bookmark("二、文献综述", "一级标题", "sec_review")
add_heading_with_bookmark("三、系统架构", "一级标题", "sec_arch")

# === 2. 在正文中需要引用其他章节时，用 Selection 插入字段 ===
def insert_ref_field(text_before, bookmark_name, text_after=""):
    """在当前位置插入 REF 字段（显示被引用标题的文本）"""
    rng = doc.Content
    rng.Collapse(0)
    rng.InsertAfter(text_before)
    rng.SetRange(rng.End - 1, rng.End - 1)
    rng.Select()
    word.Selection.Collapse(0)
    word.Selection.Fields.Add(word.Selection.Range, -1,
                              f"REF {bookmark_name} \\h", False)
    if text_after:
        word.Selection.TypeText(text_after)

def insert_pageref_field(text_before, bookmark_name, text_after=""):
    """在当前位置插入 PAGEREF 字段（显示被引用标题的页码）"""
    rng = doc.Content
    rng.Collapse(0)
    rng.InsertAfter(text_before)
    rng.SetRange(rng.End - 1, rng.End - 1)
    rng.Select()
    word.Selection.Collapse(0)
    word.Selection.Fields.Add(word.Selection.Range, -1,
                              f"PAGEREF {bookmark_name} \\h", False)
    if text_after:
        word.Selection.TypeText(text_after)

# 使用示例——在正文段落中引用其他章节
add_para("如前文所述，", "正文")
insert_ref_field("本研究在", "sec_review", "一节中已梳理了相关理论基础，")
insert_pageref_field("具体设计详见", "sec_arch", "（第")
word.Selection.TypeText("页）。")

# === 3. 全部写完后刷新字段 ===
doc.Fields.Update()
```

**使用场景总结：**

| 需求 | 方法 | 示例 |
|------|------|------|
| 引用章节名 | `REF \h` 字段 | "详见二、文献综述一节" |
| 引用页码 | `PAGEREF \h` 字段 | "见第3页" |
| 引用图表/表格 | 给图表段落加书签 + `REF \h` | "如表1所示" |
| 引用附录 | 给附录标题加书签 + `REF \h` | "详见附录A" |

**注意事项：**
- 书签名称必须唯一，建议用英文+下划线命名（如 `sec_problem`, `fig_arch`）
- 字段在 `Fields.Update()` 之前显示为空，必须刷新
- `Selection.Fields.Add` 的 `Type=-1` 表示手动字段，`\\h` 表示超链接

---

### 5.8 参考文献引用（交叉引用到参考文献表）

参考文献表用自动编号 `[1]` `[2]` `[3]`……每个条目有一个隐藏书签；正文中的 `[N]` 是 REF 字段（代码 `REF 书签名 \r \h`），`\r` 显示该条目在编号列表中的序号。

引用编号与参考文献表自动同步——增删文献后，正文中的编号自动更新。

**实现代码：**

```python
# ====== 1. 准备：给参考文献列表启用自动编号 ======
# 使用 ListGalleries 创建 [1] [2] [3] 格式的编号
list_template = doc.ListGalleries(2).ListTemplates(1)  # wdNumberGallery
# 修改编号格式为 [1]
for level in list_template.ListLevels:
    level.NumberFormat = "[%1]"
    level.TrailingCharacter = 0  # wdTrailingNone（不额外加符号）

# ====== 2. 写入参考文献表条目 ======
add_section_break()
add_para("参考文献", "一级标题")
add_para("", "正文")

ref_bookmarks = []  # 记录每个参考文献的书签

def add_reference(text):
    """写入一条参考文献，应用自动编号 + 创建隐藏书签"""
    rng = doc.Content
    rng.Collapse(0)
    rng.InsertAfter(text)
    para = doc.Paragraphs(doc.Paragraphs.Count)
    para.Style = "列表段落"
    # 应用自动编号
    para.Range.ListFormat.ApplyListTemplate(list_template, True)
    # 创建隐藏书签（Word 自动隐藏 _Ref 前缀的书签）
    bm_name = f"_Ref_ref{len(ref_bookmarks) + 1}"
    doc.Bookmarks.Add(bm_name, para.Range)
    ref_bookmarks.append((bm_name, text[:50]))

# 写入参考文献
add_reference("Lewis P, et al. Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks [J]. NeurIPS, 2020.")
add_reference("Vaswani A, et al. Attention is all you need [J]. NeurIPS, 2017.")
add_reference("Devlin J, et al. BERT: Pre-training of deep bidirectional transformers [J]. NAACL, 2019.")

# ====== 3. 在正文中插入交叉引用字段 ======
# 例如在正文某处写 "……提出了检索增强生成方法（Lewis et al., 2020 "
# 然后插入 REF 字段显示 [1]

def insert_citation_ref(bookmark_name):
    """插入指向参考文献的 REF \r 字段，显示自动编号如 [1]"""
    rng = doc.Content
    rng.Collapse(0)
    rng.Select()
    word.Selection.Collapse(0)
    word.Selection.Fields.Add(
        word.Selection.Range, -1,
        f"REF {bookmark_name} \\r \\h", False
    )

# 正文中引用第一条参考文献
add_para("本文采用的检索增强生成方法（Lewis et al., 2020 ", "正文")
insert_citation_ref(ref_bookmarks[0][0])  # 插入 [1]
add_para("）通过外部语料检索来增强生成过程。", "正文")
# 注意：上面这行会把 [1] 和后续文本接在一起，实际应在同一段内完成

# ====== 4. 全部写完后刷新字段 ======
doc.Fields.Update()
```

**同一个段落内插入引用的正确写法：**

```python
# 不要在 add_para 中间插字段——add_para 创建新段落
# 应该在一个段落内连续操作 Selection：

rng = doc.Content
rng.Collapse(0)
rng.InsertAfter("本文采用的检索增强生成方法（Lewis et al., 2020 ")
# 插入引用字段
rng.SetRange(rng.End - 1, rng.End - 1)
rng.Select()
word.Selection.Fields.Add(word.Selection.Range, -1,
    f"REF {bm_name} \\r \\h", False)
# 继续在当前段落写后续文字
word.Selection.TypeText("）通过外部语料检索来增强生成过程。")
```

**与 5.7 节（章节交叉引用）的区别：**

| | 章节引用 | 参考文献引用 |
|---|---|---|
| 显示内容 | 标题文本（`\\h`） | 编号 `[N]`（`\\r`） |
| 引用对象 | 章节标题段落 | 参考文献列表条目 |
| 字段代码 | `REF sec_method \\h` | `REF _Ref_ref1 \\r \\h` |
| 编号来源 | 不使用 | 参考文献列表自动编号 |

---

### 5.9 附录的写入

每个附录（A、B、C……）使用 `二级标题` 样式作为标题，子节使用 `三级标题`。附录标题格式为"附录X：标题"。附录正文使用 `正文` 样式。

---

### 5.10 字号与行距速查

| 号数 | 磅数 | 用途 | | 常量 | 值 | 含义 |
|------|------|------|---|------|-----|------|
| 二号 | 22 | 论文大标题 | | SINGLE | 0 | 单倍行距 |
| 小三 | 15 | 一级标题 | | — | 3 | 最小值（不用） |
| 三号 | 16 | 二级标题 | | MULTI | 4 | 多倍行距 |
| 四号 | 14 | 单位作者 | | FIXED | 5 | 固定值 |
| 小四 | 12 | 正文、三级标题、列表段落 | | | | |
| 小五 | 9 | 摘要关键词、脚注文本 | | | | |

---

## 文件 Frontmatter 规范

所有通过本技能生成的 `.md` 文件，开头必须包含标准化的 YAML frontmatter：

```yaml
---
project: 人工智能在历史学研究中的应用    # 所属研究项目（必填）
type: 阶段报告                           # 类型：阶段报告/读书报告/文献综述/评测报告
date: 2026-08-08
tags: [AI, 历史学, RAG, 智能体, 数字人文]
status: V1                               # V1/V2/终稿
---
```

| 字段 | 必填 | 说明 |
|------|------|------|
| `project` | 是 | 研究项目名，决定文件归属和 `{项目名}_概述.md` |
| `type` | 是 | 阶段报告 / 读书报告 / 文献综述 / 评测报告 / 审核报告 |
| `date` | 是 | 生成日期，格式 `YYYY-MM-DD` |
| `tags` | 推荐 | 关键词列表 |
| `status` | 推荐 | V1 / V2 / 终稿 |

---

## 文件命名规范

```
YYYYMMDD_HHMM_类型_标题_V版本.docx
```

示例：`20260808_1530_读书报告_唐人街共生与同化_V1.docx`

**Python 代码：**

```python
from datetime import datetime

now = datetime.now()
filename = f"{now.strftime('%Y%m%d_%H%M')}_{report_type}_{title_text}_V{version}.docx"
output_path = os.path.join(save_dir, filename)
doc.SaveAs(output_path)
```

---

## 交互与进度汇报规则

1. 在 MD 撰写与审核阶段（确认需求→撰写→审核→用户确认），每轮先说明当前所处环节与需要用户确认的事项。
2. 进入 Word 写入阶段后，每个写入步骤完成即同步汇报进度；遇到 COM 调用或样式异常时如实说明，并给出处理建议。
3. 全部流程执行完毕后，汇报最终文档的保存路径与排版处理结果（分节、页码、脚注/尾注），并确认是否还需调整。
