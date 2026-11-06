# NUMA Cache Latency Sweep Benchmark

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![CI](https://github.com/garys-gpu-benchmarks/410-sys-bench-nvidia-numa-cache-performance-ubu2604/actions/workflows/ci.yml/badge.svg)](https://github.com/garys-gpu-benchmarks/410-sys-bench-nvidia-numa-cache-performance-ubu2604/actions/workflows/ci.yml)

Measures NUMA local vs remote stride-walk latency. Workload 410 of the NVIDIA Ubuntu 26.04 bundle. This is a CPU benchmark: no GPU is used.

## Quick Start

On an Ubuntu 26.04 host, as root or with sudo:

```bash
git clone https://github.com/garys-gpu-benchmarks/410-sys-bench-nvidia-numa-cache-performance-ubu2604.git
cd 410-sys-bench-nvidia-numa-cache-performance-ubu2604
sudo bash setup.sh --assume-yes
bash run_benchmark.sh --smoke
```

`run_benchmark.sh` runs setup by itself if it has not been run yet. Each run prints its metrics when it finishes.

## Profiles and Parameters

| Parameter | Flag | smoke | baseline | extended |
|---|---|---|---|---|
| `source_numa_node` | `--source-numa-node` | 0 | 0 | 0 |
| `target_numa_node` | `--target-numa-node` | 0,remote | 0,remote | 0,remote |
| `num_threads` | `--num-threads` | 1 | 4 | 4 |
| `thread_affinity` | `--thread-affinity` | strict | strict | strict |
| `working_set_size` | `--working-set-size` | 32768,2097152,268435456 | 32768,262144,8388608,268435456 | 32768,262144,8388608,268435456 |
| `cache_level` | `--cache-level` | L1,L3,DRAM | L1,L2,L3,DRAM | L1,L2,L3,DRAM |
| `access_pattern` | `--access-pattern` | random | random | random |
| `stride` | `--stride` | 64 | 64 | 64 |
| `num_iterations` | `--num-iterations` | 20000 | 460000000 | 1243000000 |

Pick a profile with `--smoke` (default), `--baseline` or `--extended`. Any flag in the table overrides that profile's value:

```bash
bash run_benchmark.sh --baseline --num-threads 2
bash run_benchmark.sh --help        # every option
```

Profile values live in `config/benchmark_config.yaml`.

## Metrics

| # | Metric | Key |
|---|---|---|
| 1 | Largest cache-tier pointer-chase latency, ns | `cache_latency_nsec` |
| 2 | Local DRAM pointer-chase latency, ns | `local_numa_node_latency_nsec` |
| 3 | Pointer-chase payload rate, GB/s | `local_dram_pointer_chase_bandwidth_gb_s` |
| 4 | DRAM-tier cache misses/s | `cache_miss_counters_misses_sec` |

### Notes

- The smoke profile is a quick check with reduced sizes and iteration counts. Use `--baseline` or `--extended` for representative numbers.
- numactl --membind may be denied, making remote equal local.
- Blocked perf records zero cache misses.
- Single-node hosts have no remote node.

## Output

| Path | Contents |
|---|---|
| `results/raw/YYYYMMDD_HHMMSSZ_WORKLOAD-<NNN>_SYSTEM-<hostname>/` | Raw tool output, `metrics_summary.txt`, and `hardware_info.txt`, `software_info.txt`, `errors_info.txt` |
| `results/benchmark.db` | SQLite history of every run |
| `results/summary.json` | Metrics from the most recent run |
| `/var/opt/benchmarks/runtime_ledger.csv` | One row per run, shared by every workload on the host |

Every run is checked for completeness and against any `thresholds:` or `baselines:` in `config/benchmark_config.yaml`. A run that fails the checks exits non-zero.

## Requirements

- Ubuntu 26.04, root or sudo for `setup.sh`
- C++, CMake, GCC, numactl, Linux perf (optional), Python 3.14. `setup.sh` installs what is missing; Python packages go into a repo-local `.venv`.

## Run the Whole Bundle

This workload is also part of [`bundle-nvidia-ubuntu-2604`](https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2604), which holds all 32 NVIDIA Ubuntu 26.04 workloads as submodules:

```bash
git clone --recurse-submodules https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2604 /opt/benchmarks
cd /opt/benchmarks
./run_benchmark_suite.sh -w 410 --smoke     # just this workload
./run_benchmark_suite.sh --smoke            # all 32 workloads
./show_metrics.sh 410                       # this workload's runs from the ledger
```

The suite tools (`run_benchmark_suite.sh`, `get_benchmark_results.sh`, `show_ledger_runs.sh`, `show_metrics.sh`) are documented in [`gpu-bench-suite`](https://github.com/garys-gpu-benchmarks/gpu-bench-suite).

Other platforms: [AMD 24.04](https://github.com/garys-gpu-benchmarks/bundle-amd-ubuntu-2404) (101–132), [NVIDIA 24.04](https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2404) (201–232), [AMD 26.04](https://github.com/garys-gpu-benchmarks/bundle-amd-ubuntu-2604) (301–332).

## Troubleshooting

- **SSH dropped during setup:** reconnect and run `sudo bash setup.sh --assume-yes` again. It resumes; do not delete `.venv` or `.cache`.
- **A metric reads `na`:** the run could not measure it on this host. The reason is in the run's `metrics_summary.txt` and `errors_info.txt`; see also [Notes](#notes).

## Repository Layout

```text
.
├── setup.sh                      # installs dependencies and builds the benchmark
├── run_benchmark.sh              # runs one profile, parses, validates
├── show_metrics.sh               # prints this workload's ledger rows
├── SPEC.md                       # engineering specification
├── benchmark_specification.json  # workload definition
├── config/                       # profile values and thresholds
├── src/numa_sweep.cpp            # benchmark source
├── scripts/                      # build, collect, parse, validate
├── docs/                         # PRD and contracts
├── tests/
├── results/                      # run outputs
└── LICENSE
```

## Continuous Integration

- [CI](.github/workflows/ci.yml) runs lint and static checks on every pull request and push to `main`. It does not run the benchmark.
- [GPU Smoke Benchmark](.github/workflows/gpu-smoke.yml) runs a profile on a self-hosted runner labeled `gpu`, `nvidia`, `ubu2604`. It only starts by hand (**Actions → GPU Smoke Benchmark → Run workflow**), so pull requests and forks cannot run it.

Both workflows call shared steps in [`shared-workflows@v1`](https://github.com/garys-gpu-benchmarks/shared-workflows).
