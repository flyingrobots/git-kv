# Task: Integrate Chunking Library

**Feature:** `US-5 — Chunked values without LFS`

## 1. Description

Research, select, and integrate a Go library that provides Content-Defined Chunking (CDC), specifically using the FastCDC algorithm as specified in the design documents. This library will be the foundation for handling large values.

## 2. Acceptance Criteria

- A suitable Go library for FastCDC is chosen and added as a dependency to the `go.mod` file.
- A wrapper or service is created within the `git-kv` codebase that exposes a simple interface for chunking a file or a byte stream.
- The interface should return a list of chunk byte arrays.
- The chunking parameters (min, avg, max size) should be configurable, defaulting to the values in the spec (64KB, 256KB, 1MB).

## 3. Test Plan

- **Unit Test:** Create a test that passes a sample data file to the chunking service and verifies that it is broken into multiple chunks.
- **Unit Test:** Verify that for a known input, the number and size of the generated chunks are as expected.
- **Benchmark:** Create a benchmark test to measure the throughput of the chunking library on a large file.
