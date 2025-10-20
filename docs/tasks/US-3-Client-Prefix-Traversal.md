# Task: Client Prefix Traversal

**Feature:** `US-3 — Prefix list (index-only)`

## 1. Description

Implement the specific logic for the `list` command to translate a given key prefix into a directory path within the index branch. The index uses two sharding levels, each consuming 2 characters (total of the first 4 characters) to build nested directories (e.g., `index/XX/YY/`). This task is about converting the prefix to the path to be searched.

## 2. Acceptance Criteria

- A function takes a prefix string (e.g., `user:`) as input.
- It returns the corresponding directory path to search in the index (e.g., `index/us/er/`).
- The logic correctly handles prefixes of varying lengths, translating them into the appropriate index directory path based on the two-level, two-character-per-level sharding scheme.
  - **Example:**
    - Key `user:123` maps to `index/us/er/user:123.ref`.
    - Prefix `user:` maps to `index/us/er/`.
    - Prefix `u` maps to `index/u/` (first directory level only).
    - An empty prefix maps to `index/`.

## 3. Test Plan

- **Unit Test:** Test multiple prefixes: `user:`, `u`, `user:12345`, `cfg:feature:`, `a`, `ab`, `abc`. Verify the correct index directory path is returned for each based on the 2-character sharding.
- **Edge Case (Empty Prefix):** Test with an empty prefix (`""`). Assert it returns `index/`.
- **Edge Case (Special Characters):**
  - Keys are URL-encoded for filesystem-safe directory names.
  - Test with prefixes containing special characters (e.g., `key/with:slash`, `key?with*star`). Assert the correct URL-encoded path is returned.
  - Test with prefixes containing characters disallowed even after URL-encoding (e.g., null bytes). Assert an explicit error is returned.
