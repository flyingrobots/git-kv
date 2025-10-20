# Task: Integrate Chunking Library

**Feature:** `US-5 — Chunked values without LFS`

## 1. Description

Research, select, and integrate a Go library that provides Content-Defined Chunking (CDC), specifically using the FastCDC algorithm as specified in the design documents. This library will be the foundation for handling large values.

## 2. Acceptance Criteria

- A suitable Go library for FastCDC is chosen based on criteria: supports configurable min/avg/max, streaming mode, actively maintained (within 12 months), and permissive license. This library is added as a dependency to the `go.mod` file.
- A `Chunker` interface is defined: `type Chunker interface { Chunk(r io.Reader) ([]ChunkResult, error) }`.
- Each `ChunkResult` returned by the `Chunker` **must** include `Offset`, `Length`, `Data` (or a reference to it), and `Fingerprint` (hash) fields.
- The `Chunker` is instantiated via a constructor or factory that accepts `minSize`, `avgSize`, and `maxSize` parameters, with defaults of 64KB, 256KB, and 1MB respectively.

## 3. Test Plan

- **Unit Test (Deterministic Chunking):**
  - **Fixture:** `docs/tasks/testdata/chunker_fixtures/large-file-1mb.bin` (1,048,576 bytes of known content).
  - **Chunker Config:** `minSize=64KB`, `avgSize=256KB`, `maxSize=1MB`, specific normalization level (if applicable).
  - **Assert:**
    - The exact count of chunks produced.
    - The exact `Fingerprint` (hash) of each chunk.
    - The chunk boundaries (start/end offsets) are reproducible across multiple runs.
- **Benchmark Test:**
  - **Fixture:** `docs/tasks/testdata/chunker_fixtures/large-file-100mb.bin` (100MB of known content).
  - **Chunker Config:** Same as above.
  - **Metric:** Record throughput in MB/sec.
  - **Target:** Establish a measurable baseline/target (e.g., >50 MB/sec) for actionable results.
