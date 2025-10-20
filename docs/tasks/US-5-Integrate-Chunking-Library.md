# Task: Integrate Chunking Library

**Feature:** `US-5 — Chunked values without LFS`

## 1. Description

Research, select, and integrate a Go library that provides Content-Defined Chunking (CDC), specifically using the FastCDC algorithm as specified in the design documents. This library will be the foundation for handling large values.

## 2. Acceptance Criteria

- A suitable Go library for FastCDC is chosen based on criteria: supports configurable min/avg/max, streaming mode, permissive license, and shows recent activity (most recent commit on or after 2024-10-20). This library is added as a dependency to the `go.mod` file.
- A `Chunker` interface is defined: `type Chunker interface { Chunk(r io.Reader) ([]ChunkResult, error) }`.
- Each `ChunkResult` returned by the `Chunker` **must** include `Offset`, `Length`, `Data` (or a reference to it), and `Fingerprint` (hash) fields.
- The `Chunker` is instantiated via a constructor or factory that accepts `minSize`, `avgSize`, and `maxSize` parameters as `uint32` byte counts (defaults: `minSize = 64 * 1024`, `avgSize = 256 * 1024`, `maxSize = 1 * 1024 * 1024`).

## 3. Test Plan

- **Unit Test (Deterministic Chunking):**
  - **Fixture:** `docs/tasks/testdata/chunker_fixtures/large-file-1mb.bin` (1,048,576 bytes of known content).
  - **Chunker Config:** `minSize=64KB`, `avgSize=256KB`, `maxSize=1MB`, specific normalization level (if applicable).
  - **Assert:**
    - The exact count of chunks produced.
    - The exact `Fingerprint` (hash) of each chunk.
    - The chunk boundaries (start/end offsets) are reproducible across multiple runs.
- **Benchmark Test:**
  - **Environment:** Single-threaded run on a 4.0 GHz x86-64 CPU with 16 GB RAM, SSD storage, Go `1.23.x`, no other CPU-intensive workloads.
  - **Fixture:** `docs/tasks/testdata/chunker_fixtures/large-file-1gb.bin` (1 GB of deterministic content loaded from disk).
  - **Chunker Config:** `minSize=64*1024`, `avgSize=256*1024`, `maxSize=1*1024*1024`.
  - **Measurement:** Use `go test -bench=.` (or equivalent benchmark harness) to run 5 iterations, capture throughput (MB/sec) per iteration, compute the median.
  - **Target:** Median throughput ≥ 50 MB/sec. If the median falls below 50 MB/sec, treat as a merge blocker until performance is addressed.
  - **Reporting:** Document the benchmark command, raw output, and median calculation in the PR description (include hardware specs).
