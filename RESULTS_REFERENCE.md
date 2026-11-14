# Results: 410 NUMA Cache Latency Sweep

**Main goal:** Measure NUMA local vs remote stride-walk latency  
**Tools:** Build and run numa_sweep via collect_numa_cache.py, with numactl and optional perf stat

Reference results measured by the maintainer on the system shown in each section (generator TEMPLATE_00_140 · recorded 2026-11-13). Your own runs write `results/RESULTS_ACTUAL.md` in the same format.

> Smoke runs use deliberately small sizes. They check that the workload installs and runs end to end; their numbers are not performance results. Baseline and extended are.

<!-- profile:smoke -->
## smoke

- **System:** 1× NVIDIA H100 PCIe (81559 MiB) · Intel(R) Xeon(R) Platinum 8352Y CPU @ 2.20GHz · Ubuntu 26.04.1 LTS · kernel 6.8.0-138-generic · driver 595.91.07
- **Run:** 2026-11-13T20:33:28Z · status **N/A-EXPECTED** (exit 0)
- **Time (mm:ss):** setup 00:47 · benchmark 00:03 · whole run 01:02
- **Source:** run 410_20261113_203328
- **Command:** `bash run_benchmark.sh --smoke --source-numa-node 0 --target-numa-node 0,remote --num-threads 1 --thread-affinity strict --working-set-size 32768,2097152,268435456 --cache-level L1,L3,DRAM --access-pattern random --stride 64 --num-iterations 20000`

| # | Metric | Result |
|---|---|---|
| 1 | Largest cache-tier pointer-chase latency, ns (cache_latency_nsec) | 65.665 |
| 2 | Local DRAM pointer-chase latency, ns (local_numa_node_latency_nsec) | 193.1628 |
| 3 | Pointer-chase payload rate, GB/s (local_dram_pointer_chase_bandwidth_gb_s) | 0.0414 |
| 4 | DRAM-tier cache misses/s (cache_miss_counters_misses_sec) | na (expected: perf_event_paranoid blocks cache-misses) |
<!-- /profile:smoke -->
