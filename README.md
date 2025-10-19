# git-kv (Project Stargate)

**State as code. Commits are transactions. Git is the database. GitHub is the showroom.**

> [!warning]
> `git-kv` is just getting started! The [SPEC](./docs/SPEC.md) just dropped, and we are about to break ground on the implementation. 
> 
> This `README` is basesd on the spec, and is mostly here to generate hype and interest.   
> None of this is true (yet), but watch the repo to follow along.   
> Expect things to change as we build.
> 
> **COMING SOON:** GitHub Issues to track outstanding tasks, organized into **Milestones** according to the roadmap, which you can find in the spec doc.

`git-kv` is an auditable, versioned Key-Value store that uses Git *as* the database. It provides ledger-grade integrity, atomic multi-key transactions, and fast prefix-listing, all while remaining compatible with SaaS Git hosts like GitHub and GitLab.

It achieves this with a transparent "Stargate" gateway:

* **Reads (`git fetch`)** pull from a GitHub mirror for CDN-class speed.
* **Writes (`git push`)** are routed to a tiny, self-hosted Stargate gateway that enforces all integrity guarantees before mirroring back to GitHub.

---

## Why git-kv?

`git-kv` is designed for workflows where **audit beats microseconds**. It's a durable ledger for config, policies, and state, **not a Redis replacement** for 100k QPS.

* **Ledger-Grade History:** Every change is a signed, immutable, linear-history Git commit. No merge commits, no force-pushes.
* **Atomic Multi-Key Transactions:** Set 10,000 keys and delete 5,000 in a single, atomic transaction (`mset`). This is validated on the server in **O(1)** time via client attestation.
* **Fast Prefix Listing:** A dedicated `kv-index` ref provides fast `git kv list --prefix user:` operations without scanning all N keys.
* **Native Large File Support:** Automatically chunks values \>1MB using Content-Defined Chunking (FastCDC). This provides high-performance deduplication *inside* Git, with no Git-LFS required.
* **Bounded Clone Size:** **Epochs** (`git kv epoch new`) snapshot the database, keeping new clones small and fast.
* **SaaS-Native Ergonomics:** Developers work against `origin` (GitHub/GitLab) as usual. The Stargate is a transparent proxy on the `push` path, giving you server-side hooks and guarantees without sacrificing your existing UI/CI/Actions workflow.

---

## Core Concepts: The "Stargate" Architecture

`git-kv` works by separating the read and write paths.

```bash
dev clone
  git fetch origin  ─────────→  GitHub (CDN, UI, Read Path)
  git push origin   ─────────→  Stargate (LAN, Authoritative Write Path)
                                   │ pre-receive (O(1) attestation, policy)
                                   └─ post-receive mirror → GitHub
```

This transparency is achieved with a simple Git config rule, set by `git kv remote setup`:

```ini
[remote "origin"]
  url     = git@github.com:your-org/repo.git         # Reads
  pushurl = ssh://git@stargate.local/org/repo.git    # Writes
```

The **Stargate** is a minimal, self-hosted daemon that runs your bare repo and enforces all guarantees. It stores data across three core ref types, which are pushed atomically:

1.  `refs/kv/<ns>` (**Ledger**): The authoritative, hash-sharded data (`data/`) and metadata (`meta/`).
2.  `refs/kv-index/<ns>` (**Index**): Prefix-sharded pointer files (`.ref`) for fast listing.
3.  `refs/kv-chunks/<ns>@<epoch>` (**Chunks**): Content-addressed blobs for large values.

---

## Quick Start

### 1\. Admin: Set up Stargate

(One-time setup)

1.  Deploy the `git-kv-stargate` binary to a server (`stargate.local`).
2.  Initialize it as a bare repo with hooks and policy:
    ```bash
    # On the Stargate server
    git-kv-stargate init --repo /srv/git/kv.git --mirror-remote git@github.com:your-org/kv.git
    systemctl start git-kv-stargate
    ```
3.  On GitHub, configure the repo's `refs/kv/*` branches as "protected" to block all direct pushes.

### 2\. Developer: Clone & Bootstrap

(Each developer, one time)

