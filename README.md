# codexboost

**Keep an autonomous coding agent focused on the user's outcome, and make it reason when results look wrong.**

codexboost is a lightweight [Codex skill](SKILL.md), initially written with Claude for use in Codex. It combines two playbooks:

- **Outcome first:** notice repeated stalls, distinguish user requirements from external constraints and self-imposed blockers, and remove unnecessary process.
- **Root cause first:** treat implausible results as bugs, rank hypotheses, compare against a known-good example, read the source, do the arithmetic, and choose a cheap discriminating check.

It is a set of instructions, not a background service or a permissions mechanism. It does not change access rights, launch jobs, disable protections, or grant permission to exceed a user's budget.

## When to use it

Use it during long autonomous runs, repeated failures or WAIT states, and investigations where a tool exits successfully but its output is implausible: all zeros, unexplained 0% or 100% coverage, or a mismatch with the reference.

The central question is: **did the observable result move, or did we only produce more records of not moving?**

## Install

Clone into the user-level skills directory described in the [official Codex documentation](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills):

```sh
mkdir -p "$HOME/.agents/skills"
git clone https://github.com/xiangshang2004/codexboost.git \
  "$HOME/.agents/skills/codexboost"
```

If that directory already exists, preserve your current copy before choosing how to update it. Codex detects skill changes automatically; restart it if the new skill does not appear.

Invoke it explicitly:

```text
Use $codexboost to get this stalled task moving and investigate the wrong result.
```

For repeated autonomous work, you can also add a short pointer to your own `AGENTS.md`:

```text
During long autonomous runs, repeated stalls, or implausible results, apply codexboost.
```

## What it changes

1. Define one observable outcome.
2. After two similar stalls, label blockers as **AUTHOR / EXTERNAL / OURS**.
3. Prefer removing an unnecessary self-imposed blocker over building another layer around it.
4. Escalate unresolved choices concisely while advancing independent work.
5. For wrong results, write three ranked hypotheses and compare against a known-good case.
6. Read source and calculate timing, size, memory, or runtime before choosing the next experiment.
7. Verify the fix and check other jobs sharing the same faulty base.

Explicit user instructions, external constraints, safety limits and ownership remain authoritative. A check being inconvenient is not evidence that it can be removed.

## Files

| File | Purpose |
|---|---|
| [SKILL.md](SKILL.md) | Core workflow and trigger conditions |
| [agents/openai.yaml](agents/openai.yaml) | Codex UI metadata |
| [CPU-gate case](references/case-nvdla-cpu-gate.md) | How repeated self-imposed admission checks stalled useful work |
| [Wrong-result checklist](references/nvdla-wrong-result-checklist.md) | A hardware-simulation example of hypothesis-led debugging |

The NVDLA examples are domain-specific illustrations, not universal instructions to disable resource or license checks. The public examples omit private host details and unpublished experiment totals.

## 中文说明

这是一个给 Codex 用的“防钻牛角尖＋根因优先”skill：

- 同类失败或等待反复出现时，先分清哪些是用户要求、外部限制、自己加的门槛。
- 结果明显不对时，先列假设、比对官方或已通过实例、读源码和算时间账，再做能区分假设的检查。
- 以真实任务结果衡量进展，避免用更多计划、哈希、日志和检查代替执行。
- 不覆盖用户授权、预算、进程所有权和必要保护。

## License

[MIT](LICENSE).
