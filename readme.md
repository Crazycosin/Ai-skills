# Ai-skills

一组给 AI 助手加载的 Skill 规则包，覆盖 **代码问答表达、源码精读、概念讲解、C++ 规范、问题复盘、基本面研究** 六类任务。每个 skill 是一份带 `name` / `description` frontmatter 的规则文件（`SKILL.md`），规定何时触发、如何组织回答、最终交付什么；绝大多数无需用户点名，按意图自动生效。

贯穿全部 skill 的共同约束：**不猜测、缺信息先问到可执行粒度、能用代码/图表就别堆文字、中文说人话、实测与推测分开标注、只给当前正确结论而不写修正流水账。**

## 一览

| 目录 | Skill | 定位 |
| --- | --- | --- |
| `code-answer-style-v1` | code-answer-style | 代码类回答的表达规则 |
| `code-reader-v1` | code-reader | 精读一份具体源码 |
| `concept-explainer-v1` | concept-explainer | 讲清一个概念/原理 |
| `Google cpp code style-v1` | google-cpp-code-style | C++ 写改审规范 |
| `Issue postmortem-v1` | issue-postmortem | 排查后的复盘文档 |
| `inverse-fundamental-tracker-v2` | inverse-fundamental-tracker | 股票逆向基本面跟踪 |

## Skill 介绍

### 1. [code-answer-style](./code-answer-style-v1/skills/code-answer-style/SKILL.md) · 代码问答表达规则

涉及代码、配置、命令行、构建脚本、调试步骤的回答都受它约束。五条核心：能用代码贴就不用文字转述；三步以上流程用 mermaid 或表格；禁止「之前建议 A 不对，改成 B」的自我修正叙事，只留最终正确版本；不用翻译腔，按中文开发者的日常说法写；关键信息缺失先问，问到「跑哪条命令、看哪个文件」的粒度，未验证推测须单独标注。文末附回复前自查清单。

### 2. [code-reader](./code-reader-v1/skills/code-reader/SKILL.md) · 源码精读

面向「吃透一份具体代码」，输入支持 GitHub 链接、文件或片段。目标是讲清 WHY 而非复述 WHAT，标准是可脱离原码讲思路、能实现、能迁移、能教会别人；没亲眼读到的代码不许写进分析。分 Quick / Standard / Deep 三档，Deep 产出 md 文件；固定章节含项目地图、动机三问、概念网络、算法退化场景、设计模式、关键片段六节解析、测试反推与四能自测。风格继承 code-answer-style。

### 3. [concept-explainer](./concept-explainer-v1/skills/concept-explainer/SKILL.md) · 概念讲解

用户问「X 是什么、什么原理、有何区别」时触发，把用户视为零基础。先检查提问本身有无概念混淆、术语误用或前提不成立，有问题先纠正再作答；随后严格按「通俗轮廓（生活化类比）→ 明确过渡 → 专业完善（补齐细节与边界）」两步走。每次必须配一张思维导图、层次图或类比关系图，结尾给 1–2 道需推理一步的自测小题。禁止堆术语、翻译腔与硬翻句式；不确定的事实明说或查证，绝不编造。

### 4. [google-cpp-code-style](./Google%20cpp%20code%20style-v1/skills/google-cpp-code-style/SKILL.md) · Google C++ 风格规范

写、改、审 C++ 代码时自动生效，依据 Google C++ Style Guide 中文版（目标 C++20）。涵盖命名、头文件 guard 与 include 分组、静态变量生命周期、类设计（`explicit` / `override` / 拷贝移动、参数次序）、特性红绿灯、格式与注释。改他人文件时原有风格优先；冲突按「项目约定 > 官方规范 > 团队改编版」裁决。附自查清单及 clang-format / clang-tidy 落地建议。

### 5. [issue-postmortem](./Issue%20postmortem-v1/skills/issue-postmortem/SKILL.md) · 排查沉淀文档

问题**解决之后**把过程写成结论先行、证据可复现的复盘；代码 bug 与环境、驱动、构建、跨团队协作问题均适用，排查中不触发。固定七节：现象（报错原文＋伴生现象＋复现/不复现条件）、排查思路（分层图）、逐层过程与证据（实验/结果/结论三列）、根因→表象因果链、各层状态表、结论与行动、方法论。硬性要求：无结果的实验也要写（排除项）、推测标「待验证」、每行状态须有验证手段、输出独立 md 文件。

### 6. [inverse-fundamental-tracker](./inverse-fundamental-tracker-v2/inverse-fundamental-tracker_v2.3_SKILL.md) · 逆向基本面跟踪框架 v2.3

面向 A 股及上市公司的持续跟踪，回答前景、财报复盘、持仓、止损止盈、下一验证节点等。原则是「先找能证伪逻辑的证据，再看上涨空间」：证据按公司原始披露 > 行业高频数据 > 海外同行 > 反方证据 > 券商研报（降权）分级。流程含行业认可度独立判断、单季环比拆解、吨利润/价差趋势、FCF 与 Capex、价格与基本面分离、估值路径、熊牛情景、红黄绿灯与评分。新增定价归因、题材穿透与组合仓位风控。

## 怎么选

| 你在做什么 | 用哪个 |
| --- | --- |
| 报错排查、改配置、讨论代码方案 | code-answer-style |
| 拿着一份具体代码想彻底搞懂 | code-reader |
| 问一个术语、原理、区别（无具体代码） | concept-explainer |
| 写 / 改 / review C++ | google-cpp-code-style（叠加 code-answer-style） |
| 坑已填完，要沉淀或发给上游/供应商 | issue-postmortem |
| 个股基本面、财报、持仓与仓位 | inverse-fundamental-tracker |

分工要点：`code-reader` 继承 `code-answer-style` 的表达规则，但 WHY 部分必须展开；`concept-explainer` 处理无具体代码的概念问题；`issue-postmortem` 只在排查收尾使用。风格类与任务类 skill 可以同时生效。

## 目录结构

```text
Ai-skills/
├── readme.md
├── LICENSE
├── code-answer-style-v1/skills/code-answer-style/SKILL.md
├── code-reader-v1/skills/code-reader/SKILL.md
├── concept-explainer-v1/skills/concept-explainer/SKILL.md
├── Google cpp code style-v1/skills/google-cpp-code-style/SKILL.md
├── Issue postmortem-v1/skills/issue-postmortem/SKILL.md
└── inverse-fundamental-tracker-v2/inverse-fundamental-tracker_v2.3_SKILL.md
```

## 使用说明

- 每个目录是一个独立 skill 包，`SKILL.md` 即入口；将它放入所用 Agent 工具的 skills 目录即可加载，frontmatter 中的 `description` 定义了自动触发条件。
- `inverse-fundamental-tracker-v2` 目前是平铺的单文件（v2.3），没有 `skills/<name>/` 包裹层；若宿主要求标准结构，把该文件复制/重命名为 `SKILL.md` 并放进对应 skill 目录即可。
- `google-cpp-code-style` 主文件索引了 `references/` 细则与 `assets/` 配置，但本仓库当前只收录了 `SKILL.md` 主文件，需要完整细则时按文末索引补齐。
- 所有内容均为方法论与表达规则，不含行情、代码或日志数据；金融类输出不构成投资建议。

## 许可

MIT，详见 [LICENSE](./LICENSE)。
