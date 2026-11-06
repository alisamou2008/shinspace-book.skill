# 集成挂载点 · 不改上游源码

## 原则

上游 `SKILL.md` 质量红线第 7 条：「生成目录只读，检测到本地手改时不得静默覆盖」。而且本地改动会在下次升级时被覆盖。

所以本包**不写上游任何文件**。下面的挂载点是「若你决定把判断力路径合并进自己的流水线，该改哪里、贴什么」。每一步都需要人工确认后自行合并，并留一条审计记录。

**不合并也能用**：先用本包做阶段 -1 判定 → `answer` 交回上游 → `judgment` 手工走 J0–J6。合并只是省掉来回切换的成本。

## 挂载点总表

| # | 上游位置 | 动作 | 必要性 |
|---|---|---|---|
| M0 | 「输入要求」之前 | 插入阶段 -1 内容类型门 | 必需 |
| M1 | 阶段 1 · 5 个 extractor | `counter-example-extractor` 产出额外映射进 L4 | 必需 |
| M2 | 阶段 1.5 · V2 可执行性 | 判断力型改写为「可提问性」 | **关键** |
| M3 | 阶段 1.6 · 晋级门 | 提问器不参与 `promoted` 评审 | 建议 |
| M4 | 阶段 2 · RIA++ 六段 | 切换为 RIA-QN+ 七段 | 必需 |
| M5 | 阶段 3 · Zettelkasten | `also_read` 之外补 `depends_on` 链接 | 建议 |
| M6 | 阶段 4 · 压力测试 | 三类 bait + cross-path confusion | 必需 |
| M7 | 阶段 5 · 编译交付 | 手工编译，不调 `cangjie.py compile` | 现实约束 |

---

## M0 · 阶段 -1 内容类型门

**插在 `## 输入要求` 之前**：

```markdown
## 阶段 -1 — 内容类型判定（shinspace-book-judgment 补充）

在确认使用目的之前，先按 `docs/01-content-type-gate.md` 的 6 个可观测信号判定内容类型，
产出 books/<slug>/CONTENT_TYPE.md，逐条留痕。

- answer → 继续本 SKILL.md 原七阶段，另补 BOUNDARY.yaml 的 L1/L4/L5
- judgment → 转 shinspace-book-judgment 的 J0–J6，不在本流水线内继续
- mixed → 按判断单元切分，分别走两条路径
- undetermined → 按 answer 走，标注「内容类型未判定」

内容类型决定路径，使用目的只决定包装。没有原文文本时不判定，先要文本。
```

同时在上游「输入要求」的 4 项后追加第 5、6 项（内容类型判定、边界声明），见补充包 `SKILL.md` 的输入要求节。

## M1 · 反例提取器接 L4

**在阶段 1 的 extractor 表格下追加一行说明**：

```markdown
`counter-example-extractor` 的产出除写入 candidates/counter-example.md 外，
还须逐条映射为 BOUNDARY.yaml 的 L4_counter_examples[]，
每条带 source_location（章节/页码/时间戳）与可识别 signal。
至少 1 条；无法映射时标 unknown，不编造反例。
```

## M2 · V2 改写（最容易被漏掉）

上游阶段 1.5 的 V2 是「用合法的新输入能完成任务并核验结果吗」。用它筛判断力型内容，**几乎全部会被判不通过**——因为提问器的产出本来就不是可核验的任务结果，而是用户自己说出的假设。

**在阶段 1.5 的 V2 下追加**：

```markdown
**判断力路径的 V2 变体（verify_variant: askability）**：
当 CONTENT_TYPE.md 判定为 judgment 时，V2 改写为「可提问性」：
  a) 依据原文能否构造 3–5 个具体、不暗示答案的问题？
  b) 每个问题能否指名它要暴露的假设？
  c) 用户回答后，是否存在可判断的完成标准？
三项全过才算 V2 通过。V1（来源充分性）与 V3（任务增益）不变。
判定理由单独记录在 verified.md，不与 answer 型混记。
```

`config.json → upstream.verify_variant: askability` 记录这一开关。

