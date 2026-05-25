---
name: context-saver
description: >
  Save and load project snapshots for seamless context transfer between sessions.
  
  Use this skill when: the user says "存档", "保存进度", "上下文太长了", "备份", "transfer to new session", "continue in a new window". 
  Also use when user says "读档", "读取存档", "恢复进度", "load archive", "list-archives", "存档列表", "删档", "delete archive".
  
  This skill maintains an archive system in the project root's `archives/` directory.
  All archive selection uses interactive keyboard navigation (arrow keys + enter), NOT text input.
---

# Context Saver

Maintains an archive system in `<project_root>/archives/` for saving and loading experiment snapshots.

## Quick Reference

| Command | Action |
|---------|--------|
| `存档` / `save-archive` | Save a snapshot (interactive note input) |
| `读档` / `load-archive` | Select and load an archive (keyboard selection) |
| `存档列表` / `list-archives` | List all archives with summaries |
| `删档` / `delete-archive` | Delete an archive (keyboard selection + confirmation) |

## Archive Storage

- **Directory:** `<project_root>/archives/`
- **Naming:** `context_snapshot_{experiment}_{YYYYMMDD}_{NN}.md`
- **Index:** `<project_root>/archives/INDEX.md` (auto-generated list with summaries)

---

## Workflows

### 1. Save Archive (存档)

**Step 1:** Use `AskUserQuestion` to let the user add a note. Ask a single question with the preview showing a summary of what will be saved. The key design: the "Other" custom text input serves as the free-form note field.

```json
{
  "question": "为这份存档添加备注（可选，直接打字输入后点Other提交）：",
  "header": "存档备注",
  "options": [
    {"label": "不添加备注，直接存档", "description": "跳过备注"},
    {"label": "查看存档预览", "description": "先看要保存什么内容"}
  ]
}
```

- If user selects "不添加备注" → proceed without note.
- If user selects "查看存档预览" → show preview inline, then ask again with the same pattern.
- If user types in the "Other" field → use that text as the note.

**Step 2:** Gather all information using the template below. 信息收集完成后，**不要立即保存**。

**Step 3 (逐节审核):** 按 Section 0 → 1 → 2 → ... → 14 的顺序，每节单独展示并让用户确认。不允许一次展示全部。

每节审核用 `AskUserQuestion`，格式如下（注意：修改意见通过 Other 输入）：

```json
{
  "question": "=== Section X: 标题 ===\n\n[该节完整内容]\n\n---\n同意此节内容吗？选'要修改'通过Other输入意见：",
  "header": "Section X",
  "options": [
    {"label": "同意，下一节", "description": "此节正确"},
    {"label": "要修改", "description": "在Other输入修改内容"},
    {"label": "跳过此节", "description": "暂不处理，标记待补充"}
  ]
}
```

逐节审核规则：
- 用户选"同意" → 记下该节内容已确认，进入下一节
- 用户选"要修改"并在Other输入意见 → 根据意见修改内容 → 再次展示该节让用户确认 → 循环直到用户同意
- 用户选"跳过" → 在该节开头标注 **【待补充】**，进入下一节
- **不允许合并多个章节在一个问题里**

全部15节确认完后：
- 汇总所有标记了【待补充】的章节，询问用户是否还有信息可以补全
- 无法补全的 → 在存档开头 Note 中注明哪些章节不完整
- 用户可能补充之前未收集到的信息（如需手动贴结果）→ 按用户提供的内容更新对应章节

**Step 4:** 所有章节确认完毕后，保存完整存档文件并更新 INDEX.md。

**Step 5:** Confirm to the user: "存档已保存: archives/context_snapshot_xxx.md"

**Step 4:** Save the snapshot file and update INDEX.md.

**Step 5:** Confirm to the user: "存档已保存: archives/context_snapshot_xxx.md"

---

### 2. List Archives (存档列表 / list-archives)

Read `archives/INDEX.md` and display as a formatted list. Each entry shows:

```
[1] 2026-05-22 | MMLU distilled | 56.50% MMLU | 蒸馏完成, 未做AdvBench
[2] 2026-05-21 | VGD r=32 | 63.77% MMLU | Baseline run
```

---

### 3. Load Archive (读档 / load-archive)

**Step 1:** Read `archives/INDEX.md` to get the list of available archives.

