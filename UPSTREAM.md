# 上游来源、裁剪与安全审计

## 1. 来源

| 项 | 值 |
|---|---|
| 仓库 | `kangarooking/cangjie-skill` |
| 版本 | v2.5.0（`SKILL.md` frontmatter `metadata.cangjie.version`） |
| 主分支 | `main` |
| 快照方式 | `git clone --depth 1`，2026-11-06 |
| 许可证 | MIT（`upstream/cangjie-core/LICENSE` 原样保留） |
| 方法 | RIA-TV++ / Capability Bundle (ADR-002) |

镜像目录：`upstream/cangjie-core/`，88 个文件，约 704KB。

## 2. 保留了什么

`SKILL.md`、`methodology/`、`extractors/`、`templates/`、`scripts/`、`schemas/`、`registry/`、`docs/`，以及根目录的 `LICENSE`、`README.md`、`README.zh-CN.md`、`README.ja.md`、`CHANGELOG.md`、`CONTRIBUTING.md`、`.gitignore`。

这些是运行七阶段流水线与 `cangjie.py compile` 的完整最小集合。

## 3. 裁掉了什么、为什么

| 路径 | 处理 | 理由 |
|---|---|---|
| `.git/`, `.github/` | 剔除 | 版本历史与 CI，与技能运行无关 |
| `assets/` (约 4.7MB) | 剔除 | 二维码、宣传图、字体 base64，占全仓一半以上体积 |
| `website/`, `benchmarks/`, `dist/`, `books/`, `tests/` | 剔除 | 展示站、评测数据、发布包、示例产物、测试用例 |
| `scripts/generate_star_history.py` | **主动剔除** | 唯一联网且读取 `GITHUB_TOKEN` / `GH_TOKEN` 的脚本；用途是生成 star-history 宣传图，与蒸馏流水线无关。剔除后技能包不含任何出站网络调用 |

裁剪只做删除，不改动任何保留文件的内容。

## 4. 安全审计（安装前执行）

扫描范围：全仓 `*.py` / `*.js` / `*.sh` / `*.yml` / `*.yaml`，重点 `scripts/`（21 个 Python 文件）。

| 检查项 | 结果 |
|---|---|
| `eval()` / `exec()` 执行任意字符串 | 未发现。`cangjie.py:50` 的 `__import__(mod)` 仅用于 `doctor` 探测 `yaml` / `tiktoken` / `jsonschema` 是否可导入 |
| `subprocess` | 仅以 `sys.executable` 调用**同目录自带**脚本（`validate_skill_pack.py`、`count_tokens.py`、`run_trigger_evals.py` 等），无外部命令、无 shell=True |
| `shutil.rmtree` | 出现在 `apply_skill_patch.py`、`cangjie.py`、`cangjie_common.py`，作用域均为用户显式传入的 `target` / `staging` / `snapshot` / `backup`；补丁流程先建快照，校验失败自动回滚。无 `~`、`/`、家目录等宽路径 |
| 网络请求 | 仅 `generate_star_history.py`（已剔除）；其余脚本无 `urllib` / `requests` / `socket` |
| 凭据读取 | 仅 `generate_star_history.py` 读 `GITHUB_TOKEN` / `GH_TOKEN`（已剔除）；无 `keyring`、无配置文件凭据扫描 |
| `curl \| sh` / `wget \| sh` / `child_process` / `process.env` | 未发现 |
| 供应链 | 无 `requirements.txt` 自动安装、无 `postinstall`、无远程模板拉取 |

**结论：可安装。** 唯一的网络/凭据面已剔除；剩余脚本都是本地文本处理与自检。

补充包自身（根目录）**不含任何可执行代码**，只有 Markdown、JSON 模板与配置，不引入新的执行面。

## 5. 依赖与本机验证

上游确定性脚本需要 Python 3.10+ 与 PyYAML。本机有一个容易踩的坑：

- PATH 上的 `python` → Python 3.13.14，但 **`import yaml` 失败**
- WorkBuddy 自带 `C:/Users/alisa/.workbuddy/binaries/python/envs/default/Scripts/python.exe` → Python 3.13.14，**PyYAML 6.0.3 已装**

因此跑上游脚本要用后者，不要直接用 `python`。已验证：

```
cangjie-tools v2.5.0
python: 3.13.14
  [ok] yaml
  [warn] tiktoken 缺失(可选)
  [warn] jsonschema 缺失(可选)
  [ok] schemas/capability-bundle.schema.json
  [ok] schemas/registry-entry.schema.json
  [ok] schemas/contracts/source-document.schema.json
doctor: PASS
```

`tiktoken` / `jsonschema` 为可选依赖，缺失不影响七阶段主流程。

## 6. 升级方式

本镜像是**静态快照**，不会自动跟随上游。升级时：

1. 重新 `git clone --depth 1` 上游到临时目录
2. 按第 3 节裁剪清单重建 `upstream/cangjie-core/`
3. 重跑第 4 节审计（尤其检查是否新增联网/凭据读取脚本）
4. 对照 `docs/04-integration.md` 复核挂载点是否仍然成立
5. 在本文件更新版本与快照日期

不要对 `upstream/` 内的文件做本地修改——上游编译器会检测生成目录的手改，且升级时会被覆盖。