## M3 · 晋级门

提问器不适合 `promoted` 独立 Skill。上游晋级门判据里的「独立运行 / 独立复用」对提问器会给出误导性结果：它的运行依赖用户参与，不是自足的。

**建议**：判断力单元统一走 `router` 去向，保留为来源路由入口内的能力卡；若要独立安装，按 `templates/INTERROGATOR_SKILL.md.template` 手工产出，并在 frontmatter 标 `compiled_by: manual_from_template`。

## M4 · 能力卡切段

**在阶段 2 的六段定义下追加**：

```markdown
**判断力路径**：R / I / A1 / A2 沿用；E 替换为 Q（提问序列），新增 N（禁止输出）；
B 由四小节附录升级为 L1–L5 结构并以 BOUNDARY.yaml 为机器判定依据。
完整定义见 docs/02-interrogator-card.md。
卡片正文仍写入 .cangjie/capabilities/cards/，但 BOUNDARY.yaml 独立放 books/<slug>/。
```

**重要**：不要把 L1–L5 字段塞进 `verified.yaml`。上游 `schemas/capability-bundle.schema.json` 会校验未知字段，编译会失败。这是本包把边界独立成文件的直接原因。

## M5 · 链接层

在阶段 3 的 `also_read` 之外，为判断力单元补 `depends_on`（来自 L3），让跨书组合时能看出「这个结论依赖了哪一本书的哪个假设」。这是链接关系，不是 Bundle 字段，记在 `BOUNDARY.yaml` 与 `INDEX` 说明里。

## M6 · 压力测试加 bait

上游阶段 4 已有 bait questions 与 cross-skill confusion tests。判断力路径追加三类：

```json
{
  "bait_types": [
    "induce_direct_answer",
    "cross_domain_hard_apply",
    "author_conclusion_as_universal_rule"
  ],
  "cross_path_confusion": [
    "answer_type_book_routed_to_interrogator",
    "judgment_type_book_routed_to_executor"
  ],
  "pass_criterion": "no_conclusion_before_user_reply AND boundary_signal_honored"
}
```

完整模板见 `templates/test-prompts.json.template`。**通过标准不是「会调用」，是「被诱导时仍不给答案」以及「负例信号命中时确实不激活」**——与上游阶段 4「不能用会调用替代做得对」一致。

## M7 · 编译交付

`scripts/cangjie.py compile` 目前只实现 `single` / `pack`（`config.json → upstream.output_modes_supported`）。**没有 `--output interrogator`**。

所以：

1. 按 `templates/INTERROGATOR_SKILL.md.template` 手工产出提问器 Skill
2. 产出 `DIGEST-JUDGMENT.md`（面向读者的「这本书训练了什么判断力」，不是精华摘录）
3. frontmatter / `BOUNDARY.yaml` 里如实写 `compiled_by: manual_from_template`
4. **不得**在产物里写「由 cangjie.py 编译」，也不得伪造 `runs/<run-id>/` 决策报告

若日后上游支持 interrogator，把 `config.json → upstream.compiler_interrogator_support` 改为 `true`，并同步更新 `docs/` 与模板。

## 不建议做的事

- 直接改 `upstream/cangjie-core/SKILL.md`：违反只读红线，且升级必被覆盖
- 给 `cangjie.py` 加 `--output interrogator`：等于维护一个 fork，本包不打算承担
- 用原始 V2 筛提问器：会把判断力内容全灭
- 把 `BOUNDARY.yaml` 塞进 `verified.yaml`：schema 校验失败
- 把 `boundary-misses.md` 放进上游生成目录：会被「检测本地手改」逻辑拦下
- 在没有原文的情况下先跑判定门补内容类型：不凭记忆拆书，也不凭记忆判类型

## 合并审计模板

每次合并挂载点，记一条：

```markdown
| 日期 | 挂载点 | 合并到哪个文件 | 片段版本 | 操作人 | 回滚方式 |
|---|---|---|---|---|---|
| | M2 | | v0.1.0 | | git revert / 从 upstream 镜像重拷 |
```
