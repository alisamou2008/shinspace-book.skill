# 边界五层 · 从附录变成结构

## 问题起点

书的边界是**解释性**的：作者写清限定条件，交给读者自己判断。
Skill 的边界必须是**可判定**的：交给运行时执行。

写在 `description` 里的一句「不适用于 X」几乎不生效——模型读到了，但不会主动发起否定判断。所以边界不是一段文字，是五道门。

蒸馏是有损压缩，而且损失恰好集中在边界：限定条件、作者自述的失败案例、论证过程（负责给读者校准直觉）被当成噪音削掉，只留下框架与步骤。结果不是「少了点信息」，是**有条件成立的洞见变成了一组看起来无条件的指令**。这五层就是把被削掉的东西按可判定的形式装回去。

## 与上游 B 段的关系

| | 上游 B | 本包 L1–L5 |
|---|---|---|
| 粒度 | 单张能力卡 | 单卡 + 整本书 / 整个判断单元 + 跨卡组合 |
| 位置 | 卡片末尾一小节 | 触发、前置、输出、反例、回写五处 |
| 谁判定 | 读者 / 模型自觉 | 运行时 gate + 机器可读 yaml |
| 是否演化 | 静态 | L5 随误用回写 |
| 产物 | Markdown 文字 | `BOUNDARY.yaml` + 卡片 B 段人读摘要 |

**兼容，不替换。** 上游 B 的四小节这样映射：

| 上游 B 小节 | 映射到 |
|---|---|
| 不要在以下情况使用 | L1（负例信号） |
| 作者在书中警告的失败模式 | L4（硬规则，带原文定位） |
| 作者的盲点 / 时代局限 | L2（assumptions / probe） |
| 容易混淆的邻近方法论 | L1（近邻负例，与上游 A2 的区分合流） |

再补上上游没有的 L3（随结果流动的假设标签）与 L5（回写演化）。

## L1 · 触发层

- **位置**：frontmatter `description`（≤300 字，沿用上游 A2 规范）
- **必须含**：何时用 + 何时不用 + **可识别的负例信号**（用户真会说的话，不是「其他场景」）
- **判定**：负例命中 → 不激活，转向提问 / 拒绝
- **反模式**：「适用于需要决策时」——太宽，必然误激活
- **自检**：能否写出至少 2 条具体的负例措辞？写不出说明边界还没想清楚

## L2 · 前置层

- **位置**：进入 skill 的第一步 `precheck`
- **必需字段**：`assumptions[]`（本方法成立依赖的条件）、`probe`（如何探测输入是否落在域内）、`on_violation`（降级分支）
- **降级三选一**：`ask`（补信息）/ `degrade`（转向提问器或给出范围声明）/ `refuse`（明确不套用）
- **硬约束**：禁止「没探测到就默认适用」。探测不到 → `degrade`，不静默猜值（沿用上游 E 段的输入契约纪律）

## L3 · 输出层

- **位置**：每个结论 / 每轮提问的随行元数据
- **字段**：`depends_on[]`、`unknown[]`、`confidence`（可选）
- **目的**：把边界从「事前约束」变成「随结果流动的元数据」。下游把结果喂给别的 skill 时，假设不会在传递中丢失
- **对提问器**：每一问之后标注「本问依赖用户对 X 的描述」

## L4 · 反例层 ★ 最容易被漏掉

- **位置**：`BOUNDARY.yaml → counter_examples[]` + 卡片 B 段
- **来源**：上游 `counter-example-extractor` 的产出（作者自述失败案例、警告、时代局限）
- **形式**：`if <可识别信号> then do_not_apply + redirect`
- **要求**：至少 1 条，**必须带原文定位**
- **为什么最有效**：具体信号能被模式匹配，抽象的「适用范围」不能。「看到 A 就别套这个框架」比任何边界描述都更容易在运行时被触发
- **禁止**：把「效果可能不佳」这类模糊话写成反例

## L5 · 回写层