**Step 2:** Use `AskUserQuestion` with one option per archive. Each option has:
- `label`: Short name (e.g., "0522 MMLU蒸馏")
- `description`: One-line summary from INDEX.md
- `preview`: First 30 lines of the archive file

This gives the user a keyboard-navigable list (arrow keys + enter to select).

**Step 3:** Read the selected archive file and present its full content.

**Step 4:** Tell the user: "已加载存档 [filename]。内容如上，可以开始继续工作了。"

---

### 4. Delete Archive (删档 / delete-archive)

**Step 1:** Read INDEX.md and present archives using `AskUserQuestion` (same as load).

**Step 2:** After user selects, ask for confirmation: "确定要删除 [filename] 吗？"

**Step 3:** If confirmed, delete the file and update INDEX.md.

---

## Snapshot Content Template

> **⚠️ 关键规则: 必须覆盖 ALL 14 个章节，每一个都不能跳过。**
> 存档的目的是让读档的人（压缩上下文后的自己）能直接进入工作状态，不需要问任何问题。
> 漏掉一个章节 = 读档后无法行动 = 存档无效。

**收集信息的方法：**
- 读 config.yaml、读脚本源码、读实验结果 JSON、扫目录结构、读 memory 文件
- 不要靠记忆写，要去读实际文件确认
- 如果某节信息无法获取（如文件不存在），写"未找到"而不是跳过该节

### 0. Research Overview（必须放最前面）

用2-3段说明整体研究背景，让读档的人立刻明白在做什么：

- **研究目标**: 这段研究在解决什么问题？（如：联邦大模型微调中的安全对齐保持）
- **核心方法**: 用了什么方法？（如：Fed-SaLoRA 双LoRA架构，safety adapter + task adapter）
- **两个框架各自的职责**:
  - **FS-LLM** (`FederatedScope-llm/`): 联邦学习训练框架，负责 FedAvg / Fed-SaLoRA 等方法的训练流程，数据异构划分（LDA）等
  - **SafeTuneBed** (`SafeTuneBed/`): 统一测评框架，负责加载adapter后在各个benchmark上评测（Commonsense 8任务、MMLU、AdvBench、MT-Bench、AlpacaEval）
- **当前阶段**: 在做什么实验？为什么做这个实验？

### 1. Project Experiment Settings (包括但不限于)
Read `config.yaml` from the experiment directory. Extract model type, LoRA configs (safety + task), training params, federated settings, distillation config, data config.

必须包含：model type, federate.method, federate.client_num, federate.total_round_num, data.type, data.splitter + alpha, train.optimizer + lr, llm.adapter.args (r, target_modules), llm.fed_salore.* (如果是Fed-SaLoRA)

### 2. Environment Configuration (包括但不限于)
Conda environments, sys.path insertions, environment variables, hardware specs.

必须包含：所有conda环境及路径、关键环境变量(OPENAI_API_KEY, OPENAI_BASE_URL, GPT_JUDGE_MODEL, NO_PROXY)、GPU型号和显存

### 3. External Models and API Configuration (包括但不限于)
All external models used (DeepSeek flash), API format, key locations, endpoint URLs.

必须包含：Judge模型名称和provider、API endpoint URL、API key存放位置、哪些benchmark需要API key

### 4. Config File Key Parameters Guide (包括但不限于)
Explain config.yaml sections, mark most-frequently-changed parameters.

必须包含：federate/、train/、data/、llm.adapter/、llm.fed_salore/ 各节的参数含义，标注哪些参数最常改

### 5. Existing Experiment Results (包括但不限于)
Table of all results (baselines + current), search all experiment directories.

必须包含：所有实验的对比表格（基座模型基线 + 所有方法 + 所有benchmark）。标注哪些结果是旧的不需要再关注的。格式：方法名 | 数据 | 异构度(α) | Commonsense | MMLU | AdvBench ASR | MT-Bench

### 6. Current Operation State (包括但不限于)
What's being worked on, last command, completion status.

必须包含：刚才在执行什么命令、完成到什么程度、输出结果是什么、有什么error/警告

### 7. Errors and Known Issues (包括但不限于)
Each bug: Symptom → Root Cause → Fix. Include pad_token OOB, config loading order, etc.

