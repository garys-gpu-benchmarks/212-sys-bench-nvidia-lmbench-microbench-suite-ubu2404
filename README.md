# lmbench Microbench Suite Benchmark

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![CI](https://github.com/garys-gpu-benchmarks/212-sys-bench-nvidia-lmbench-microbench-suite-ubu2404/actions/workflows/ci.yml/badge.svg)](https://github.com/garys-gpu-benchmarks/212-sys-bench-nvidia-lmbench-microbench-suite-ubu2404/actions/workflows/ci.yml)

Target: Ubuntu 24.04 · NVIDIA · see Hardware Requirements. This is a host benchmark, not a laptop `pip install` project.

## Quick Start

```bash
git clone https://github.com/garys-gpu-benchmarks/212-sys-bench-nvidia-lmbench-microbench-suite-ubu2404.git
cd 212-sys-bench-nvidia-lmbench-microbench-suite-ubu2404
sudo bash setup.sh --assume-yes
bash run_benchmark.sh --profile smoke --validate
```
Results are written to `results/benchmark.db` and `results/summary.json`.

This workload is executed on the validation host after the repository is copied there. `setup.sh` and `run_benchmark.sh` do not open an outbound SSH session.

Prerequisites: Ubuntu 24.04; NVIDIA; Python 3.12.3; root or sudo for `setup.sh`. Framework: Bash, SQLite, Python, PyYAML, lmbench. This is a host benchmark, not a laptop `pip install` project.

```mermaid
flowchart LR
  setup.sh --> run_benchmark.sh --> parse_results.py --> results/benchmark.db
```

## Download as a Bundle

This repository is one of the 32 workloads in the NVIDIA Ubuntu 24.04 bundle, [`bundle-nvidia-ubuntu-2404`](https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2404), which holds them as git submodules pinned to a release. To put every workload for this platform on a GPU host in one step, run this as root (or put `sudo` in front of the `apt-get` and `git clone` commands):

```bash
apt-get update && apt-get install -y git
git clone --recurse-submodules https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2404 /opt/benchmarks \
  && cd /opt/benchmarks \
  && ./run_benchmark_suite.sh --list \
  && cd /opt/benchmarks/212-sys-bench-nvidia-lmbench-microbench-suite-ubu2404 \
  && bash run_benchmark.sh --profile smoke --validate \
  && cd /opt/benchmarks \
  && echo "Done: 32 workloads in /opt/benchmarks; 212-sys-bench-nvidia-lmbench-microbench-suite-ubu2404 smoke run passed. Next: ./run_benchmark_suite.sh --help"
```

Every line in `./run_benchmark_suite.sh --list` should show `ready`; it exits with an error if any workload is not fetched. The commands are joined with `&&`, so if the clone, the list or the smoke run fails, the chain stops there, and the closing message appears only when everything succeeded. The chain ends in `/opt/benchmarks`, where the suite runner is. For more extensive testing:

```bash
./run_benchmark_suite.sh --smoke                  # smoke test of all 32 workloads
./run_benchmark_suite.sh -w 207 --baseline        # workload 207, baseline profile
./run_benchmark_suite.sh --help                   # all options
```

A workload that is not set up yet sets itself up on its first run (its `run_benchmark.sh` calls `scripts/ensure_setup.sh`). See [Suite Tools](#suite-tools) for more.

To move to the newest published release later: `cd /opt/benchmarks && git pull && git submodule update --init --recursive`.

| Bundle repository | Platform | Workloads |
|---|---|---|
| [`bundle-amd-ubuntu-2404`](https://github.com/garys-gpu-benchmarks/bundle-amd-ubuntu-2404) | AMD · Ubuntu 24.04 | 101–132 |
| [`bundle-nvidia-ubuntu-2404`](https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2404) | NVIDIA · Ubuntu 24.04 | 201–232 |
| [`bundle-amd-ubuntu-2604`](https://github.com/garys-gpu-benchmarks/bundle-amd-ubuntu-2604) | AMD · Ubuntu 26.04 | 301–332 |
| [`bundle-nvidia-ubuntu-2604`](https://github.com/garys-gpu-benchmarks/bundle-nvidia-ubuntu-2604) | NVIDIA · Ubuntu 26.04 | 401–432 |

## Suite Tools

Three operator scripts from [`garys-gpu-benchmarks/gpu-bench-suite`](https://github.com/garys-gpu-benchmarks/gpu-bench-suite) work across the whole bundle. `run_benchmark_suite.sh` and `show_metrics.sh` are also at the top of every bundle (and the generator's remote copy installs `show_metrics.sh` at `/opt/benchmarks`); each script prints full usage with `--help`.

- **`run_benchmark_suite.sh`** (on the GPU host) — runs the installed workloads one after another for each profile, writes one log to `/opt/benchmarks/benchmark_suite_log/`, and ends with a pass/fail summary.
  ```bash
  cd /opt/benchmarks
  ./run_benchmark_suite.sh --dry-run                    # list what would run
  ./run_benchmark_suite.sh -w 212 --smoke --baseline  # just this workload
  ./run_benchmark_suite.sh --smoke                      # every workload, smoke profile
  ./run_benchmark_suite.sh -r 20                        # soak: everything, 20 times
  ```
- **`get_remote_info.sh`** (on your laptop; Git Bash on Windows) — over one SSH connection, copies the runtime ledger and every workload's `results/raw`, `results/parsed` and suite logs into `RUNTIME_LEDGERS/` and `DIRECTORIES/`.
  ```bash
  git clone https://github.com/garys-gpu-benchmarks/gpu-bench-suite && cd gpu-bench-suite
  ./scripts/get_remote_info.sh --host 203.0.113.10              # root@host, port 22
  ./scripts/get_remote_info.sh --host ubuntu@203.0.113.10 --port 2222 -i ~/.ssh/id_ed25519 -o ~/results
  ```
- **`show_metrics.sh`** (laptop or GPU host) — prints one workload's runs from the runtime ledger in fixed-width metric columns with per-profile means. On the GPU host it is `/opt/benchmarks/show_metrics.sh`.
  ```bash
  cd /opt/benchmarks && ./show_metrics.sh 212             # on the GPU host
  ./scripts/show_metrics.sh 212 --host 203.0.113.10            # over SSH, all profiles
  ./scripts/show_metrics.sh 212 --baseline --file RUNTIME_LEDGERS/<ledger>.csv
  ```

## 1. Overview

Runs a fixed lmbench subset from PATH or /usr/lib/lmbench/bin: lat_syscall (null and read only), lat_ctx, lat_mem_rd 16 128, bw_mem 512m rd, and bw_pipe. repetitions: suite repeats. Yaml lat_pipe, lat_unix, lat_proc, and extra lat_syscall variants are documented but not executed. output_format: csv Sweep dimensions: parallel_jobs, warmup_duration_us, lat_ctx, lat_mem_rd, bw_mem, bw_pipe, lat_pipe, lat_unix.

## 2. What It Validates

- Validates that the five executed binaries return finite values. write/stat/open syscalls and lat_pipe/lat_unix/lat_proc are not collected
- #1: Null syscall latency, us (lat_syscall_null_us); is present and physically sensible.
- #2: Context-switch latency, us (context_switch_latency_us); is present and physically sensible.
- #3: 16 MB stride-128 sequential-read latency, cache- and prefetch-assisted, ns (lat_mem_rd_16mb_stride128_ns); is present and physically sensible.
- #4: Single-thread memory read bandwidth, MB/s (bw_mem_mb_s); is present and physically sensible.
- #5: Inter-process pipe bandwidth, MB/s (bw_pipe_mb_s) is present and physically sensible.

## 3. Metrics Captured

- **#1: Null syscall latency, us** — stored as `lat_syscall_null_us`.
- **#2: Context-switch latency, us** — stored as `context_switch_latency_us`.
- **#3: 16 MB stride-128 sequential-read latency, cache- and prefetch-assisted, ns** — stored as `lat_mem_rd_16mb_stride128_ns`.
- **#4: Single-thread memory read bandwidth, MB/s** — stored as `bw_mem_mb_s`.
- **#5: Inter-process pipe bandwidth, MB/s** — stored as `bw_pipe_mb_s`.

## 4. Hardware Requirements

### Supported environment

- OS: Ubuntu 24.04
- GPU vendor: NVIDIA
- Framework family: Bash, SQLite, Python, PyYAML, lmbench
- Python: Python 3.12.3

### Reference validation environment

The tables below describe the machine used to generate the reference results. They are not a requirement that every user buy that exact cloud instance.

### System

Runs a fixed lmbench subset from PATH or /usr/lib/lmbench/bin: lat_syscall (null and read only), lat_ctx, lat_mem_rd 16 128, bw_mem 512m rd, and bw_pipe. repetitions: suite repeats. Yaml lat_pipe, lat_unix, lat_proc, and extra lat_syscall variants are documented but not executed.

### GPU

Ubuntu 24.04 / NVIDIA / Bash, SQLite, Python, PyYAML, lmbench

## 5. Software Requirements

| Component | Version |
|---|---|
| OS | Ubuntu 24.04 |
| Kernel | kernel 6.8.0 |
| Python | Python 3.12.3 |
| ROCm | CUDA 12.6 (nvcc 12.6.85 preinstalled in the container; setup installs 12.8 only when nvcc is missing). Driver 580.159.04 reports CUDA 13.0 |
| rocBLAS | N/A - cuBLAS not used |

Runs a fixed lmbench subset from PATH or /usr/lib/lmbench/bin: lat_syscall (null and read only), lat_ctx, lat_mem_rd 16 128, bw_mem 512m rd, and bw_pipe. repetitions: suite repeats. Yaml lat_pipe, lat_unix, lat_proc, and extra lat_syscall variants are documented but not executed.

## 6. Installation

```bash
Run lmbench lat_syscall (null, read), lat_ctx, lat_mem_rd 16 128, bw_mem 512m rd, and bw_pipe
```

## 7. Running the Benchmark

```bash
Run lmbench lat_syscall (null, read), lat_ctx, lat_mem_rd 16 128, bw_mem 512m rd, and bw_pipe
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

One wide CSV row with the six lmbench results. A single row is duplicated to two samples

sample_index,status,lat_syscall_null_us,lat_syscall_read_ns,context_switch_latency_us,lat_mem_rd_16mb_stride128_ns,bw_mem_mb_s,bw_pipe_mb_s,error_message
0,ok,0.12,200,2.5,80,24000,8000,

```bash
Run lmbench lat_syscall (null, read), lat_ctx, lat_mem_rd 16 128, bw_mem 512m rd, and bw_pipe
```

### `results/summary.json`

Consolidated metrics from the most recent run — suitable for CI artifact upload or dashboard ingestion.

### `results/raw/<timestamp>.txt`

One wide CSV row with the six lmbench results. A single row is duplicated to two samples

sample_index,status,lat_syscall_null_us,lat_syscall_read_ns,context_switch_latency_us,lat_mem_rd_16mb_stride128_ns,bw_mem_mb_s,bw_pipe_mb_s,error_message
0,ok,0.12,200,2.5,80,24000,8000,

## 9. Baselines / Thresholds

Expected ranges and gates live in `config/benchmark_config.yaml` under `baselines:` or `thresholds:`. To update them, edit that file — never edit validation code directly.

## 10. Troubleshooting

**`setup.sh` missing collector**
Create cannot finish without `scripts/collect_workload.py`.

**`self_check` overlay rewritten**
`Locked overlay rewritten` means the sibling file and the `sha256` in `results/overlay_lock.json` disagree. The repair is to make `implementation_components/<component-id>/files/<path>`, the sibling path, and that digest the same bytes, then recheck with `create_one_workload.py --validate-only`. Do not replace the overlay with a new collector, and do not regenerate a non-empty sibling.

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
| [GPU Smoke Benchmark](.github/workflows/gpu-smoke.yml) | self-hosted runner labeled `gpu`, `nvidia`, `ubu2404` | only when started by hand: **Actions → GPU Smoke Benchmark → Run workflow** (choose `smoke`, `baseline` or `extended`) | Verifies the pre-provisioned GPU stack, records `results/environment.json` (driver, runtime, kernel, GPU), runs the profile with `--validate`, shows headline metrics on the run page, and uploads the results. |

Both files are short callers. The steps themselves live once, for every workload in the suite, in [`garys-gpu-benchmarks/shared-workflows`](https://github.com/garys-gpu-benchmarks/shared-workflows), pinned at `@v1`. The GPU workflow is never triggered by pull requests, so code from a fork cannot run on the GPU host.

The GPU Smoke Benchmark workflow runs one workload. To install and run the whole NVIDIA Ubuntu 24.04 bundle by hand on the same machines, see [Download as a Bundle](#download-as-a-bundle) and [Suite Tools](#suite-tools).
