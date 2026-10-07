# SPEC.md: "The Human How"; Exact technical requirements, environment setup, implementation details, etc.

## Execution Description

Runs a fixed lmbench subset from PATH or /usr/lib/lmbench/bin: lat_syscall (null and read only), lat_ctx, lat_mem_rd 16 128, bw_mem 512m rd, and bw_pipe. repetitions: suite repeats. Yaml lat_pipe, lat_unix, lat_proc, and extra lat_syscall variants are documented but not executed. output_format: csv Sweep dimensions: parallel_jobs, warmup_duration_us, lat_ctx, lat_mem_rd, bw_mem, bw_pipe, lat_pipe, lat_unix.

## Parameters

| Parameter | CLI Flag | Tested Values | Default | Description |
| --- | --- | --- | --- | --- |
| parallel_jobs | `--parallel-jobs` | smoke=1, baseline=2, extended=2 | 2 | From Parameter list; see Execution Description With Parameters. |
| warmup_duration_us | `--warmup-duration-us` | smoke=1000, baseline=1000000, extended=1000000 | 1000000 | From Parameter list; see Execution Description With Parameters. |
| lat_ctx | `--lat-ctx` | smoke=1, baseline=1, extended=1 | 1 | From Parameter list; see Execution Description With Parameters. |
| lat_mem_rd | `--lat-mem-rd` | smoke=1, baseline=1, extended=1 | 1 | From Parameter list; see Execution Description With Parameters. |
| bw_mem | `--bw-mem` | smoke=1, baseline=1, extended=1 | 1 | From Parameter list; see Execution Description With Parameters. |
| bw_pipe | `--bw-pipe` | smoke=1, baseline=1, extended=1 | 1 | From Parameter list; see Execution Description With Parameters. |
| lat_pipe | `--lat-pipe` | smoke=false, baseline=false, extended=false | false | From Parameter list; see Execution Description With Parameters. |
| lat_unix | `--lat-unix` | smoke=false, baseline=false, extended=false | false | From Parameter list; see Execution Description With Parameters. |
| lat_syscall | `--lat-syscall` | smoke=1, baseline=1, extended=1 | 1 | From Parameter list; see Execution Description With Parameters. |
| lat_proc | `--lat-proc` | smoke=false, baseline=false, extended=false | false | From Parameter list; see Execution Description With Parameters. |
| repetitions | `--repetitions` | smoke=1, baseline=1, extended=4 | 1 | From Parameter list; see Execution Description With Parameters. |
| output_format | `--output-format` | smoke=csv, baseline=csv, extended=csv | csv | From Parameter list; see Execution Description With Parameters. |

## Invocation

```bash
Run lmbench lat_syscall (null, read), lat_ctx, lat_mem_rd 16 128, bw_mem 512m rd, and bw_pipe
```

## Raw Output Format

One wide CSV row with the six lmbench results. A single row is duplicated to two samples

sample_index,status,lat_syscall_null_us,lat_syscall_read_ns,context_switch_latency_us,lat_mem_rd_16mb_stride128_ns,bw_mem_mb_s,bw_pipe_mb_s,error_message
0,ok,0.12,200,2.5,80,24000,8000,

## Metrics

## Framework

Runs a fixed lmbench subset from PATH or /usr/lib/lmbench/bin: lat_syscall (null and read only), lat_ctx, lat_mem_rd 16 128, bw_mem 512m rd, and bw_pipe. repetitions: suite repeats. Yaml lat_pipe, lat_unix, lat_proc, and extra lat_syscall variants are documented but not executed.

## Installation and Execution Summary

Run lmbench lat_syscall (null and read), lat_ctx, lat_mem_rd 16 128, bw_mem 512m rd, and bw_pipe once per yaml repetitions, then parse the first float, to measure syscall, context-switch, and memory-read latency plus memory and pipe bandwidth. lat_pipe, lat_unix, and lat_proc are not executed

## Platform Portability

- **AMD (primary):** ```bash
Run lmbench lat_syscall (null, read), lat_ctx, lat_mem_rd 16 128, bw_mem 512m rd, and bw_pipe
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

One wide CSV row with the six lmbench results. A single row is duplicated to two samples

sample_index,status,lat_syscall_null_us,lat_syscall_read_ns,context_switch_latency_us,lat_mem_rd_16mb_stride128_ns,bw_mem_mb_s,bw_pipe_mb_s,error_message
0,ok,0.12,200,2.5,80,24000,8000,

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
5. All required aggregate metrics are physically sensible (positive values). Runs a fixed lmbench subset from PATH or /usr/lib/lmbench/bin: lat_syscall (null and read only), lat_ctx, lat_mem_rd 16 128, bw_mem 512m rd, and bw_pipe. repetitions: suite repeats. Yaml lat_pipe, lat_unix, lat_proc, and extra lat_syscall variants are documented but not executed.
6. At least 2 sample rows exist for the latest `run_id` (sweep coverage).
7. No sample has `status = 'error'`.
8. Runs a fixed lmbench subset from PATH or /usr/lib/lmbench/bin: lat_syscall (null and read only), lat_ctx, lat_mem_rd 16 128, bw_mem 512m rd, and bw_pipe. repetitions: suite repeats. Yaml lat_pipe, lat_unix, lat_proc, and extra lat_syscall variants are documented but not executed.

### Baseline / Threshold configuration (`config/benchmark_config.yaml`)

Expected ranges and gates live in `config/benchmark_config.yaml` under `baselines:` or `thresholds:`. To update them, edit that file — never edit validation code directly.

Threshold key suffixes encode comparison direction when `thresholds:` is present: `_min` → observed value must be ≥ threshold. `_max` → observed value must be ≤ threshold. Informational `baselines:` ranges are not pass/fail gates.