必须包含：每个问题的症状、原因、修复方法。标注哪些已修复、哪些未解决

### 8. Pending Tasks / Next Steps (包括但不限于)
Prioritized list with exact commands to run.

必须包含：下一步要做的事 + 精确命令（cd + python路径 + 全部参数），包含每项的状态（排队中 / 进行中 / 卡住）

### 9. Project Key Script Index (包括但不限于)
Scan scripts/ and evaluation/ directories. For each script: path, purpose, env, work dir, usage example, parameters, inputs, outputs.

Key scripts to document:
- `federatedscope/main.py` — FL training entry
- `scripts/run_distill_only.py` — standalone distillation
- `scripts/eval_distilled.py` — (legacy, avoid)
- `evaluation/evaluate.py` — unified evaluation
- `evaluation/mmlu_evaluator.py` — MMLU logic
- `evaluation/advbench_evaluator.py` — AdvBench + LLM judge
- `federatedscope/contrib/trainer/fed_salore_*.py` — trainer internals

必须包含：每个脚本的工作目录要求（cd到哪）、用哪个conda环境、输入输出文件、完整命令行示例

### 10. Data Flow Documentation (包括但不限于)
All datasets: path, size, format, purpose. Note format differences.

必须包含：每个数据集的完整路径、行数、字段结构(JSON key名)、用途(训练/蒸馏/测评)、源数据(来自哪个HF dataset或来源)

### 11. Experiment Directory Structure (包括但不限于)
Naming convention, file-by-file explanation, result file locations.

必须包含：命名规则解读（如 fed_salore_strong_100_vgd_20260519_6 = 方法_异构度_轮数_数据_日期_运行号）、目录内每个文件的含义（哪个是adapter、哪个是log、哪个是结果）

### 12. Memory / CLAUDE.md Content
Read and include MEMORY.md and linked files, any CLAUDE.md.

必须包含：把所有反馈型memory列入（discuss-then-execute, check-archives-after-compression, no-unsolicited-actions等）、关键参考型memory摘要（环境配置、默认训练参数）。标注memory中哪些信息已过时

### 13. Recent Operation Details (包括但不限于)
Chronological log of recent actions, commands, results, decisions.

必须包含：最近干了什么、按时间顺序列出重要操作和决策，包括失败的尝试和原因

### 14. Current Experiment Full Context (包括但不限于)
Research goal, current phase, what's validated, hypotheses, next milestone.

必须包含：为什么当前实验是这个参数组合、之前试过什么方案、哪些假设已被验证/推翻、下一步里程碑是什么

---

## Guiding Principles

1. **强制覆盖全部14+1节** — 存档必须包含 Section 0 (Research Overview) 到 Section 14。少一节就是无效存档。每节至少要写3行实质内容，不能写"暂无"跳过。
2. **Completeness over brevity** — 存档的目的是让读档的人不需要问问题就能继续工作。宁可太长不能太短。
3. **Actionable** — 每个命令必须是完整的可执行的（含cd路径、python路径、全部参数），不能写"跑训练"这种模糊描述，要写 `cd /root/drw/FederatedScope-llm && /root/miniconda3/envs/fs-llm/bin/python federatedscope/main.py --cfg xxx.yaml`
4. **读文件，不靠记忆** — 所有信息必须从实际文件读取（config.yaml、脚本源码、结果JSON、目录ls），不要凭记忆写。特别是实验结果，必须 cat 结果 JSON 文件确认数字。
5. **Don't lose unfinished work** — 记录所有正在跑的、跑了一半的、pending的命令和状态
6. **Don't let scripts be rewritten** — 记录已有脚本的路径和功能，避免被重写
7. **Keyboard selection** — ALL archive selection uses AskUserQuestion with options, NOT text input for IDs/names
8. **Interactive notes** — when saving, let the user type their note directly
9. **标注过时信息** — 如果发现memory或存档中有信息已过时（如参数已删除、模型名已改），用【过时】标记并说明正确的信息是什么
10. **逐节审核，不能批量** — 必须一节一节问，不能合并。用户对某节提修改意见后必须修改并重新展示该节，不能忽略
11. **用户说"不对"就是不对** — 用户指出的任何错误必须修正，不要辩解"我觉得这样没问题"。存档的目的是服务用户，不是证明自己正确
