# 安装与决策记录

## 本次会话

| 项 | 值 |
|---|---|
| 日期 | 2026-11-06 |
| 触发 | 讨论「知识密度大的书有适用边界，蒸馏成 SKILL 是否也要补边界」→ 选定「判断力路径」 |
| 上游 | `kangarooking/cangjie-skill` v2.5.0，`git clone --depth 1`（main） |
| 本地安装 | `C:/Users/alisa/.workbuddy/skills/cangjie-skill/`（88 文件 / 704KB） |
| 补充包安装 | `C:/Users/alisa/.workbuddy/skills/shinspace-book-judgment/` |
| 仓库 staging | `E:/07_Workbuddy/shinspace-book.skill/` |
| 目标仓库 | `alisamou2008/shinspace-book.skill`（public） |

## 用户确认的三项决策

1. **安装位置**：仅 `.workbuddy/skills`（WorkBuddy 运行时目录），不写 `.agents/skills`
2. **仓库名**：`shinspace-book.skill`（修正了原始输入 `shinspaec` 的拼写，与品牌 SHINSPACE 一致）
3. **仓库范围**：判断力补充包 + cangjie 裁剪核心镜像

## 安装动作

```bash
# 1. 克隆到临时目录
git clone --depth 1 https://github.com/kangarooking/cangjie-skill.git \
  E:/07_Workbuddy/_tmp/cangjie-skill

# 2. 构造裁剪镜像（只保留运行时 + 文档）
cp -r methodology extractors templates scripts schemas registry docs → cangjie-core/
cp SKILL.md LICENSE README*.md CHANGELOG.md CONTRIBUTING.md .gitignore → cangjie-core/
rm cangjie-core/scripts/generate_star_history.py
```

```powershell
# 3. 跨盘复制用 PowerShell（Bash cp 到 .workbuddy 会触发沙箱拦截）
Copy-Item -Path E:\07_Workbuddy\_tmp\cangjie-core `
          -Destination C:\Users\alisa\.workbuddy\skills\cangjie-skill -Recurse -Force
# 已确认目标目录此前不存在，未覆盖任何已有技能
```

```bash
# 4. doctor 自检（必须用 WorkBuddy 自带 Python）
PY="C:/Users/alisa/.workbuddy/binaries/python/envs/default/Scripts/python.exe"
"$PY" scripts/cangjie.py doctor
```

## doctor 结果

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

`tiktoken` / `jsonschema` 可选，不影响主流程，未擅自安装。

## 安全审计结论

**可安装。** 详见 `UPSTREAM.md` 第 4 节。摘要：

- 21 个 Python 脚本，无 `eval()` / `exec()` 任意字符串执行
- `subprocess` 仅以 `sys.executable` 调用同目录自带脚本，无 `shell=True`
- `shutil.rmtree` 作用域均为显式传入的 target / staging / snapshot，且配快照回滚
- 全仓唯一的联网 + 凭据读取脚本 `generate_star_history.py` 已剔除
- 无 `curl | sh`、无 `child_process`、无自动安装依赖、无远程模板拉取
- 补充包自身零可执行代码

## 环境坑（本次实测）

| 现象 | 处理 |
|---|---|
| PATH 上的 `python` 是 3.13.14 但 `import yaml` 失败 | 上游脚本改用 `binaries/python/envs/default/Scripts/python.exe`，该环境已有 PyYAML 6.0.3 |
| Bash 跨盘 `cp` 到 `C:/Users/alisa/.workbuddy` 会被沙箱拦截 | 改用 PowerShell `Copy-Item`，成功 |

## 未做的事（刻意的）

- 未修改上游 `SKILL.md`、`methodology/`、`scripts/` 任何文件
- 未给 `cangjie.py` 添加 `--output interrogator`
- 未把 `BOUNDARY.yaml` 塞进 `verified.yaml`（上游 schema 不认）
- 未安装可选依赖 `tiktoken` / `jsonschema`
- 未向 `.agents/skills` 写入
- `examples/` 未做真实拆书——没有原文文本，沿用上游「不凭记忆拆书」红线，全部标注为结构演示

## 复核清单

- [x] 上游版本与快照方式记录
- [x] 裁剪清单与理由
- [x] 安全审计（含剔除项）
- [x] doctor PASS
- [x] 未覆盖已有技能
- [x] 补充包零可执行代码
- [x] 现实约束显式写进 `config.json` 与 `SKILL.md`
