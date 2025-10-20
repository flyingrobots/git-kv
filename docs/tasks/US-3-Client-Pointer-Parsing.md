# Task: Client Pointer Parsing

**Feature:** `US-3 — Prefix list (index-only)`

## 1. Description

Implement the logic to read and parse the `.ref` pointer files found in the index. The `list` command must inspect these files to determine if a key is still valid or has been deleted.

## 2. Acceptance Criteria

- A function takes the raw string content of a `.ref` file and returns a `PointerStatus` enum (e.g., `Active`, `Deleted`, `Expired`) or an error.
- The input must be valid JSON whose root is an object. Only the optional `"type"` field is inspected; additional fields are ignored.
- If the JSON root is not an object (e.g., `null`, array, string, number), return a validation error indicating the pointer file structure is invalid.
- If the JSON lacks a `"type"` field, it is treated as `Active`.
- Allowed `"type"` values are: `"deleted"`, `"active"`, `"expired"`. The value may include surrounding whitespace; trim before comparison.
- If the trimmed `"type"` string is `"deleted"`, the function returns `Deleted`.
- If the trimmed `"type"` string is `"expired"`, the function returns `Expired`.
- If the trimmed `"type"` string is `"active"`, the function returns `Active`.
- If the `"type"` field is missing or explicitly `null`, treat it as `Active`.
- If the `"type"` field exists but is not a string (e.g., number, boolean, object, array), return an error indicating the type is malformed.
- Any other trimmed string value defaults to `Active`.
- The function returns an error object for malformed JSON.

## 3. Test Plan

- **Unit Test (Type Handling):** Test with `.ref` files containing:
  - Missing `"type"` field.
  - `"type"` explicitly `null`.
  - Unrecognized `"type"` value (e.g., `"unknown"`).
  - Explicit `"deleted"` type.
  - Explicit `"active"` type.
  - Explicit `"expired"` type.
  - Case variants (e.g., `"DELETED"`, `"Expired"`).
  - Values with surrounding whitespace (e.g., `" deleted "`).
  - Malformed JSON.
  - Non-string `"type"` values (numbers, booleans, arrays, objects) that should return an error.
  - Empty JSON object `{}`.
  - Verify the function returns the correct `PointerStatus` or error for each case.
- **Unit Test (Invalid Root Structure):** Provide inputs whose JSON roots are `null`, `[]`, `"string"`, and `123`. Assert each returns the structural validation error and never defaults to `Active`.
