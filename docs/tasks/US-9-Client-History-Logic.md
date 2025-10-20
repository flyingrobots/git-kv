# Task: Client History Logic

**Feature:** `US-9 — History across epochs`

## 1. Description

Implement the core client-side logic for retrieving the commit history of a specific key. This involves using Git commands to filter the commit log for changes affecting the key's path within the ledger branch.

## 2. Acceptance Criteria

- A function exists that takes a key and a Git branch reference as input.
- It constructs the correct file path for the key within the ledger tree (e.g., `data/<hh>/<hh>/<urlsafe-key>`).
- It executes a Git command (e.g., `git log -- <path>`) to get the history of that specific path on the given branch.
- It returns a list of `CommitInfo` objects. Each `CommitInfo` object **must** contain:
  - `OID`: string (full commit SHA)
  - `Timestamp`: int64 (Unix epoch seconds)
  - `Message`: string (first line of commit message)
  - `AuthorName`: string
  - `AuthorEmail`: string
  - `EpochID`: string (from `Epoch` trailer)
  - `OperationType`: string (e.g., `set`, `del`, from `Op` trailer, if available)

## 3. Test Plan

- **Unit Test:** Test the path construction logic for various keys.
- **Integration Test:** Using a mock Git repository, create a few commits affecting a specific key. Call the history function and verify it returns the correct commit OIDs.
- **Edge Case:** Test with a key that has no history on the given branch; the function should return an empty list.
