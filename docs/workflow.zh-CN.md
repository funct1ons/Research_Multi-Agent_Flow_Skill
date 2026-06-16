# Research Multi-Agent Flow 中文指南

这份文档面向工作流的使用者，帮助你理解如何在个人科研项目中同时使用 Codex、执行 agent、Git、任务卡和实验记录。

这套工作流的目标不是让 agent 尽可能多地自动改代码，而是让每一次修改都有边界、可审查、可复现，并且能追溯到具体代码版本。

## 一、这套工作流解决什么问题

个人科研项目通常不是纯软件工程项目。它会同时包含模型代码、数据处理、训练脚本、实验配置、notebook、论文图表和实验结果。最常见的问题是：

- 代码一直在变，但不知道哪个版本生成了哪个结果。
- 本来只是想修一个 bug，agent 顺手改了训练逻辑或 baseline 配置。
- notebook 里跑出了结果，但半年后无法复现。
- 多个 agent 都在改代码，任务边界不清楚。
- 实验结果被写进论文，但缺少对应的 commit、config 和 command。

Research Multi-Agent Flow 的核心思路是：

```text
AGENTS.md 负责长期规则
tasks/*.md 负责具体任务
执行 agent 负责本地实现
handoff 负责交接记录
Codex 负责规划和审查
Git 负责隔离和事实记录
experiments/ 负责科研结果追踪
人类用户负责最终研究判断和合并决策
```

## 二、核心角色分工

### 你

你是研究负责人，而不是 agent 输出的搬运工。你负责：

- 定义研究问题。
- 决定任务优先级。
- 判断实验语义是否成立。
- 决定是否合并到 `main`。
- 决定实验结果是否可以写进论文。

### Codex

Codex 更适合做高层规划和审查：

- 拆解研究任务。
- 生成任务卡。
- 审查 diff 和 handoff。
- 检查实验语义风险。
- 检查数据处理、训练循环、指标计算和论文结果风险。
- 总结实验结果和辅助写作。

Codex 在 review 时默认应该只读，不直接修改代码，除非你明确要求它实现。

### 执行 agent

执行 agent 是一个角色，不是固定工具。它可以是 OpenCode、Reasonix、Codex 的低成本 subagent，或者其他能读取任务卡、修改仓库、运行验证命令并填写 handoff 的本地 coding agent。

执行 agent 适合做本地执行：

- 按任务卡修改代码。
- 补测试。
- 跑 pytest、lint、dry-run。
- 生成配置文件。
- 整理 scripts。
- 根据报错做局部修复。
- 完成后填写 handoff。

执行 agent 不应该自由决定实验设计，也不应该随意修改 baseline 语义。

如果 Codex 的低成本 subagent 被用作执行 agent，负责规划和 review 的 Codex 仍然必须以任务卡、handoff、验证输出和 `git diff` 为事实依据，而不是直接相信 subagent 的总结。

### Git

Git 是这套工作流的硬边界：

- `main` 保持稳定。
- 每个任务开短生命周期分支或 worktree。
- `git diff` 暴露 agent 实际改了什么。
- commit 记录确认过的状态。
- commit hash 绑定实验结果。

## 三、项目中需要哪些文件

一个使用这套工作流的科研项目，最小需要这些文件：

```text
AGENTS.md
tasks/
LOG.md
experiments/
.gitignore
```

推荐结构：

```text
my-research-project/
  AGENTS.md
  LOG.md
  README.md
  .gitignore

  tasks/
    README.md
    2026-06-16_add-dropout-ablation.md

  src/
  configs/
  scripts/
  tests/
  notebooks/

  experiments/
    2026-06-16_dropout_ablation/
      configs/
      command.txt
      commit.txt
      metrics.csv
      notes.md

  paper/
```

## 四、任务卡状态

任务卡使用一个小状态机，不需要复杂项目管理系统。

状态含义：

- `planned`：任务卡已创建，但还没有开始实现。
- `in_progress`：实现 agent 正在执行任务。
- `implemented`：实现完成，handoff 已填写，但还没有 review。
- `needs_review`：等待 Codex 或人类 review。
- `changes_requested`：review 后发现需要修改。
- `ready_to_merge`：review 和验证通过，等待人类决定是否合并。
- `done`：任务已合并、关闭，或人类明确接受。
- `blocked`：当前任务无法推进，需要人类决策、外部资源或重新拆分任务。
- `abandoned`：任务明确放弃。

正常转换：

```text
planned -> in_progress
in_progress -> implemented
implemented -> needs_review
needs_review -> changes_requested
changes_requested -> in_progress
needs_review -> ready_to_merge
ready_to_merge -> done
```

例外转换：

