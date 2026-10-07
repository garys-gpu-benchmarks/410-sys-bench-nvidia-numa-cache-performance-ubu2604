# PRD.md:  "The Why"; Product requirements, benchmark metadata table, high-level requirements, etc.

Product Requirements Document

"The Why"; Product requirements, benchmark metadata table, high-level requirements, etc. Defines the benchmark goal, validation objective, test name, benchmark number, category, and high-level success criteria.

## Benchmark Matrix Document Metadata (via benchmark_specification.json)

This PRD.md section is populated from benchmark_specification.json, which is the structured source of benchmark-specific product requirements.

## Workload Number
410

## Workload Name
NUMA Cache Latency Sweep

## Execution Summary (Run and Measure)
Build src/numa_sweep.cpp and run a stride-64 pointer chase on yaml local and remote NUMA targets for each working_set_size, optionally under perf stat -e cache-misses, to measure local versus remote latency, cross-socket ratio, bandwidth, and miss rate. collect_numa_cache.py writes those metrics into the harness raw file

## Main Goal
Measure NUMA local vs remote stride-walk latency

## Validation Objective
Validates paired local/remote numa_sweep runs. On single-node or membind-denied hosts, remote latency may equal local

## Workload Category
Memory, Bandwidth & Data Movement

## Validation Requirement

The benchmark must include an automated SQLite-integrated validation layer that verifies persisted results from `results/benchmark.db`. Validation must confirm:

1. The benchmark run completed successfully with no tool errors.
2. Required samples and aggregate metrics were persisted for every swept shape.
3. Metrics are finite and physically sensible (positive, within plausible bounds).
4. Measured values satisfy configured thresholds when the workload defines pass/fail gates.
5. The benchmark fails validation when required data is missing, invalid, or outside bounds.

## Non-Functional Requirements

| Requirement | Target |
|---|---|
| Automation | Runs to completion without manual intervention after `bash run_benchmark.sh` |
| Idempotency | Re-running `run_benchmark.sh` appends a new run; never corrupts existing rows |
| Persistence | All metrics survive script exit; `results/benchmark.db` is the durable record |
