# Task: Stargate Watchlog Generation

**Feature:** `US-7 — Watch Feed`

## 1. Description

Update the Stargate's `post-receive` hook to generate a commit in the `refs/kv-watchlog/<ns>@<epoch>` ref for every accepted ledger change. The watchlog is an append-only Git ref that clients discover via API and consume via polling or streaming endpoints.

## 2. Acceptance Criteria

- The `post-receive` hook logic is triggered after a successful push to a ledger ref.
- The hook creates a new commit on `refs/kv-watchlog/<ns>@<epoch>` with a strictly increasing sequence number stored in the commit message or metadata, and a blob containing structured JSON (`{"seq":<int>,"ledger_oid":"...","timestamp":"<ISO8601>","epoch":"<YYYYQn>","op":"set",...}`).
- Discovery: expose `GET /v1/watchlog/<ns>/refs` (auth required) returning the available epoch watchlog refs and their latest sequences; ensure refs are also advertised via Git `ls-remote`.
- Polling consumption: `GET /v1/watchlog/<ns>?since=<seq>&limit=<n>` returns JSON `{ "entries": [...], "next": <seq|null> }` with entries ordered ascending by sequence and including commit metadata. Supports pagination up to 1000 entries per call. `next` is `null` when no additional entries are currently available; otherwise it is the exclusive sequence clients should pass as the next `since` to continue pagination (especially when the limit is reached).
- Streaming consumption: provide a WebSocket stream (`wss://<host>/v1/watchlog/<ns>/stream`) as the required interface. (An optional gRPC stream may be offered but must mirror the same semantics.) Streams must deliver entries in order with at-least-once delivery, require client acknowledgement tokens (integers equal to the acknowledged `seq`, idempotent on re-send), and support reconnect using the last acknowledged `seq` (clients send `since=<ack>` on reconnect). Recommended client retry policy: exponential backoff starting at 200 ms doubling to 3 s.
- Delivery guarantees: entries are emitted in commit sequence order with at-least-once semantics; clients deduplicate via sequence number across WebSocket (and optional gRPC) implementations.
- Epoch identifiers must follow the format `YYYYQn` (e.g., `2025Q4`). Rollover occurs when a commit timestamp enters a new calendar quarter.
- The Stargate's mirror configuration is updated to push `refs/kv-watchlog/*` to the GitHub remote.

## 3. Test Plan

- **Integration Test (Hook Commit):** Perform a `git kv set` operation. Verify `refs/kv-watchlog/<ns>@<epoch>` advanced by one commit with monotonic sequence.
- **Integration Test (JSON Payload):** Fetch latest watchlog blob and validate JSON schema (`seq`, `ledger_oid`, `timestamp`, `epoch`, `op`).
- **Integration Test (Discovery API):** Call `GET /v1/watchlog/<ns>/refs`; assert returned refs include the updated epoch with latest sequence.
- **Integration Test (Polling API):** Request `GET /v1/watchlog/<ns>?since=<seq>&limit=100`; ensure entries ordered, `next` cursor correct, pagination enforced.
- **Integration Test (Streaming API):** Connect to the WebSocket stream, publish a change, and assert the event is received within a reasonable test timeout and contains the expected sequence (tests should use a bounded wait but no SLA is implied).
- **Integration Test (Mirroring):** Verify the watchlog ref is pushed to the GitHub remote.
- API responses must follow documented schema:
  - `GET /v1/watchlog/<ns>/refs` → `{ "epochs": [{ "epoch":"2025Q4", "ref":"refs/kv-watchlog/<ns>@2025Q4", "latest_seq":1234 }] }`
  - `GET /v1/watchlog/<ns>?since=<seq>&limit=<n>` → `{ "entries": [{ "seq":1234, "ledger_oid":"abc...", "timestamp":"2025-10-26T10:30:00Z", "epoch":"2025Q4", "op":"set" }], "next":1235 }`
  - WebSocket frames emit `{ "seq":1234, "ledger_oid":"...", "timestamp":"...", "epoch":"...", "op":"set" }` and expect client acknowledgements `{ "ack":1234 }`. Optional gRPC streams must use equivalent message fields and acknowledgement semantics.