- **位置**：`books/<slug>/boundary-misses.md`，每次误用一条
- **字段**：`date / input_signal / wrong_path / what_happened / layer_to_patch / patch_applied`
- **触发**：L1 误激活、L2 漏拦、L4 出现新反例、用户明确指出「这里不该套」
- **规则**：边界不是一次性补全的。每条 miss 必须落到具体某一层，否则只记录不修改
- **这是 Skill 相对书最大的优势**：书的边界是静态的，Skill 的可以随使用演化

## `BOUNDARY.yaml` 字段字典

```yaml
unit_id: U-02
source: "《书名》第 4-5 章"
content_type: judgment
compiled_by: manual_from_template    # 不写 cangjie.py：上游不支持 interrogator

L1_trigger:
  use_when:
    - "用户在权衡 X 且列出了正反理由"
    - "user says: I am torn between ..."
  do_not_use_when:                  # 可识别负例，至少 2 条
    - "只要一个可直接执行的步骤清单"
    - "纯信息查询：XX 是什么"
  near_neighbor_confusions:
    - "与 U-01 的区别：U-01 给核算口径，本单元只提问"

L2_precheck:
  assumptions:
    - "用户能描述自己的具体情境，而非抽象原则"
    - "该情境存在可识别的取舍，而非单一约束下的最优解"
  probe: "输入里是否同时出现两个及以上互相冲突的目标？"
  on_violation: degrade             # ask | degrade | refuse
  degrade_to: "只输出情境澄清问题，不进入 Q 序列"
  never_default_to_applicable: true

L3_output_label:
  depends_on:
    - "用户对自身约束条件的描述"
  unknown:
    - "作者未说明跨领域迁移时的条件"
  attach_to: every_question_and_every_conclusion

L4_counter_examples:
  - signal: "用户已锁定唯一目标，只问怎么最快达成"
    action: do_not_apply
    redirect: "转向答案型执行器"
    source_location: "第 5 章，作者自述的失败案例"
  - signal: "用户要求给出评分或排名"
    action: do_not_apply
    redirect: "复述 N 段禁止输出清单"
    source_location: "结论章，作者对量化的警告"

L5_writeback:
  path: "books/<slug>/boundary-misses.md"
  on: ["L1_false_positive", "L2_miss", "L4_new_signal", "user_override"]
  require_layer_tag: true
```

| 字段组 | 必填 | 校验 |
|---|---|---|
| `L1_trigger.do_not_use_when` | 是 | 至少 2 条具体措辞 |
| `L2_precheck.on_violation` | 是 | 三选一，且不为 `continue` |
| `L3_output_label.depends_on` | 是 | 至少 1 条；写不出填 `unknown` |
| `L4_counter_examples` | 是 | 至少 1 条，带 `source_location` |
| `L5_writeback.path` | 是 | 指向可写文件，不在上游生成目录内 |

**存放位置**：`books/<slug>/BOUNDARY.yaml`，**不要塞进上游 `verified.yaml`**。上游 Capability Bundle 有 `capability-bundle.schema.json` 校验，未知字段会导致编译失败。独立文件是本包刻意的设计。

## 校验清单

1. 五层是否都出现且非空？写不出的位置标 `unknown`，不用模糊话术填满
2. L1 的负例是「用户真会说的话」还是「其他场景」？
3. L2 有降级分支吗？有没有「探测不到就默认适用」的漏洞？
4. 每个输出都带 `depends_on` 了吗？
5. L4 的反例带原文定位吗？能模式匹配吗？
6. L5 的回写路径可写吗？是否落在上游只读生成目录之外？
7. 提问器的 N 段是否已压进 L1 的负例？
8. `compiled_by` 是否如实标注 `manual_from_template`？

## 一个反向张力

边界写太细，Skill 会退化成一份「什么都不敢用」的说明书。所以：

- L2 的 `assumptions` 只写**真正会让结论翻转**的条件，不写所有前提
- L4 只收作者明确警告过的失败模式，不把每种不确定性都做成硬规则
- 无法事前枚举的边界，交给 L5 回写，不硬凑

边界分层 + 可演化，比一次性补全更现实。
