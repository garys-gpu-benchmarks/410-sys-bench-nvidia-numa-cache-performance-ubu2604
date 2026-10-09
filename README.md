# NUMA Cache Latency Sweep Benchmark

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![CI](https://github.com/garys-gpu-benchmarks/410-sys-bench-nvidia-numa-cache-performance-ubu2604/actions/workflows/ci.yml/badge.svg)](https://github.com/garys-gpu-benchmarks/410-sys-bench-nvidia-numa-cache-performance-ubu2604/actions/workflows/ci.yml)

Target: Ubuntu 26.04 · NVIDIA · see Hardware Requirements. This is a host benchmark, not a laptop `pip install` project.

## Quick Start

```bash
git clone https://github.com/garys-gpu-benchmarks/410-sys-bench-nvidia-numa-cache-performance-ubu2604.git
cd 410-sys-bench-nvidia-numa-cache-performance-ubu2604
sudo bash setup.sh --assume-yes
bash run_benchmark.sh --profile smoke --validate
```
Results are written to `results/benchmark.db` and `results/summary.json`.

This workload is executed on the validation host after the repository is copied there. `setup.sh` and `run_benchmark.sh` do not open an outbound SSH session.

Prerequisites: Ubuntu 26.04; NVIDIA; Python 3.14.4; root or sudo for `setup.sh`. Framework: Bash, SQLite, Python, PyYAML, C++, CMake, GCC, numactl, Linux perf (optional). This is a host benchmark, not a laptop `pip install` project.

```mermaid
flowchart LR
  setup.sh --> run_benchmark.sh --> parse_results.py --> results/benchmark.db
```

## 1. Overview

Builds src/numa_sweep.cpp and runs scripts/collect_numa_cache.py (dispatched by scripts/collect_workload.py) for local/remote working-set sizes. source_numa_node/target_numa_node: intended bind pair (yaml 0 versus 0,remote). working_set_size: swept footprints. num_threads, page_size, cache_level, access_pattern, stride, and num_iterations come from yaml. Sweep dimensions: source_numa_node, target_numa_node, num_threads, thread_affinity, page_size, working_set_size, cache_level, access_pattern.

## 2. What It Validates

- Validates paired local/remote numa_sweep runs. On single-node or membind-denied hosts, remote latency may equal local
- #1: Largest cache-tier pointer-chase latency, ns (cache_latency_nsec); is present and physically sensible.
- #2: Local DRAM pointer-chase latency, ns (local_numa_node_latency_nsec); is present and physically sensible.
- #3: Pointer-chase payload rate, GB/s (local_dram_pointer_chase_bandwidth_gb_s); is present and physically sensible.
- #4: DRAM-tier cache misses/s (cache_miss_counters_misses_sec) is present and physically sensible.

## 3. Metrics Captured

- **#1: Largest cache-tier pointer-chase latency, ns** — stored as `cache_latency_nsec`.
- **#2: Local DRAM pointer-chase latency, ns** — stored as `local_numa_node_latency_nsec`.
- **#3: Pointer-chase payload rate, GB/s** — stored as `local_dram_pointer_chase_bandwidth_gb_s`.
- **#4: DRAM-tier cache misses/s** — stored as `cache_miss_counters_misses_sec`.

## 4. Hardware Requirements

### Supported environment

- OS: Ubuntu 26.04
- GPU vendor: NVIDIA
- Framework family: Bash, SQLite, Python, PyYAML, C++, CMake, GCC, numactl, Linux perf (optional)
- Python: Python 3.14.4

### Reference validation environment

The tables below describe the machine used to generate the reference results. They are not a requirement that every user buy that exact cloud instance.

### System

Builds src/numa_sweep.cpp and runs scripts/collect_numa_cache.py (dispatched by scripts/collect_workload.py) for local/remote working-set sizes. source_numa_node/target_numa_node: intended bind pair (yaml 0 versus 0,remote). working_set_size: swept footprints.

### GPU

Ubuntu 26.04 / NVIDIA / Bash, SQLite, Python, PyYAML, C++, CMake, GCC, numactl, Linux perf (optional)

## 5. Software Requirements

| Component | Version |
|---|---|
| OS | Ubuntu 26.04 |
| Kernel | kernel 7.0.0 |
| Python | Python 3.14.4 |
| ROCm | CUDA 13.3 |
| rocBLAS | N/A - rocBLAS not used |

Builds src/numa_sweep.cpp and runs scripts/collect_numa_cache.py (dispatched by scripts/collect_workload.py) for local/remote working-set sizes. source_numa_node/target_numa_node: intended bind pair (yaml 0 versus 0,remote). working_set_size: swept footprints.

## 6. Installation

## 7. Running the Benchmark

```bash
Build numa_sweep with cmake/g++; launch via collect_numa_cache.py with numactl; optionally wrap with perf stat -e cache-misses
```

**Validating results separately:**

```bash
export BENCHMARK_PYTHON=/usr/bin/python3.13  # optional
python3 -m venv .venv
source ".venv/bin/activate"
".venv/bin/python" scripts/validate_results.py
```

## 8. Output

### `results/benchmark.db` (SQLite)

collect_numa_cache.py CSV written as the harness raw file. Per-size detail is also written to per_size.csv

check,source_numa_node,target_numa_node,num_threads,thread_affinity,page_size,working_set_size,cache_level,access_pattern,stride,num_iterations,status,cache_latency_nsec,local_numa_node_latency_nsec,remote_numa_node_latency_nsec,cross_socket_numa_penalty_ratio,local_dram_pointer_chase_bandwidth_gb_s,cache_miss_counters_misses_sec
summary,0,0,1,0,4KB,268435456,DRAM,random,64,1000000,ok,12,80,,,1.2,1000

```bash
Build numa_sweep with cmake/g++; launch via collect_numa_cache.py with numactl; optionally wrap with perf stat -e cache-misses
```

### `results/summary.json`

Consolidated metrics from the most recent run — suitable for CI artifact upload or dashboard ingestion.

### `results/raw/<timestamp>.txt`

collect_numa_cache.py CSV written as the harness raw file. Per-size detail is also written to per_size.csv

check,source_numa_node,target_numa_node,num_threads,thread_affinity,page_size,working_set_size,cache_level,access_pattern,stride,num_iterations,status,cache_latency_nsec,local_numa_node_latency_nsec,remote_numa_node_latency_nsec,cross_socket_numa_penalty_ratio,local_dram_pointer_chase_bandwidth_gb_s,cache_miss_counters_misses_sec
summary,0,0,1,0,4KB,268435456,DRAM,random,64,1000000,ok,12,80,,,1.2,1000

## 9. Baselines / Thresholds

Expected ranges and gates live in `config/benchmark_config.yaml` under `baselines:` or `thresholds:`. To update them, edit that file — never edit validation code directly.

## 10. Troubleshooting

**`setup.sh` missing collector**
Create cannot finish without `scripts/collect_workload.py`.

**`self_check` overlay rewritten**
Do not overwrite files listed in `results/overlay_lock.json`.

**Remote SSH drop during setup**
Reconnect and resume `bash setup.sh --assume-yes`. Do not wipe `.venv` or `.cache`.

## 11. NVIDIA H100 Coding Differences

Native NVIDIA CUDA workload. Execute on the stated Ubuntu release with the host NVIDIA driver and CUDA userspace. ROCm porting notes do not apply.

## Repository layout

```text
.
├── setup.sh
├── run_benchmark.sh
├── benchmark_specification.json
├── .github/workflows/      # thin CI callers (see Continuous Integration)
├── config/
├── scripts/
├── src/
├── tests/
├── docs/
├── results/
└── LICENSE
```

## Continuous Integration

| Workflow | Runs on | When | What it does |
|---|---|---|---|
| [CI](.github/workflows/ci.yml) | GitHub-hosted runner | every pull request, and every push to `main` | shellcheck, ruff, `bash -n`, `compileall`, `run_benchmark.sh --help`, specification schema, the results validator on a seeded fixture, required files, and actionlint. No GPU and no benchmark run. |
| [GPU Smoke Benchmark](.github/workflows/gpu-smoke.yml) | self-hosted runner labeled `gpu`, `nvidia`, `ubu2604` | only when started by hand: **Actions → GPU Smoke Benchmark → Run workflow** (choose `smoke`, `baseline` or `extended`) | Verifies the pre-provisioned GPU stack, records `results/environment.json` (driver, runtime, kernel, GPU), runs the profile with `--validate`, shows headline metrics on the run page, and uploads the results. |

Both files are short callers. The steps themselves live once, for every workload in the suite, in [`garys-gpu-benchmarks/shared-workflows`](https://github.com/garys-gpu-benchmarks/shared-workflows), pinned at `@v1`. The GPU workflow is never triggered by pull requests, so code from a fork cannot run on the GPU host.

### Running it as part of the NVIDIA Ubuntu 26.04 bundle

This repository is one of the 32 workloads in [`bundle-nvidia-ubuntu-2604`](https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2604), which holds them as git submodules. To put the whole bundle on a GPU host and run this workload from it:

```bash
git clone --recurse-submodules https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2604 /opt/benchmarks
cd /opt/benchmarks/410-sys-bench-nvidia-numa-cache-performance-ubu2604
bash run_benchmark.sh --profile smoke --validate
```
