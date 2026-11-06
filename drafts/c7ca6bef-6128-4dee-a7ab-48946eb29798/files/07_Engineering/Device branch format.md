---
type: engineering-reference
owner: "[[Kai]]"
maintained_by: "[[Kai]]"
derived_from:
  - "src/model.rs"
  - "src/codec.rs"
  - "src/transport.rs"
  - "src/engine.rs"
  - "[[ADR-0008 — Store content as blobs referenced by metadata]]"
verified: 2026-11-06
source_revision: 296a09d
refresh: Review against the implementation whenever the transport format or session lifecycle changes; recheck before relying on this reference for a newer release.
---

# Device branch format

This describes the implemented version-two device snapshot format in the 0.2.5 engine, checked on 2026-11-06. It is a reference for inspecting the transport, not instructions for manually editing Mesh's private Git store.

## One branch per device identity

The exact branch name is **`mesh/devices/<device-id>`** (plural `devices`). It is not one branch per user or email address. Workspace initialization generates a device UUID and saves it in `.mesh/config.json`. Two independently initialized folders belonging to the same person have different device IDs and branches. The person's name and email are recorded separately as the session author.

The device branch carries recoverable drafts and synchronization metadata. The shared branch, normally `main` (configurable), carries permanent finalized session commits. Other clients fetch device snapshots and integrate drafts before permanent publication, so an open draft can already be visible in their working folders while absent from `main`.

Each changed snapshot becomes an ordinary commit whose parent is the preceding device snapshot. An identical snapshot reuses the existing commit; a failed upload can be retried. These are full Git trees with deduplicated blobs, not a directory of patches. There is no force-pushed amendment of shared history.

## Tree layout

```text
mesh/devices/<device-id>                 # branch, not a folder
├── state.json                           # metadata index
├── drafts/
│   └── <session-id>/                    # session UUID
│       ├── base/<relative-path>         # original bytes, if the file existed
│       └── files/<relative-path>        # session's resulting bytes, if present
├── conflicts/
│   └── <conflict-id>/
│       ├── base                         # common input, if present
│       ├── accepted                     # accepted input, if present
│       └── incoming                     # competing input, if present
└── resolutions/
    └── <conflict-id>/
        └── selected                     # chosen bytes, if present
```

Git stores files, not empty directories. Every illustrated folder is conditional on at least one file below it. Paths beneath `base/` and `files/` preserve the workspace-relative hierarchy, for example `files/notes/design.md`. Conflict and resolution filenames are literal names; their original workspace path is in the metadata.

## What `state.json` does

`state.json` is the device **manifest**: it relates sessions, authors, timestamps, changes, dependencies, intents, conflicts, and selections to the exact file versions stored in the tree. Clients decode it when fetching device branches. File trees alone cannot express deletion, authorship, session boundaries, or conflict decisions.

This is different from the local engine state in `.mesh/state.sqlite` and from CLI `status` JSON. Local fields such as `observed`, `pending_capture`, `uploaded`, `offline`, and recovery journals are not properties of this remote index. In particular, there is no top-level workspace `status` or per-session `stage` in the manifest.

### JSON conventions and content references

- Maps are JSON objects keyed by IDs or workspace-relative paths; sets serialize as arrays of unique strings. Empty maps and arrays are valid.
- Timestamps (`created_at`, `last_edit`) are unsigned integer Unix seconds, not formatted dates. Counters are unsigned integers. Boolean values are `true` or `false`.
- An identity is `{ "name": "Kai", "email": "kai@example.com" }`. It is commit attribution, not authenticated account identity.
- A file version is either `{ "blob": "<Git blob object ID>" }` or `null`. The object ID identifies native Git content, not a filename or an embedded byte array.
- `null` means the file does not exist in that version. An empty file has a non-null reference to an actual zero-byte blob.
- Every referenced blob must also be a file entry in the same snapshot's tree. Decoding rejects references outside that tree or to the metadata blob itself. This keeps fetch and Git garbage collection from losing required versions.

### Top-level properties

| Property | Values / structure | Purpose |
|---|---|---|
| `version` | Integer `2` for newly written snapshots. The reader also accepts legacy `1`; other versions are rejected. | Selects the decoding format. Version 1 embedded bytes in JSON; version 2 uses blob references and native files. |
| `sessions` | Object: session UUID → session record. | All retained sessions originating on this device, including open, closed, published, and discarded sessions. Sessions fetched from other devices are not republished in this map. |
| `conflicts` | Object: conflict ID → conflict record. | Preserves conflict evidence known to this device, including received records. |
| `resolutions` | Object: conflict ID → resolution record. | Shares known selections, including received selections. |

The current writer emits all four properties. For compatibility, the reader defaults an absent `conflicts` property to an empty map.

### Session record (`sessions[session-id]`)

