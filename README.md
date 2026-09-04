# claude-skill-academic-report-writer

一个面向 Claude Code / Claude Agent 的 **Skill（技能）**：把研究笔记、阶段性发现、读书心得整理成格式规整、符合中文学术写作规范的 **Word（.docx）学术报告**。

执行严格的**中文学术写作规范**：标点符号使用、语言风格（去 AI 套话 / 空泛赞美 / 绝对化措辞）、引用格式、多轮子智能体审核、脚注尾注体系，以及通过 OfficeMCP 排版写入 Word 的完整样式模板。

---

> ### ⚠️ 全量使用需要配套 OfficeMCP（重要）
>
> 本技能分成两个阶段，依赖不同：
>
> | 阶段 | 产出 | 是否需要 OfficeMCP |
> |------|------|--------------------|
> | ① 需求确认 + Markdown 稿撰写 + 子智能体审核 | `.md` 草稿（含审核报告） | ❌ 不需要，纯文本处理 |
> | ② 排版写入 Word 生成最终报告 | `.docx` | ✅ **必须** |
>
> **阶段 ② 依赖以下运行环境：**
> 1. **OfficeMCP Server**：微软 Office 的 MCP 服务器，向模型暴露控制 Word 的工具（`Officer.Word` / `RunPython` / COM 对象操作等）。请把它注册进你的 MCP 配置。
> 2. **Microsoft Word（Windows 版）**：SKILL.md 第五节的所有写入代码基于 **Word COM / pythoncom** 编写，目前仅在 **Windows + 中文版 Microsoft Word** 下验证。
> 3. **字体**：默认样式使用宋体 / 楷体 / 等线(DengXian) / Times New Roman（中文 Windows 预装）；`方正小标宋简体` 多数系统未预装，Skill 会自动回退到宋体，或按 5.4 节说明自行安装调整。
>
> 没有上述环境时，本技能仍能完成 **Markdown 稿的撰写与 14 项规范审核**，只是无法产出最终 `.docx`。

---

## 功能特性

- **强制中文学术写作规范**：全角标点、禁止破折号/列表连字符/下划线、限制冒号与引号的使用场景。
- **语言风格把关**：自动剔除 AI 套话（"不仅…更…"、"标志着"、"众所周知"等）、空泛赞美、绝对化词语、无意义排比与结尾自我指涉。
- **脚注与参考文献分离**：Markdown 层 `[^N]` 脚注与 `(作者, 年份 [N])` 文献引用各归其位；Word 层分别落地为「页面底部脚注」与「尾注交叉引用参考文献表」。
- **多轮子智能体审核**：14 项检查（标点 A1–A6 / 语言 B1–B6 / 引用 C1–C2 / 结构 D1–D2），审核→修改→再审，直到 14/14 通过。
- **Word 排版模板**：A4 页面、页眉日期+标题、居中页码、九种中文命名样式（论文大标题 / 单位作者 / 摘要关键词 / 一级二级三级标题 / 正文 / 列表段落 / 脚注文本），含可直接复用的 Python(COM) 代码。
- **交叉引用**：章节 / 图表 / 附录通过书签 + `REF / PAGEREF` 字段实现，参考文献编号自动同步。
- **文件规范**：标准 YAML frontmatter、统一命名规则（`YYYYMMDD_HHMM_类型_标题_V版本.docx`）。

## 工作流程

```
确认需求 → 撰写/接收 Markdown → 子智能体审核（多轮至 14/14 通过）→ 用户确认 → OfficeMCP 写入 Word → 完成确认
```

## 安装

Skill 是一个含 `SKILL.md` 的目录，将其放入技能目录即可被 Claude Code / Claude Agent 识别：

```bash
# Windows（个人级全局安装）
xcopy /E /I claude-skill-academic-report-writer %USERPROFILE%\.claude\skills\academic-report-writer

# macOS / Linux
cp -r claude-skill-academic-report-writer ~/.claude/skills/
```

也可以在项目内 `.claude/skills/` 下只对当前项目生效。重启会话后，技能由 `SKILL.md` frontmatter 中的 `name`（`academic-report-writer`）与 `description` 触发。

> 目录名可随用途调整，但请保持 frontmatter 的 `name` 唯一。

## 使用

向模型表达整理 / 导出意图即可触发，例如：

> "把这篇研究笔记整理成阶段报告并导出为 Word"

常用触发词：`写报告` / `生成报告` / `阶段性报告` / `读书报告` / `研究笔记整理成文` / `导出为Word`。

建议先提供以下信息以获得更贴合的输出：报告类型、主题、素材、字数要求、是否需要参考文献/附录、项目名、作者信息、保存路径。

## 目录结构

```
claude-skill-academic-report-writer/
├── SKILL.md     # 技能定义：规范 + 工作流 + Word(COM) 排版代码模板
├── README.md    # 本说明
└── LICENSE      # GPL-3.0
```

## 兼容性说明

- 排版参数（样式字号 / 行距 / 页边距等）面向 **中文学术报告版式** 设计，来自作者在 Windows + Microsoft Word（中文版）环境下的写作实践。不同系统 / 语言版本可能略有差异，建议先在 5.4 节确认字体可用性。
- Word 写入代码仅在 Windows + Word COM 环境验证；macOS / 网页版 Word 未支持。
- 引用格式遵循 GB/T 7714；`[M] [J] [D] [EB/OL]` 等文献类型标识符见 SKILL.md「引用与参考格式」。

## License

本项目以 **GNU General Public License v3.0 (GPL-3.0)** 开源发布。完整许可文本见 [LICENSE](LICENSE)。

使用、修改或分发时请遵守 GPL-3.0 的相应义务（保留版权声明、以相同许可证分发衍生作品、提供源码等）。

## 参考

- [Claude Skills（官方规范）](https://docs.anthropic.com/en/docs/claude-code/skills) — 技能的 `SKILL.md` 格式与安装约定
- [OfficeMCP（Office MCP Server）](https://github.com/officemcp/officemcp) — 通过 Windows COM 接口控制 Word/Excel/PowerPoint 等的 MCP 服务器（PyPI 包名 `officemcp`），本技能 Word 写入阶段的依赖
