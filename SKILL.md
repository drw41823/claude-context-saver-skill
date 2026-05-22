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

**Step 2:** Gather all information using the template below.

**Step 3:** Save the snapshot file and update INDEX.md.

**Step 4:** Confirm to the user: "存档已保存: archives/context_snapshot_xxx.md"

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

Include ALL sections below. Every time.

### 1. Project Experiment Settings (包括但不限于)
Read `config.yaml` from the experiment directory. Extract model type, LoRA configs (safety + task), training params, federated settings, distillation config, data config.

### 2. Environment Configuration (包括但不限于)
Conda environments, sys.path insertions, environment variables, hardware specs.

### 3. External Models and API Configuration (包括但不限于)
All external models used (DeepSeek flash), API format, key locations, endpoint URLs.

### 4. Config File Key Parameters Guide (包括但不限于)
Explain config.yaml sections, mark most-frequently-changed parameters.

### 5. Existing Experiment Results (包括但不限于)
Table of all results (baselines + current), search all experiment directories.

### 6. Current Operation State (包括但不限于)
What's being worked on, last command, completion status.

### 7. Errors and Known Issues (包括但不限于)
Each bug: Symptom → Root Cause → Fix. Include pad_token OOB, config loading order, etc.

### 8. Pending Tasks / Next Steps (包括但不限于)
Prioritized list with exact commands to run.

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

### 10. Data Flow Documentation (包括但不限于)
All datasets: path, size, format, purpose. Note format differences.

### 11. Experiment Directory Structure (包括但不限于)
Naming convention, file-by-file explanation, result file locations.

### 12. Memory / CLAUDE.md Content
Read and include MEMORY.md and linked files, any CLAUDE.md.

### 13. Recent Operation Details (包括但不限于)
Chronological log of recent actions, commands, results, decisions.

### 14. Current Experiment Full Context (包括但不限于)
Research goal, current phase, what's validated, hypotheses, next milestone.

---

## Guiding Principles

1. **Completeness over brevity** — include but not limited to the examples above
2. **Actionable** — every section helps the new session take action immediately
3. **Exact commands** — always include exact shell commands
4. **Don't lose unfinished work** — document exact state of in-progress work
5. **Don't let scripts be rewritten** — document existing scripts so they're reused
6. **Keyboard selection** — ALL archive selection uses AskUserQuestion with options, NOT text input for IDs/names
7. **Interactive notes** — when saving, let the user type their note directly