```text
任何非终态 -> blocked
blocked -> planned / in_progress
任何非终态 -> abandoned
```

关键规则：

- 只有实现 agent 应该把 `in_progress` 改成 `implemented`，并且必须先填写 handoff。
- 只有 review 后才能进入 `ready_to_merge`。
- 只有人类用户可以把任务标记为 `done`。
- `done` 和 `abandoned` 是终态，除非人类明确重新打开任务。

## 五、什么时候需要完整流程

不是所有改动都需要完整任务卡。

可以轻量处理的情况：

- README 中修一个 typo。
- 改一行无风险说明文字。
- 更新明显不影响行为的格式。

应该走完整流程的情况：

- 修改代码行为。
- 修改训练、评估、数据处理或指标计算。
- 修改实验配置。
- 生成论文表格或图。
- 任何影响 baseline 可比性的改动。
- 任何需要执行 agent 实现、Codex review 的任务。

经验规则：

如果一个改动影响实验语义，或者会改动两个以上关键文件，就创建任务卡。

## 六、日常工作流：从早上到合并

下面是实际怎么用。

### Step 1：早上回到稳定状态

先回到 `main`：

```bash
git checkout main
git status
```

你希望看到：

```text
nothing to commit, working tree clean
```

然后跑一个最小检查：

```bash
pytest
```

或者：

```bash
python scripts/run_train.py --config configs/debug.yaml
```

这个检查不一定要完整训练，它的目的只是确认 `main` 当前没有明显损坏。

### Step 2：用 Codex 规划今天的任务

不要一上来就让 agent 改代码。先让 Codex 把目标拆成小任务。

可以这样对 Codex 说：

```text
请根据当前研究目标，把今天的工作拆成 2 个适合执行 agent 执行的小任务。
每个任务需要包含：
- Goal
- Background
- Allowed files
- Forbidden files
- Acceptance criteria
- Validation commands
- Review checklist
```

Codex 的输出应该变成 `tasks/*.md` 中的任务卡。

你需要审查任务卡：

- 任务是否太大？
- allowed files 是否太宽？
- forbidden files 是否合理？
- 验收标准是否可执行？
- 验证命令是否现实？

### Step 3：为一个任务开分支

例如今天要做 dropout ablation 配置：

```bash
git checkout -b exp/dropout-ablation
```

规则：

- 一个任务，一个分支。
- 同一时间，一个分支只让一个实现 agent 修改代码。
- Codex 可以 review，但不要和执行 agent 同时在同一分支上改代码。

### Step 4：执行 agent 执行任务卡

给执行 agent 的指令应该非常明确：

```text
请读取 tasks/2026-06-16_add-dropout-ablation.md，并严格按任务卡执行。

要求：
- 只修改 allowed files
- 不要修改 forbidden files
- 如果需要超出 allowed files，停止并说明原因
- 完成后运行 validation commands
- 更新任务卡中的 Handoff 部分
```

执行 agent 执行期间，任务卡状态可以从：

```text
planned -> in_progress
```

实现完成并填写 handoff 后，状态变为：

```text
in_progress -> implemented
```

### Step 5：你先看 Git diff

不要直接相信 agent 的总结。先看：

```bash
git status
git diff
```

重点检查：

- 是否只改了 allowed files？
- 有没有改 forbidden files？
- 有没有大范围重构？
- 有没有改 baseline？
- 有没有改论文结果？
- 有没有生成大文件？
- 有没有把实验结果写死？

如果发现明显越界，先让执行 agent 解释或回退，不要进入 review。

### Step 6：Codex review

把任务状态改为：

```text
implemented -> needs_review
```

然后让 Codex 做只读审查：

```text
请读取任务卡和 Handoff，review 当前分支相对 main 的 diff。

不要修改代码，只输出：
1. 是否满足任务卡
2. 是否有越界修改
3. 是否有实验语义风险
4. 是否缺少验证
5. 是否建议合并
6. 如果不建议合并，请给出 follow-up task card
```

如果 Codex 发现问题，任务状态变为：

```text
needs_review -> changes_requested
```

然后让执行 agent 做一个有边界的修正任务：

```text
changes_requested -> in_progress
```

如果 Codex 认为可以合并，并且验证结果足够，状态变为：

```text
needs_review -> ready_to_merge
```

### Step 7：提交

确认没问题后提交：

```bash
git add .
git commit -m "exp: add dropout ablation configs"
```

commit message 建议使用简单类型：

- `fix:` 修 bug
- `feat:` 加功能
- `exp:` 实验相关
- `test:` 测试
- `refactor:` 重构
- `paper:` 论文相关
- `docs:` 文档

### Step 8：合并回 main

个人科研项目推荐 squash merge：