| Property | Values / structure | Purpose |
|---|---|---|
| `id` | UUID string matching the map key. | Stable session identity and the folder name beneath `drafts/`. |
| `device` | Device UUID matching the branch suffix. | Identifies the originating device. |
| `author` | Identity object (`name`, `email`, both strings). | Attribution for the eventual permanent commit. |
| `created_at` | Unix seconds. | Time the session was created by capture. |
| `last_edit` | Unix seconds. | Last local capture that changed the session; used for the inactivity boundary. Incoming edits do not advance this timer. |
| `revision` | Unsigned integer; a new captured session starts at `1`. | Increments on local capture updates. This is not a Git commit count and does not increment for every metadata-only transition. |
| `closed` | Boolean. | No longer accepts new edits after inactivity or explicit finish; publication may still be waiting. |
| `published` | Boolean. | The session has been published to the shared history. |
| `discarded` | Boolean, defaults to `false` when absent. | Session settled without a permanent commit because it has no effective changes. Original records remain recoverable. |
| `changes` | Object: relative file path → change record (below). | Aggregate attributable changes across the session, not every individual save. |
| `dependencies` | Array of session UUIDs. | Sessions that must settle before this session can finalize; also used to order draft integration. |
| `intents` | Object: intent UUID → intent record (below). | Recorded requested purposes associated with captured edits. |
| `message` | `null` or `{ "title": string, "description": string }`. | Prepared commit message, including local fallback messages; discarded sessions can also have an explanatory message. |
| `message_base` | `null` or shared-history commit ID string; defaults to `null` when absent. | Identifies the history base used to prepare the message. If the base changes before publication, Mesh invalidates the stale message and prepares another. |

Normal lifecycle combinations:

| Meaning | `closed` | `published` | `discarded` |
|---|---|---|---|
| Open session | `false` | `false` | `false` |
| Closed, awaiting finalization | `true` | `false` | `false` |
| Published | `true` | `true` | `false` |
| Discarded (no effective changes) | `true` | `false` | `true` |

These are separate boolean properties, not an enum. A session is considered settled when `published` or `discarded` is true. Closing alone does not mean it has reached `main`.

### Change record (`session.changes[relative-path]`)

| Property | Values / structure | Purpose |
|---|---|---|
| `base` | Blob reference or `null`. | Original version seen when this path first entered this session's retained change set. |
| `content` | Blob reference or `null`. | Current resulting version attributable to this session. |
| `intent` | Intent UUID string or `null`. | Links to `session.intents`. Mixed or unrelated edits must not inherit an earlier purpose. |
| `last_edit` | Unix seconds. | Most recent local capture affecting this path. |

### Intent record (`session.intents[intent-id]`)

| Property | Values / structure | Purpose |
|---|---|---|
| `id` | Intent UUID string. | Identifies the recorded scope. |
| `purpose` | String. CLI registration requires nonblank text of at most 8,192 bytes. | Requested purpose, not evidence that the work was completed. |
| `paths` | Nonempty array of exact relative paths. | Restricts attribution to explicitly declared files; not a glob or whole-folder scope. |
| `created_at` | Unix seconds. | Time the intent scope was registered. |

### Conflict record (`conflicts[conflict-id]`)

| Property | Values / structure | Purpose |
|---|---|---|
| `id` | SHA-256 hex digest string matching the map key. | Identifies the exact path, base, accepted, incoming, and incoming session combination. This is a domain fingerprint, distinct from a Git blob ID. |
| `path` | Relative path string. | File in the synchronized workspace. |
| `base` | Blob reference or `null`. | Common input for reconciliation. |
| `accepted` | Blob reference or `null`. | Version kept in the working view while unresolved. |
| `incoming` | Blob reference or `null`. | Competing incoming version. |
| `session` | Session UUID string. | Session supplying the incoming change. |
| `author` | Identity object. | Incoming session author. |
| `binary` | Boolean. | True if the accepted or incoming bytes contain a NUL byte or invalid UTF-8. Binary resolution selects an entire file version. |
| `created_at` | Unix seconds. | Time the conflict was recorded. |
| `accepted_sessions` | Object: contributing session UUID → per-file change fingerprint string (SHA-256); defaults to `{}` when absent. | Records exact revisions contributing to the accepted view so a selection survives different publication orders without suppressing later distinct edits. Values are fingerprints, not `session.revision` counters. |

### Resolution record (`resolutions[conflict-id]`)

| Property | Values / structure | Purpose |
|---|---|---|
| `conflict` | Conflict ID string matching the map key. | Links the selection to its preserved inputs. |
| `selected` | Blob reference or `null`. | Exact selected bytes; `null` selects absence/deletion. The current CLI chooses accepted or incoming. |
| `author` | Identity object. | Person who recorded the selection. |
| `created_at` | Unix seconds. | Time the selection was made. |

