# Task: Client Prefix Traversal

**Feature:** `US-3 — Prefix list (index-only)`

## 1. Description

Implement the specific logic for the `list` command to translate a given key prefix into a directory path within the index branch. The index uses a fixed 2-character sharding depth, splitting keys by their first 2 characters into nested directories (e.g., `index/XX/YY/`). This task is about converting the prefix to the path to be searched.

## 2. Acceptance Criteria

- A function takes a prefix string (e.g., `user:`) as input.
- It returns the corresponding directory path to search in the index (e.g., `index/us/er/`).
- The logic correctly handles prefixes of varying lengths, translating them into the appropriate index directory path based on the 2-character sharding depth.
  - **Example:**
    - Key `user:123` maps to `index/us/er/user:123.ref`.
    - Prefix `user:` maps to `index/us/er/`.
    - Prefix `u` maps to `index/u/`.
    - An empty prefix maps to `index/`.

## 3. Test Plan

- **Unit Test:** Test with a variety of prefixes (`user:`, `u`, `user:12345`) and verify the correct index directory path is returned for each.
- **Edge Case:** Test with an empty prefix; it should return the root of the index (`index/`).
- **Edge Case:** Test with prefixes containing special characters that might be valid in keys but not in file paths, and ensure they are handled gracefully.
