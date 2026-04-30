# Fault-Tolerant Distributed Data Processing System (gRPC + C++)

> High-throughput, crash-resilient distributed pipeline that processes millions of CSV rows across a 6-node cluster — built with gRPC, C++17, and a heartbeat-driven health monitor.

## About

This project is a **fault-tolerant distributed data processing system** designed for scalability, resilience, and high throughput. Built in **C++17** and powered by **gRPC streaming**, it organises six nodes into a three-tier hierarchy and can sustain **29–77 MB/s throughput** while processing up to **10 million rows** of CSV data concurrently.

Key capabilities at a glance:

| Feature | Detail |
|---|---|
| **Fault Tolerance** | Heartbeat-based health monitoring; crashed workers are automatically detected and their tasks reassigned |
| **Chunked Streaming** | 67 % memory reduction vs. bulk transfer — data flows in configurable chunks |
| **Multi-Client Support** | Session-ID correlation prevents request collisions from concurrent clients |
| **Configurable Timeouts** | Fine-tune latency/reliability trade-offs via environment variables (`MINI3_LEADER_TIMEOUT_MS`, `MINI3_TEAMLEADER_TIMEOUT_MS`) |
| **Partial Success Delivery** | Returns available results even when some nodes fail |
| **Capacity-Aware Scheduling** | Workers are selected based on live health status and queue length |

### Why this repo?

- Looking for a **gRPC C++ example** with real distributed fault tolerance? This is it.
- Need a reference for **heartbeat-based health monitoring** in a multi-node system? Start here.
- Building a **distributed CSV / data pipeline** and want a battle-tested template? Clone and adapt.

---

This project implements a distributed data processing system using gRPC and C++.
It was built as a course project (CMPE 275 Mini Project 3) and is designed to run on one or more machines.

The main idea:
- A **leader node (A)** receives client requests.
- One or two **team leaders (B, E)** forward work to **workers (C, D, F)**.
- Workers process CSV data in chunks and send results back.

Everything is configured from JSON, and there are helper scripts to build and run.

---

## 1. Prerequisites

- CMake and a C++17 compiler (tested on macOS with `g++`/`clang`).
- `protoc` and gRPC development files (already set up in this repo).
- Python 3 (only for small utility tests and the optional Python gRPC server).

From the project root (`mini_2/`), you should see folders like `src/`, `config/`, `scripts/`, `tests/`, and `test_data/`.

---

## 2. Build the C++ code

From the project root:

```bash
cd scripts
./build.sh
```

This creates a `build/` directory and builds these main targets:
- `build/src/cpp/mini2_server`  – server process for nodes A–F
- `build/src/cpp/mini2_client`  – simple client
- `build/src/cpp/cpp_unit_tests` – small C++ sanity test

If the build succeeds, you are ready to run the system.

---

## 3. Network configuration

The cluster layout is described in `config/network_setup.json`.

- Each node (A–F) has an `id`, `host`, and `port`.
- The overlay edges define who can talk to whom.

For local testing on one machine, you can keep the default localhost config.
For multiple machines, adjust the `host` fields to real IPs/hostnames and copy the repo to each machine.

---

## 4. Starting the servers

All helper scripts live in `scripts/`. The most useful ones:

- `start_servers.sh` – start a group of nodes (A–F) based on a simple profile.
- `start_node.sh`     – start a single node.
- `test_real_data.sh` – run a quick client test against the running cluster.

### 4.1 Start a small cluster on one machine

From the project root:

```bash
./scripts/start_servers.sh --computer 1   # starts A, B, D
./scripts/start_servers.sh --computer 2   # starts C, E, F
```

Each node is launched with `mini2_server --config config/network_setup.json --node <ID>`
and logs are written under `logs/` (e.g., `logs/server_A.log`).

You can also pick nodes explicitly:

```bash
./scripts/start_servers.sh --nodes "A C"
```

To stop all servers:

```bash
pkill -f mini2_server
```

### 4.2 Start a single node manually

If you just want one server:

```bash
./scripts/start_node.sh A
```

This script finds the built `mini2_server` binary and passes the right `--config` and `--node` flags.

---

## 5. Running a simple client test

After servers are up (at least A and one team), you can send a request
that uses one of the sample CSV files in `test_data/`.

Quick way (uses a default dataset):

```bash
./scripts/test_real_data.sh
```

Or specify a particular CSV:

```bash
./scripts/test_real_data.sh --dataset test_data/data_10k.csv
```

Under the hood this runs something like:

```bash
./build/src/cpp/mini2_client --mode request --dataset test_data/data_10k.csv
```

The leader node will print how many rows/bytes were processed.

---

## 6. Basic tests and sanity checks

### 6.1 Python tests

There is a small Python test file that checks the config, scripts, and dataset:

```bash
python3 tests/python_unit_tests.py
```

It verifies:
- `config/network_setup.json` has the expected 6 nodes and overlay.
- The main shell scripts exist, have a shebang, and are executable.
- `test_data/data_10k.csv` is present and looks like a CSV.

### 6.2 C++ unit test

If CTest is enabled in your environment, from the `build/` directory you can try:

```bash
cd build
ctest
```

or directly run the binary:

```bash
./src/cpp/cpp_unit_tests
```

This mainly sanity-checks that the config loader works.

---

## 7. Notes

- The project was structured to follow the course phases (config, forwarding, aggregation, chunked responses).
- Most of the behavior is driven by `config/network_setup.json`, so keep that file in sync across machines.
- For debugging, check `logs/server_*.log` on each node.

This README is intentionally short; for a deeper explanation of the design and phases, see `docs/IMPLEMENTATION_GUIDE.md`.
