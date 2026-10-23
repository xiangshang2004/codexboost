---
name: codexboost
description: "Use during long or autonomous runs (goal mode, heartbeats, multi-attempt pipelines) and whenever results look wrong. Triggers: a step fails or WAITs the same way twice; a fix starts adding gates/proofs/diagnostics instead of moving the user's real outcome; a run 'succeeds' but the result is implausible (all-zero output, 0 detected, 0%/100% coverage, gold mismatch); a failure has sat as UNKNOWN for more than a day; you are about to rerun 'with more logging'. Keeps you on the user's outcome, removes self-imposed blockers, and drives hypothesis-led root-cause debugging."
---

# codexboost

Long autonomous runs rarely fail for lack of skill. They fail in two predictable ways:

1. **Stuck on the process.** Each failed attempt gets a new guard, proof or diagnostic, and the thing the user wants never happens. Part A is the brake.
2. **Stuck on the science.** A wrong result is parked as `UNKNOWN` because there is "no direct evidence", and nobody reasons it out. Part B is the engine.

Evidence rules decide what you **claim**. They never limit what you **investigate**.

Apply this workflow within current user authorization, budgets and safety constraints. Only remove self-imposed blockers; do not treat an explicit user requirement or an external protection as optional.

---

## A. Stay on the outcome

### A1. One outcome metric
Write the user's real outcome as one observable fact (e.g. "a real Z01X job is running", "the module's Δ is scored"). Before each action ask: does this move that fact, or only make my record of not-moving more complete? Receipts, hashes and reviews are means, not the metric. Keep a **progress counter**: it resets only when the metric moves or a *new* error class appears.

### A2. Two strikes → step back (mandatory, in writing)
When the same step fails or WAITs with the same error class twice:
1. **Label every blocker:** `AUTHOR` (quote the user's words) / `EXTERNAL` (hardware, vendor, OS, other people's jobs, a watchdog we do not own) / `OURS` (a threshold, assertion, gate or ordering we designed). Most long stalls are `OURS` treated as `EXTERNAL`.
2. **Cheapest path past each `OURS` blocker:** can the platform do it (kernel scheduling, vendor license queueing, the tool's own retry)? Prefer delete > record-only > relax > redesign > instrument.
3. **What would the user say?** If it is "why are you checking that at all?", stop and ask instead of building around it.
Never answer a twice-failed gate with a new diagnostic, sampler, retry window or contract version unless the gate is `AUTHOR` or `EXTERNAL`.

### A3. Three strikes, or ~1 hour without progress → escalate once
Put one plain question with 2–3 options and your recommendation at the top of your next report. Meanwhile move a different outcome; do not rerun the stuck pipeline with micro-variations.

### A4. Proportional evidence
Heavy machinery (once-tickets, hash pins, independent review, never-replay) only for actions that can double-launch real jobs, destroy data, spend money or publish. For probes and read-only checks, a command plus its output is enough. Before writing a diagnostic, name the decision it will change; if none, skip it. A job that started without error is not killed by our own rules; only its deadline, an explicit STOP, or the system (OOM/watchdog) ends it.

### A5. Reports lead with the outcome
First line: did the metric move (`仍未起／已起／已完成`), the single blocker and its provenance, the next action, any question. Hashes and paths go below. If the user cannot read it in 20 seconds, it failed.

### A6. Other agents
A reviewing agent's (e.g. Claude's) suggestions are proposals, weighed like your own `OURS` items; the user's quoted words outrank both.

---

## B. Find the root cause

### B1. A wrong result is a bug, whatever the exit code says
All-zero or constant output; 0 detected / 0% coverage; 100% everywhere; different configurations giving byte-identical results; gold mismatch; a number far from the last comparable run. Never record such a value as a result (e.g. Δ=0) until the cause is understood. `begun` without `completed`, or parts exiting −15, mean the report is partial.

### B2. The loop (in writing, in order)
1. **Symptom precisely**: what, since when, which variants share it. Shared symptoms → shared component; find the common base.
2. **Three ranked hypotheses with rough probabilities.** Guessing is required here. Include at least one "our setup/stimulus is wrong", not only "the tool is wrong".
3. **Diff against the nearest known-good**: official example, earlier passing run, sibling mode. Compare inputs, program/register **order and timing**, config values, environment. Bisect if the space is large.
4. **Read the source when logs are silent.** Logs are often muted; gate-level netlists drop RTL assertions. Grep RTL/TB/docs for the signal, register or assertion message (`grep -rn "Error!\|hazard" rtl/`). An assertion that would have fired is a top suspect.
5. **Do the arithmetic before running**: reset release, clock periods, init/flush lengths, latencies, buffer sizes, memory per worker. Five minutes of arithmetic beats a day of reruns.
6. **Cheapest discriminating check**: what separates the top two hypotheses? Prefer read-only (count lines, compare two files, read one register value). If a run is needed, the smallest one, with logging on only for the signal in question; write each hypothesis's prediction first.
7. **Update and repeat** until one hypothesis predicts every observation.
8. **Close the loop**: one paragraph cause → why it was silent → fix; then find every other job on the same base and fix them together.

### B3. Time-box
About 45 minutes on B2 steps 1–6 before any new run. Still stuck → send symptom, ranked hypotheses and the next check to the user or reviewing agent. Do not substitute gates, receipts or another instrumentation layer for reasoning.

### B4. Quick sanity checks
Did the measured thing actually happen? Is the value read from the address/unit/width/RAM type you think? Does an independent path agree (different parallelism, reference model, second tool)? Is it in the expected range?

---

## References (read only when relevant)
- [references/nvdla-wrong-result-checklist.md](references/nvdla-wrong-result-checklist.md): NVDLA / Z01X causes of all-zero output, 0 detected, memory sizing, and the CBUF-flush worked example. Read when simulation outputs, coverage or fault counts look wrong.
- [references/case-nvdla-cpu-gate.md](references/case-nvdla-cpu-gate.md): nine rounds against a self-imposed CPU gate. Read when stuck in admission gates, resource checks or `WAIT_UNSTARTED` loops.
