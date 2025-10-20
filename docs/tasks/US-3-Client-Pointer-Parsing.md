# Task: Client Pointer Parsing

**Feature:** `US-3 — Prefix list (index-only)`

## 1. Description

Implement the logic to read and parse the `.ref` pointer files found in the index. The `list` command must inspect these files to determine if a key is still valid or has been deleted.

## 2. Acceptance Criteria

- A function takes the content of a `.ref` file as input and returns a `PointerStatus` enum (e.g., `Active`, `Deleted`, `Expired`) or an error.
- It parses the JSON content of the file.
- If the JSON lacks a `"type"` field, it is treated as `Active`.
- Allowed `"type"` values are: `"deleted"`, `"active"`, `"expired"`. Any other unrecognized value defaults to `Active`.
- If the type is `"deleted"`, the function returns `Deleted`.
- If the type is `"expired"`, the function returns `Expired`.
- Otherwise (including `"active"` or unrecognized), the function returns `Active`.
- The function returns an error object for malformed JSON.

## 3. Test Plan

- **Unit Test (Type Handling):** Test with `.ref` files containing:
  - Missing `"type"` field.
  - Unrecognized `"type"` value (e.g., `"unknown"`).
  - Explicit `"deleted"` type.
  - Explicit `"active"` type.
  - Explicit `"expired"` type.
  - Malformed JSON.
  - Verify the function returns the correct `PointerStatus` or error for each case.
