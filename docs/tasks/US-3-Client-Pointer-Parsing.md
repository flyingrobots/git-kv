# Task: Client Pointer Parsing

**Feature:** `US-3 — Prefix list (index-only)`

## 1. Description

Implement the logic to read and parse the `.ref` pointer files found in the index. The `list` command must inspect these files to determine if a key is still valid or has been deleted.

## 2. Acceptance Criteria

- A function takes the content of a `.ref` file as input.
- It parses the JSON content of the file.
- It checks the value of the `"type"` field.
- If the type is `"deleted"`, the function indicates that the key should be excluded from the list results.
- The logic is robust against malformed JSON.
- (Future) The logic can be extended to check for TTL expiration.

## 3. Test Plan

- **Unit Test:** Provide the content of a standard `.ref` file and verify it is parsed correctly.
- **Unit Test:** Provide the content of a tombstone (`"type":"deleted"`) `.ref` file and verify the function correctly identifies it as a deleted key.
- **Failure Case:** Provide malformed JSON content and verify the function returns an error instead of crashing.