A conflict is unresolved when it has no resolution entry. Resolving it does not delete the conflict record. Concurrent selections are reconciled deterministically using timestamp and a content digest; individual device manifests remain retained.

## When `drafts/<session-id>/` appears and what it contains

1. Mesh captures a saved local change. It updates the current open session or creates a session UUID if none is open. Initialization with existing files also creates a session.
2. Further captures update that session across multiple files. There is no separate folder per save, author name, or revision.
3. On device snapshot encoding, each non-null `base` and `content` becomes a native file under that session's directory. The remote sees it after a successful upload. The runner normally attempts draft sync every three seconds after writes settle; offline work stays local until sync succeeds.
4. Five minutes without local edits closes the session by default (`idle_seconds` is configurable). `session finish` closes it sooner. Subsequent edits enter a new session, even if the previous session is still waiting for publication.
5. Closed sessions finalize after dependencies, conflict handling, and message preparation allow it. Effective changes produce an immutable shared-branch commit; no effective changes means discard without an empty commit.

The directory contains only paths in that session's current change set. It is **not a complete workspace checkout**. If capture brings a path back to its session base, the change entry is removed and its files disappear from the next snapshot tree; earlier device commits retain earlier uploaded versions. An empty `changes` map can leave a session in `state.json` with no corresponding directory at all.

### `base/` versus `files/`

`base/` holds the **before** side; `files/` holds the session's **after** side. Mesh uses both with the accepted current version for three-way reconciliation, attribution, and preservation of conflict inputs. For an existing path within a retained change, later captures update `content` while keeping its original base.

The base can include a received draft the author was editing on top of; it is not necessarily the version on `main` or the workspace state when the session began. Likewise, `files/` is the session's attributable result, not necessarily the complete current working view after other drafts have been integrated.

| Operation | `base` / `base/<path>` | `content` / `files/<path>` |
|---|---|---|
| Add a file | `null`; no file entry | Blob reference; new file bytes |
| Modify a file | Blob reference; original bytes | Blob reference; resulting bytes |
| Delete a file | Blob reference; original bytes | `null`; no file entry |
| Create an empty file | `null`; no file entry | Blob reference; actual zero-byte file |
| Empty an existing file | Blob reference; original bytes | Blob reference; actual zero-byte file |

For example, changing `notes/design.md` from `Hello` to `Hello world` stores the original bytes at `drafts/<id>/base/notes/design.md` and the resulting bytes at `drafts/<id>/files/notes/design.md`. Deleting it instead keeps the base file but sets `content: null` in the index. This is why absence from `files/` alone is insufficient to interpret a session.

## Retention and compatibility

**There is no automatic pruning or retention period in this preview.** Published and discarded sessions remain in the device manifest, and their remaining referenced snapshots continue to appear in later trees. Conflict inputs and resolutions also remain. Publishing a session to `main` does not move or remove its draft folder. An old `drafts/` folder therefore does not imply unfinished work; inspect its session flags.

Git history also retains earlier uploaded snapshot trees. Saves captured only locally between uploads are not necessarily separate remote commits. Identical content shares Git blobs, but session records, trees, commits, and distinct content still accumulate. Normal Git garbage collection preserves referenced blobs; it is not Mesh-level retention cleanup. Retention and application-level cleanup are future work. Do not delete `.mesh`, reset branch history, or remove recovery data as a cleanup shortcut.

Legacy version-one snapshots remain readable and historical commits are not rewritten. New snapshots use version two; collaborating clients must support that format. The decoder enforces the configured per-blob size limit (default 16 MiB) and a 128 MiB reconstructed snapshot limit, so indefinite accumulation remains a preview constraint.

## Sources and maintenance register

Implementation checked at source revision `296a09d`: `src/model.rs` (serialized records), `src/codec.rs` (blob mapping), `src/transport.rs` (`encode`/`decode`, file layout and validation), `src/engine.rs` (`capture`, `close_idle`, `finish`, `push_manifest`, publication and resolution), and `README.md` (supported behavior and storage growth). Existing `tests/storage_transport.rs` covers native snapshots, deleted and empty files, conflict versions, reachability after Git GC, and legacy decoding; this documentation change did not rerun those tests.

Maintenance owner: [[Kai]]. Refresh registered here and in [[index]]: review this reference whenever the transport schema, capture/session lifecycle, or retention policy changes, and recheck its source revision before use with a newer release. This is a dated implementation reference; no recurring automation is configured.

Related decisions: [[ADR-0001 — Synchronize drafts separately from permanent history]], [[ADR-0002 — Group files into sessions with an inactivity boundary]], [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions]], and [[ADR-0008 — Store content as blobs referenced by metadata]].
