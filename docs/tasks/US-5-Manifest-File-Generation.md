# Task: Manifest File Generation

**Feature:** `US-5 — Chunked values without LFS`

## 1. Description

Implement the logic to generate the `.manifest.json` file. This file acts as a pointer for chunked values, containing all the metadata needed for the `get` command to reconstruct the original file.

## 2. Acceptance Criteria

- A function is created that generates the manifest content as a JSON object.
- The manifest **must** include:
  - `v`: version number
  - `key`: the key name
  - `total_size`: total size of the original file
  - `chunking`: details about the chunking algorithm used
  - `algo`: digest algorithm for chunks (e.g., `sha256`)
  - `etype`: ETag of the full file (e.g., `blake3:...`)
  - `chunks`: an array of chunk objects.
- Each object in the `chunks` array **must** contain:
  - `i`: the chunk index (0, 1, 2, ...)
  - `size`: the size of the chunk in bytes
  - `digest`: the checksum of the chunk's content
  - `blob_oid`: the Git blob OID of the chunk.

## 3. Test Plan

- **Unit Test:** After chunking a sample file, pass the resulting chunk metadata to the manifest generation function. Verify that the output JSON is correctly structured and contains all the required fields with the correct values.
- **Schema Validation:** Test that the generated JSON conforms to a predefined schema.
