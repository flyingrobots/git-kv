# Task: Client Index Reader

**Feature:** `US-3 — Prefix list (index-only)`

## 1. Description

Implement the client-side logic responsible for reading data from the `refs/kv-index/<ns>` branch. This is the core of the `list` command's performance.

## 2. Acceptance Criteria

- Implement an async function `listIndexKeys(repoPath: string, branchOrTreeish: string, prefix?: string): Promise<string[]>`.
- The function must invoke Git plumbing (`git ls-tree -r --name-only <treeish>`) to enumerate blobs without checking out the branch.
- Apply prefix filtering to Git path names before transformation; only paths starting with the provided `prefix` (when supplied) should be considered.
- Convert tree paths into keys by removing the leading `index/` component and normalizing separators to `/`. Example: `index/us/er/name` → `us/er/name`.
- Return keys in lexicographic order. If no entries match, resolve with an empty array.
- Throw a descriptive error if the repository cannot be accessed or the treeish is invalid.
- Document behavior and include at least two examples in the docstring (e.g., call with no prefix, call with `prefix="index/us/"`).
- Add a unit test that seeds a fake Git repo and asserts prefix filtering and key normalization.

## 3. Test Plan

### Correctness (Unit Tests)

Use a deterministic bare repository fixture created via `git init --bare` and populated by writing trees/commits directly. The fixture should contain an `index/` tree with two sharding levels and the following blobs:

- `index/us/er/user:1`
- `index/be/ta/beta:2`
- `index/ba/rq/bar:qux`

Each unit test clones this fixture into a temporary directory, commits the blobs to `refs/kv-index/test-ns`, and runs assertions:

- `listIndexKeys(repoPath, 'refs/kv-index/test-ns')` → `['bar:qux', 'beta:2', 'user:1']` (sorted, transformed, no leading `index/`).
- `listIndexKeys(repoPath, 'refs/kv-index/test-ns', 'index/us/')` → `['user:1']`.
- `listIndexKeys(repoPath, 'refs/kv-index/test-ns', 'index/missing/')` → `[]`.
- Passing a missing branch or bogus treeish rejects with an error mentioning the invalid treeish.
- Passing an invalid prefix containing spaces or `*` throws a validation error before running Git.
- Include entries whose filenames contain control characters (e.g., `index/co/nt/key\nline`); ensure the returned keys escape them as `key\nline`.

### Performance / Benchmark (Integration)

Create a synthetic bare repo with ~1,000,000 entries in `index/` (60% shallow, 40% additional depth). Execute `listIndexKeys(repoPath, 'refs/kv-index/test-ns')` on CI hardware (4 vCPU, 8 GB RAM) and assert:

- Runtime ≤ 1.0 second.
- Peak RSS < 256 MB (capture via `/usr/bin/time -v`).
- Throughput ≥ 800k keys/sec.

Mark this benchmark as non-default (requires explicit opt-in, e.g., `npm run benchmark:listIndexKeys`).