```bash
git checkout main
git merge --squash exp/dropout-ablation
git commit -m "exp: add dropout ablation configs"
```

合并后，人类用户可以把任务状态改为：

```text
ready_to_merge -> done
```

如果暂时不合并，可以保持 `ready_to_merge`，并在任务卡或 `LOG.md` 中说明原因。

### Step 9：记录实验

如果这个任务会影响实验结果，就记录实验版本。

```bash
mkdir -p experiments/2026-06-16_dropout_ablation
git rev-parse HEAD > experiments/2026-06-16_dropout_ablation/commit.txt
```

实验目录建议包含：

```text
experiments/2026-06-16_dropout_ablation/
  configs/
  command.txt
  commit.txt
  metrics.csv
  notes.md
```

`notes.md` 可以写：

```markdown
# Dropout ablation

## Goal

Compare dropout values 0.0, 0.1, and 0.3 under the baseline setup.

## Code version

See `commit.txt`.

## Command

See `command.txt`.

## Notes

This experiment only creates and validates configs. Full training still needs to be run.

## Risks

Need 3-seed average before using the result in a paper claim.
```

### Step 10：晚上收尾

收工前检查：

```bash
git status
```

然后更新 `LOG.md`：

```markdown
## 2026-06-16

### Done

- Added dropout ablation configs.
- Ran config validation and dry-run.
- Merged task into main.

### Not validated yet

- Full training not run.
- No 3-seed average yet.

### Risks

- Need to confirm dropout is actually applied in model construction.

### Next

- Run dropout values 0.0, 0.1, 0.3 with seeds 1, 2, 3.
- Generate summary table.
```

最后推送：

```bash
git push
```

## 七、完整示例：新增 dropout ablation 配置

目标：比较 `dropout = 0.0, 0.1, 0.3`。

Codex 生成任务卡：

````markdown
# Task: Add dropout ablation configs

## Execution agent

OpenCode

## Status

planned

## Branch

exp/dropout-ablation

## Goal

Add config files for dropout ablation based on the current baseline.

## Background

We want to compare dropout values 0.0, 0.1, and 0.3 while keeping all other key baseline settings unchanged.

## Allowed files

- `configs/`
- `tests/test_config.py`

## Forbidden files

- `src/model.py`
- `src/data.py`
- `scripts/run_train.py`
- `paper/`

## Acceptance criteria

- Add `configs/ablation_dropout_0.0.yaml`
- Add `configs/ablation_dropout_0.1.yaml`
- Add `configs/ablation_dropout_0.3.yaml`
- Keep all non-dropout key settings aligned with baseline.
- Add or update config loading tests if needed.
- Do not generate or invent experiment results.

## Validation commands

```bash
pytest tests/test_config.py
python scripts/run_train.py --config configs/ablation_dropout_0.1.yaml --dry-run
```

## Review required

Codex review required before merge.

Focus on:

- Whether only dropout differs from baseline.
- Whether seed settings are explicit.
- Whether config names are clear.
- Whether 3-seed follow-up is needed.
````

执行 agent 实现后填写 handoff。你检查 diff，确认没有改 `src/model.py`、`src/data.py`、`scripts/run_train.py` 或 `paper/`。Codex review 通过后，你提交并 squash merge。正式跑实验后，再把 config、command、commit、metrics 和 notes 放进 `experiments/`。

## 八、常见错误

### 错误 1：把 AGENTS.md 写得太长

`AGENTS.md` 应该写稳定规则，不要塞入当天任务、长篇论文背景或实验流水账。

### 错误 2：任务卡太大

“优化整个训练系统”不是好任务。应该拆成多个小任务，例如“补 config loader 测试”“新增 dropout ablation 配置”“修复 dry-run 参数解析”。

### 错误 3：Codex review 时顺手改代码

review 默认只读。发现问题后，优先生成 follow-up task card，再交给执行 agent 实现。

### 错误 4：实验结果没有 commit hash

重要实验必须记录：

```bash
git rev-parse HEAD
```

否则以后无法确认论文里的数字来自哪版代码。

### 错误 5：把执行 agent 当成研究决策者

执行 agent 适合执行任务卡，不适合判断实验是否支撑论文 claim。研究判断必须由你负责，Codex 可以辅助审查。

## 九、最小可执行版本

如果你刚开始使用，只需要遵守这几个规则：

1. `main` 保持稳定。
2. 每个非平凡任务写一个任务卡。
3. 每个任务开一个短生命周期分支。
4. 执行 agent 按任务卡实现并写 handoff。
5. 你先看 `git diff`。
6. Codex 做只读 review。
7. 验证后再 commit 和合并。
8. 重要实验记录 commit hash。

这就是最小可用的 Research Multi-Agent Flow。
