# NVDLA / Z01X wrong-result checklist

These are debugging hypotheses for hardware simulation, not benchmark results or universal configuration requirements.

## Output all zero

| Suspect | Cheap check |
|---|---|
| Data is written into CBUF while the power-on flush is still active | Compare register-program order and timing with a known-good official trace; check for a `CDMA_S_CBUF_FLUSH_STATUS` poll before enabling CDMA |
| Read address or RAM type differs from what the writer uses | Compare observer address, destination base, MC/CV selection, memory width, and line/surface stride |
| Completion is observed before writes land | Read the memory model's commit point and the relevant write-response completion logic |
| Conversion or shifting zeroes the data | Check scale, shift, clipping, and output precision |
| Precision differs across the pipeline | Compare input, processing, and output precision in each configured module |
| The observer reads unwritten memory | Check initialization and four-state behavior; distinguish X from a value actually written as zero |

## Zero detected faults or implausible coverage

- A terminated partition, `All partitions failed fault simulation`, or `Tests with no faultsim results` means the report may be partial. Find the terminating owner before interpreting a score.
- Compare the observation point, strobe window, fault scope, and hierarchy against a passing case.
- Compare complete per-fault classifications under different parallelism only when the program, fault universe, observation contract, and simulation semantics are equivalent.

## Memory and runtime

Use measurements from the actual job. A worker can briefly hold overlapping simulator processes, so process count is not always equal to logical part count. Include the host's shared workload and the user's budget. Do not turn a footprint from one design into a universal per-worker constant.

## Worked example: a silent CBUF flush hazard

- **Symptom:** the build and execution finish without reported errors, but the software-visible output is all zero.
- **Shared base:** several precision variants use the same CDMA-to-CSC program. Investigate that shared setup first.
- **Diff against known-good:** the generated program enables CDMA earlier than the passing official trace.
- **Arithmetic:** estimate flush completion as reset release plus the required flush cycles multiplied by the core clock period, then compare it with actual program timing.
- **Source:** the RTL includes a flush/write hazard assertion. That assertion may be absent from the gate-level netlist, and quiet harness logs can hide the symptom.
- **Candidate fix:** poll flush completion before enabling CDMA; preserve the reference output, fault universe, observation point and observation window.
- **Verification:** rerun an affected case and compare the full output against its unchanged reference before claiming the cause is fixed.

This example illustrates a method. It does not publish experiment totals or claim that every zero-output failure has the same cause.
