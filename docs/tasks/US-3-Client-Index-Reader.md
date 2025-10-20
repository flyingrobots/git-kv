# Task: Client Index Reader

**Feature:** `US-3 — Prefix list (index-only)`

## 1. Description

Implement the client-side logic responsible for reading data from the `refs/kv-index/<ns>` branch. This is the core of the `list` command's performance.

## 2. Acceptance Criteria

- A function exists that can read the contents of the index branch without performing a full checkout (e.g., using `git ls-tree`).
- The function can list all entries in the index or list entries under a specific path corresponding to a prefix.
- The function returns a list of key names derived from the file paths in the index.

## 3. Test Plan

- **Unit Test:** Using a mock Git repository, test that the function can correctly list all files in a sample index tree.
- **Unit Test:** Test that the function correctly lists only the files under a specific directory path (e.g., `index/us/er/`).
- **Performance:** Create a test with an index containing 1 million entries and verify the function completes within an acceptable time limit.
