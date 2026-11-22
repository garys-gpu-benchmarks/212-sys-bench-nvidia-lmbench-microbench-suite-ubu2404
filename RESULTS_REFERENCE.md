# Results: 212 lmbench Microbench Suite

**Main goal:** Measure a fixed lmbench microbench subset  
**Tools:** Run lmbench lat_syscall (null, read), lat_ctx, lat_mem_rd 16 128, bw_mem 512m rd, and bw_pipe

Reference results from the validation run on the system shown in each section (recorded 2026-11-22). Your own runs write `results/RESULTS_ACTUAL.md` in the same format.

> Smoke runs use deliberately small sizes. They check that the workload installs and runs end to end; their numbers are not performance results. Baseline and extended are.

<!-- profile:smoke -->
## smoke

- **System:** 1× NVIDIA H100 80GB HBM3 (81559 MiB) · Intel(R) Xeon(R) Platinum 8470 · Ubuntu 24.04.5 LTS · kernel 6.8.0-149-generic · driver 595.112.07
- **Run:** 2026-11-22T10:53:49Z · status **PASS** (exit 0)
- **Time (mm:ss):** setup 00:29 · benchmark 00:44 · whole run 01:18
- **Source:** run 212_20261122_105349
- **Command:** `bash run_benchmark.sh --smoke --warmup-duration-us 1000 --repetitions 1`

| # | Metric | Result |
|---|---|---|
| 1 | Null syscall latency, us (lat_syscall_null_us) | 0.1242 |
| 2 | Context-switch latency, us (context_switch_latency_us) | 2.5 |
| 3 | 16 MB stride-128 sequential-read latency, cache- and prefetch-assisted, ns (lat_mem_rd_16mb_stride128_ns) | 6.84 |
| 4 | Single-thread memory read bandwidth, MB/s (bw_mem_mb_s) | 13304.69 |
| 5 | Inter-process pipe bandwidth, MB/s (bw_pipe_mb_s) | 3400.96 |
<!-- /profile:smoke -->
