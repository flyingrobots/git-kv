# Task: Client Prefix Traversal

**Feature:** `US-3 — Prefix list (index-only)`

## 1. Description

Implement the specific logic for the `list` command to translate a given key prefix into a directory path within the index branch. The index uses the first few characters of the key to create a sharded directory structure (e.g., key `user:123` might be at `index/us/er/user:123.ref`). This task is about converting the prefix to the path to be searched.

## 2. Acceptance Criteria

- A function takes a prefix string (e.g., `user:`) as input.
- It returns the corresponding directory path to search in the index (e.g., `index/us/er/`).
- The logic correctly handles prefixes of varying lengths, including those shorter or longer than the sharding depth.
- For a prefix shorter than the shard depth (e.g., `u`), it returns the top-level path to search (e.g., `index/u/`).

## 3. Test Plan

- **Unit Test:** Test with a variety of prefixes (`user:`, `u`, `user:12345`) and verify the correct index directory path is returned for each.
- **Edge Case:** Test with an empty prefix; it should return the root of the index (`index/`).
- **Edge Case:** Test with prefixes containing special characters that might be valid in keys but not in file paths, and ensure they are handled gracefully.
