# shinspace-book.skill

> 把一本书蒸馏成 Skill 之前，先判断它是在给答案，还是在给判断力。

这是 [kangarooking/cangjie-skill](https://github.com/kangarooking/cangjie-skill) v2.5.0 的**判断力路径补充包**，附带一份裁剪过的上游核心镜像，便于一个仓库 clone 下来就能完整跑通。作者：SHINSPACE（Leo）。

## 为什么要做这个

上游 cangjie-skill 是一套很成熟的元技能：RIA-TV++ 七阶段、5 个并行提取器、三重验证、晋级门、Capability Bundle、确定性编译。它甚至已经有 `B = Boundary` 段。

但它的默认假设是：**书里的方法论都应该被编译成可执行的能力**。于是有一类书会被压坏——那些价值不在结论、而在论证过程让你亲自体验一次认知偏差的书。把这类书做成 checklist，等于把「有条件成立的洞见」变成「看起来无条件的指令」：蒸馏只留下框架与步骤，限定条件、反例和论证过程被当成噪音削掉了，而损失恰好集中在边界上。

同时，B 段是**每张能力卡的一段附录**，写的是「这个能力何时别用」。它解决不了两个更上层的问题：

- 这本书 / 这个判断单元的适用域在哪里？
- 写在 description 里的「不适用于 X」，运行时由谁判定？

这个补充包就是把这两个问题做成结构。

## 两条路径

| | 答案型 answer | 判断力型 judgment |
|---|---|---|
| 产物 | 执行器 Skill（RIA++ 六段，E = 1-2-3 步骤） | 提问器 Skill（RIA-QN+ 七段，Q = 提问序列，N = 禁止输出） |
| 成功标准 | 换个人照着做，得到同样结果 | 用户自己说出「我这个判断依赖了什么假设」 |
| 边界形态 | 能力卡 B 段 + L1/L4/L5 | 完整 L1–L5 五层 + `BOUNDARY.yaml` |

**内容类型判定门**（`docs/01-content-type-gate.md`）用 6 个可观测信号判定，≥4 项落在判断力侧，或两个硬信号同时命中，才走判断力路径。混合型按判断单元切，不按整书一刀切。

**边界五层**（`docs/03-boundary-five-layers.md`）：

| 层 | 位置 | 作用 |
|---|---|---|
| L1 | 触发层 | description 写清「何时不用」+ 可识别的负例信号 |
| L2 | 前置层 | 探测输入是否落在假设域内，不满足走降级分支 |
| L3 | 输出层 | 结论 / 每轮提问自带 `depends_on` 假设标签 |
| L4 | 反例层 | 作者自述的失败案例写成 `if see X → do not apply` 硬规则 |
| L5 | 回写层 | 每次误用记一条，边界随使用演化 |

L4 是最容易被漏掉、却最有效的一层：作者的失败案例是全书里唯一天然可模式匹配的边界信号。

## 目录

```
shinspace-book.skill/
├── SKILL.md                  ← 补充包本体，可直接作为技能安装
├── config.json               ← 判定阈值、五层开关、上游能力开关
├── requirements.txt          ← 依赖说明（本包零第三方依赖）
├── README.md / UPSTREAM.md / LICENSE
├── docs/
│   ├── 01-content-type-gate.md      阶段 -1 判定门与 6 个信号
│   ├── 02-interrogator-card.md      RIA-QN+ 提问器能力卡七段
│   ├── 03-boundary-five-layers.md   L1-L5 规范 + 字段字典
│   ├── 04-integration.md            挂到上游七阶段的挂载点（不改源码）
│   └── 05-install-audit.md          安装记录与安全审计
├── templates/
│   ├── INTERROGATOR_SKILL.md.template
│   ├── BOUNDARY.yaml.template
│   └── test-prompts.json.template
├── examples/                 结构演示，明确不是真实蒸馏结果
└── upstream/cangjie-core/    上游 v2.5.0 裁剪镜像（保留原 LICENSE）
```

## 安装

### 1. 上游 cangjie-skill

`upstream/cangjie-core/` 已经是裁剪好的可运行核心（88 个文件 / 704KB，去掉了图片、网站、benchmark、示例书和联网维护脚本）。复制到技能目录即可：

```powershell
Copy-Item -Path .\upstream\cangjie-core `
          -Destination "$env:USERPROFILE\.workbuddy\skills\cangjie-skill" -Recurse
```

脚本自检（**必须用 WorkBuddy 自带 Python**，系统 `python` 可能没装 PyYAML）：

```powershell
& "$env:USERPROFILE\.workbuddy\binaries\python\envs\default\Scripts\python.exe" `
  "$env:USERPROFILE\.workbuddy\skills\cangjie-skill\scripts\cangjie.py" doctor
```

### 2. 判断力补充包

把仓库根目录（不含 `upstream/`）复制到 `.workbuddy/skills/shinspace-book-judgment/`。它本身就是一个 `SKILL.md` 在根的 prompt-driven 技能。

## 一次典型使用

1. 说「帮我拆《XX》」→ 补充包先做阶段 -1，产出 `CONTENT_TYPE.md`，给出 6 个信号的逐条判定。
2. 判定 `answer` → 交回 cangjie-skill 原流程，额外补 `BOUNDARY.yaml` 的 L1/L4/L5。
3. 判定 `judgment` → 走 J0–J6，产出提问器 Skill：不列 1-2-3 步骤，列 3–5 个带「不代答」约束的问题，以及 N 段禁止输出清单。
4. 压力测试用三类 bait 验证它会不会滑回执行器：诱导直接给答案、跨域硬套、把作者结论当通用规则。

## 现实约束

- 上游 `cangjie.py` 只支持 `single` / `pack`，**不支持 interrogator 输出**。提问器产物按 `templates/` 手工编译，本包不伪造编译结果。
- 上游要求生成目录只读，所以本包不改上游任何文件；`docs/04-integration.md` 给出的是挂载点说明与可复制 patch，需用户确认后自行合并。
- 没有原文文本就不拆书，沿用上游红线。`examples/` 里的东西全部标注为结构演示。

## 许可

补充包：MIT，Copyright (c) 2026 SHINSPACE。
`upstream/cangjie-core/`：MIT，Copyright (c) kangarooking，原 LICENSE 原样保留。详见 `UPSTREAM.md`。
