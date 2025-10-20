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

- **Unit Test:** Create a test that passes a sample data file to the chunking service and verifies that it is broken into multiple chunks.
- **Unit Test:** Verify that for a known input, the number and size of the generated chunks are as expected.
- **Benchmark:** Create a benchmark test to measure the throughput of the chunking library on a large file.