1.  Install the `git-kv` client CLI.
    ```bash
    # (Installation instructions TBD: e.g., go install ... or download from releases)
    ```
2.  Clone the repo from GitHub as usual:
    ```bash
    git clone git@github.com:your-org/kv.git
    cd kv
    ```
3.  Run the `remote setup` command. This reads `.kv/policy.yaml` from the repo and configures your `pushurl` automatically.
    ```bash
    git kv remote setup
    # > Configured origin.pushurl → ssh://git@stargate.local/org/kv.git
    ```

### 3\. Developer: Read & Write

```bash
# Set a simple value (this creates one atomic commit)
git kv set cfg:feature:new-ui "true"

# Get the value
git kv get cfg:feature:new-ui
# > "true"

# Perform an atomic multi-key transaction
git kv mset --file changes.jsonl

# changes.jsonl
# {"op":"set","key":"cfg:feature:new-ui","ctype":"application/json","value_b64":"eyJhY3RpdmUiOnRydWV9"}
# {"op":"set","key":"user:42:avatar","ctype":"image/png","file":"avatar.png"}
# {"op":"del","key":"secret:old-token"}

# List all keys under a prefix (fast!)
git kv list --prefix cfg:
# > cfg:feature:new-ui

# In CI, wait for a change to be visible on the GitHub mirror
git kv wait --oid <commit-hash> --visible-on=github
```

---

## Core Commands

### Developer CLI

* `git kv remote setup`: **(First command)** Configures `origin.pushurl` to point to Stargate.
* `git kv get <key>`: Reads a value.
* `git kv set <key> <value>`: Sets a value (string, file, or stdin).
* `git kv del <key>`: Deletes a value (writes a tombstone).
* `git kv mset --file <batch.jsonl>`: Executes a list of `set`/`del` ops atomically.
* `git kv list [--prefix P]`: Lists keys (fast, uses the index).
* `git kv history <key>`: Shows the commit log for a single key.
* `git kv wait --oid <hash>`: **(For CI)** Pauses a script until a commit is visible on the GitHub mirror.

### Admin (Stargate) CLI

* `git kv stargate status`: Checks the health and mirror backlog of the Stargate.
* `git kv stargate sync [--repair]`: Audits and repairs a diverged GitHub mirror.
* `git kv epoch new --label <name>`: Creates a new, bounded snapshot for fast clones.

-----

## Advanced Features

* **Large Value Chunking**: Automatic for files \> `max_value_inline` (e.g., 1MB). Uses FastCDC to minimize delta size.
* **Read-Side TTL**: Set a `ttl` in the metadata; `git kv get` will return "not found" (exit 7) if it's expired.
* **Policy Enforcement**: The `.kv/policy.yaml` file (enforced by Stargate) controls:
  * `stargate.push_url`
  * Allowed signer keys (SSH/GPG)
  * Allowed key prefixes (`user:*`, `cfg:*`)
  * Max value sizes

---

## Stargate Operations (For SREs)

The Stargate gateway is a critical service that must be run with **High Availability** for production use.

* **HA Model**: **Active-Passive** is the recommended deployment. A floating IP (`stargate.local`) points to the leader, which holds a leader lock. The standby continuously mirrors the primary.
* **Health Checks**:
  * `/healthz`: Is the repo readable and are hooks loaded?
  * `/readyz`: Is this node the leader and is the mirror remote reachable?
* **Observability**: Stargate exposes a `/metrics` endpoint for Prometheus with key SLOs:
  * `stargate_txn_latency_ms` (p95 \< 150ms)
  * `stargate_mirror_lag_seconds` (p95 \< 2s)
  * `stargate_mirror_backlog_size`
* **Recovery**: If the GitHub mirror *ever* diverges from the Stargate (Source of Truth), an admin can force a repair:
    ```bash
    git kv stargate sync --repair --ns main
    ```

---

## License

This project is licensed under the **[MIND-UCAL License](https://github.com/universalcharter/mind-ucal)**. See `LICENSE` file for details.

## Contributing

Pull Requests are welcome for issues marked with the `Help Wanted` label\! 

*Please* run `make test` and `make lint` before submitting. 

**For major architectural changes**, please use Discussions to submit a RFC.
