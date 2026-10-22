# SPEC.md: "The Human How"; Exact technical requirements, environment setup, implementation details, etc.

## Execution Description

## Parameters

| Parameter | CLI Flag | Tested Values | Default | Description |
| --- | --- | --- | --- | --- |
| source_numa_node | `--source-numa-node` | smoke=0, baseline=0, extended=0 | 0 | From Parameter list; see Execution Description With Parameters. |
| target_numa_node | `--target-numa-node` | smoke=0,remote, baseline=0,remote, extended=0,remote | 0,remote | From Parameter list; see Execution Description With Parameters. |
| num_threads | `--num-threads` | smoke=1, baseline=4, extended=4 | 4 | From Parameter list; see Execution Description With Parameters. |
| thread_affinity | `--thread-affinity` | smoke=strict, baseline=strict, extended=strict | strict | From Parameter list; see Execution Description With Parameters. |
| page_size | `--page-size` | smoke=4KB, baseline=4KB, extended=4KB | 4KB | From Parameter list; see Execution Description With Parameters. |
| working_set_size | `--working-set-size` | smoke=32768,2097152,268435456, baseline=32768,262144,8388608,268435456, extended=32768,262144,8388608,268435456 | 32768,262144,8388608,268435456 | From Parameter list; see Execution Description With Parameters. |
| cache_level | `--cache-level` | smoke=L1,L3,DRAM, baseline=L1,L2,L3,DRAM, extended=L1,L2,L3,DRAM | L1,L2,L3,DRAM | From Parameter list; see Execution Description With Parameters. |
| access_pattern | `--access-pattern` | smoke=random, baseline=random, extended=random | random | From Parameter list; see Execution Description With Parameters. |
| stride | `--stride` | smoke=64, baseline=64, extended=64 | 64 | From Parameter list; see Execution Description With Parameters. |
| num_iterations | `--num-iterations` | smoke=20000, baseline=460000000, extended=1243000000 | 460000000 | From Parameter list; see Execution Description With Parameters. |

## Invocation

```bash
Build numa_sweep with cmake/g++; launch via collect_numa_cache.py with numactl; optionally wrap with perf stat -e cache-misses
```

## Raw Output Format

collect_numa_cache.py CSV written as the harness raw file. Per-size detail is also written to per_size.csv

check,source_numa_node,target_numa_node,num_threads,thread_affinity,page_size,working_set_size,cache_level,access_pattern,stride,num_iterations,status,cache_latency_nsec,local_numa_node_latency_nsec,remote_numa_node_latency_nsec,cross_socket_numa_penalty_ratio,local_dram_pointer_chase_bandwidth_gb_s,cache_miss_counters_misses_sec
summary,0,0,1,0,4KB,268435456,DRAM,random,64,1000000,ok,12,80,,,1.2,1000

## Metrics

- **#1: Largest cache-tier pointer-chase latency, ns** — stored as `cache_latency_nsec`.
- **#2: Local DRAM pointer-chase latency, ns** — stored as `local_numa_node_latency_nsec`.
- **#3: Pointer-chase payload rate, GB/s** — stored as `local_dram_pointer_chase_bandwidth_gb_s`.
- **#4: DRAM-tier cache misses/s** — stored as `cache_miss_counters_misses_sec`.

## Framework

Builds src/numa_sweep.cpp and runs scripts/collect_numa_cache.py (dispatched by scripts/collect_workload.py) for local/remote working-set sizes. source_numa_node/target_numa_node: intended bind pair (yaml 0 versus 0,remote). working_set_size: swept footprints.

## Installation and Execution Summary

Build src/numa_sweep.cpp and run a stride-64 pointer chase on yaml local and remote NUMA targets for each working_set_size, optionally under perf stat -e cache-misses, to measure local versus remote latency, cross-socket ratio, bandwidth, and miss rate. collect_numa_cache.py writes those metrics into the harness raw file

## Platform Portability

- **AMD (primary):** ```bash
Build numa_sweep with cmake/g++; launch via collect_numa_cache.py with numactl; optionally wrap with perf stat -e cache-misses
```
- **NVIDIA:** Native NVIDIA CUDA workload. Execute on the stated Ubuntu release with the host NVIDIA driver and CUDA userspace. ROCm porting notes do not apply.

## Model Context Protocols

- **Active:** None

## Execution-Loop Validation Contract

EXECUTION CHAIN: `run_benchmark.sh` ➔ raw output ➔ `scripts/parse_results.py` ➔ `results/benchmark.db` ➔ `scripts/validate_results.py`

This benchmark uses a lightweight, SQLite-integrated execution loop for result validation. All validation is performed by `scripts/validate_results.py`.

### Validation script usage

```bash
export BENCHMARK_PYTHON=/usr/bin/python3.13  # optional; select the installed interpreter

# After a live run:
".venv/bin/python" scripts/validate_results.py --db results/benchmark.db

# CI / no-GPU path (seeds fixture and validates it):
".venv/bin/python" scripts/validate_results.py --seed-fixture --quiet

# Override DB path via environment variable:
BENCHMARK_DB=tests/fixtures/benchmark.db \
  ".venv/bin/python" scripts/validate_results.py
```

### Run artifact contract

collect_numa_cache.py CSV written as the harness raw file. Per-size detail is also written to per_size.csv

check,source_numa_node,target_numa_node,num_threads,thread_affinity,page_size,working_set_size,cache_level,access_pattern,stride,num_iterations,status,cache_latency_nsec,local_numa_node_latency_nsec,remote_numa_node_latency_nsec,cross_socket_numa_penalty_ratio,local_dram_pointer_chase_bandwidth_gb_s,cache_miss_counters_misses_sec
summary,0,0,1,0,4KB,268435456,DRAM,random,64,1000000,ok,12,80,,,1.2,1000

```bash
bash run_benchmark.sh --help
bash run_benchmark.sh --profile smoke --validate
bash run_benchmark.sh --profile baseline --validate
bash run_benchmark.sh --profile extended --validate
```
`run_benchmark.sh --help` prints usage and exits. The harness calls `scripts/ensure_setup.sh` when `.setup_state` is absent.

### Required integrity checks (built into `validate_results.py`)

1. Latest run exists and `runs.status = 'ok'`.
2. `run.error_message` is NULL.
3. `started_at` and `finished_at` are valid ISO-8601 UTC strings.
4. All required aggregate metrics in `runs` are non-NULL and finite.
5. All required aggregate metrics are physically sensible (positive values). Builds src/numa_sweep.cpp and runs scripts/collect_numa_cache.py (dispatched by scripts/collect_workload.py) for local/remote working-set sizes. source_numa_node/target_numa_node: intended bind pair (yaml 0 versus 0,remote). working_set_size: swept footprints.
6. At least 2 sample rows exist for the latest `run_id` (sweep coverage).
7. No sample has `status = 'error'`.
8. Builds src/numa_sweep.cpp and runs scripts/collect_numa_cache.py (dispatched by scripts/collect_workload.py) for local/remote working-set sizes. source_numa_node/target_numa_node: intended bind pair (yaml 0 versus 0,remote). working_set_size: swept footprints.

### Baseline / Threshold configuration (`config/benchmark_config.yaml`)

Expected ranges and gates live in `config/benchmark_config.yaml` under `baselines:` or `thresholds:`. To update them, edit that file — never edit validation code directly.

Threshold key suffixes encode comparison direction when `thresholds:` is present: `_min` → observed value must be ≥ threshold. `_max` → observed value must be ≤ threshold. Informational `baselines:` ranges are not pass/fail gates.
