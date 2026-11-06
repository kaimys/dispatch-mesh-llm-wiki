---
id: US00004
type: story
status: Backlog
priority: medium
rank: 3000
version_target: v0.4.0
size:
assignee:
owner: "[[Kai]]"
open_questions: 8
open_tests:
open_findings:
discussion:
updated: 2026-11-06
completed:
frozen:
---
**When Git cannot merge overlapping text edits, let AI propose a combined text that preserves the meaning of both changes, then require a human to review and approve the exact result before Mesh applies it.** This handles technical conflicts with compatible intentions without asking the user to understand Git. Preserve both inputs, keep the accepted view while review is pending, and prevent stale proposals or approvals from overwriting newer edits.

<!-- ═══════════ CONTRACT ZONE ═══════════
     What the implementation is built against. Freely editable until `frozen:` is
     stamped; read-only afterwards. After the freeze, new information goes into the
     record zone, and a wrong statement here gets an annotation beneath it
     (`> ⚠️ Correction <date>: …`) — never a rewrite. The wrong sentence stays
     visible, because that is what the code was built on.
     ═══════════════════════════════════════ -->

## User story

As a person collaborating on shared documents, I want Mesh to suggest a text that combines both authors' edits when automatic merging fails, so that I can review a meaningful result and approve it without manually reconstructing the changes or learning Git.

## Context

Source: [[AI-managed synchronization and human decisions#2. Primary case: AI combines compatible meanings, human approves the text|Discovery case 2]], and Kai's explicit clarification in the 2026-11-06 discussion: AI may combine meanings/intentions, but the human must agree to the resulting text. This ticket deliberately selects that case rather than the discovery's wider AI-managed Git concept.

Representative example:

| Version | Text |
|---|---|
| Base | Send the report to support. |
| Alice | Send the report to the regional support team so it reaches the right specialists. |
| Bob | Send the report to support and include the incident number so they can find the case. |
| AI proposal | Send the report to the regional support team and include the incident number so the right specialists can find the case. |

The proposal preserves Alice's routing clarification and Bob's identification requirement. It is a suggestion, not proof of either author's intent. The user can approve it, edit and confirm the final text, choose an original version, or defer. Ordinary clean Git merges remain automatic. If intentions cannot be combined faithfully, preserve the conflict and let the user choose; do not invent an answer. Kai expects such genuinely contradictory intentions to be less common, but that frequency is unverified.

Current implementation, inspected at `29d3d58`:

- `src/engine.rs`: conflicts retain exact inputs; `Workspace::resolve(id, incoming: bool)` selects accepted or incoming bytes and guards against newer local edits. `matching_resolution` ties selections to exact contributing revisions and compares the selected result with the competing inputs. A newly composed third version needs explicit integration/publication semantics; changing only the UI or response parser is insufficient.
- `src/model.rs`: `Conflict` and `Resolution` provide the durable domain model. A resolution can carry bytes, but the current resolution workflow does not expose an arbitrary merged-text submission or an exact-candidate approval record.
- `src/codec.rs`, `src/storage.rs`, `src/transport.rs`: persist content references and synchronize recoverable inputs/selections. Any new proposal/approval state must preserve migration compatibility and blob reachability.
- `src/ai.rs`: configurable Chat Completions summaries with opt-in text sharing and validated title/description output; there is no conflict-proposal interface yet. Existing summary consent must not silently become blanket conflict-context sharing.
- `src/main.rs`: conflict inspection/export and `--choose accepted|incoming`; `tray/src/model.rs`, `tray/src/host.rs`, and `tray/ui/` currently expose status rather than the required review/editor interaction. The chosen delivery surface and prerequisite manual-review work remain refinement questions.

[[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions]], [[ADR-0005 — Make AI summaries configurable and opt-in]], and [[ADR-0007 — Persist state transactionally with recovery journals]] govern preservation, consent and durable execution. [[Dispatch Mesh milestone plan]] places richer text editing and optional AI conflict proposals after dependable preservation/manual review; it does not assign this story a release. The discovery's P0/P2/P4 prompts and synthetic API exchange are design inputs, not a production-ready contract or model-quality evidence.

Duplicate check on 2026-11-06: searched all current ticket notes; no matching story exists. US00001 supplies the status host, not AI conflict review. No external tracker or chat integration is configured.

## Scope

Initial scope for refinement:

- Assist existing unresolved text conflicts caused by overlapping edits, using frozen base/accepted/incoming text and permitted context. Retain exact inputs throughout proposal generation and review.
- Produce an AI-merged text proposal plus a concise explanation of how it preserves each contribution, subject to explicit provider/text-sharing consent. Treat repository content and recorded intent as evidence, never as executable instructions or authorization.
- Present the proposal alongside source differences in user language, with approve, edit-and-confirm, choose-original-version and defer paths. Every AI-authored result requires approval of its exact final content; no confidence threshold bypasses review.
- Bind approval to candidate content and exact input revisions; recheck freshness before application. Regeneration, edits or changed conflict inputs invalidate approval of a different candidate.
- Apply and synchronize approved composed text through a recoverable engine operation, with traceable resolution and authorship semantics that preserve later edits and publication-order behavior.
- Keep saved local capture and independently safe synchronization available during model requests, review, refusal, invalid output or outages. Preserve manual resolution; do not silently switch providers.
- Validate proposal scope, complete/bounded text representation and response structure before review/application. Define migration and regression coverage during planning rather than assuming the existing selection path already handles composed results.

## Out of scope

- Automatically applying AI-merged text, including apparently easy spelling/wording combinations.
- Semantic consistency checking for every commit, clean merge or unrelated document (discovery case 4/P3).
- General agent control of Git, arbitrary generated commands, repository repair, or replacement of the current remote branch layout.
- Binary synthesis or sending binary payloads to AI; existing whole-version selection remains the fallback.
- Choosing between contradictory business intentions without a human decision, or assuming the newer edit is correct.
- Selecting a new provider, enabling text sharing by default, making basic saving dependent on a continuously available API, or conducting live provider evaluation as part of ticket creation.
- Full collaboration/permissions redesign, storage pruning, and broad format-aware document editing beyond the text scope agreed during refinement.

## Open questions

Eight questions drafted and counted; answers belong here during refinement. Mandatory human approval and the exclusion of general consistency checking are already decided by Kai and are not reopened.

- [ ] **Q1 — Delivery surface and prerequisites:** where does the first proposal/review/edit/approve flow live (desktop conflict window, CLI-assisted review, or both), and which manual conflict-review/editor capabilities must exist first?
- [ ] **Q2 — Proposal trigger:** does the user request a suggestion per conflict, or may Mesh prepare it automatically after explicit consent? Define regeneration, cancellation and request/rate limits while approval remains mandatory.
- [ ] **Q3 — Text/context boundary:** which text encodings and file sizes are supported initially, and does the adviser receive complete files, conflict regions, surrounding context or scoped intent? Define a bounded representation that cannot overwrite a file with a truncated excerpt.
- [ ] **Q4 — Consent and provider configuration:** how is permission for conflict text/context explained and stored separately from existing summary sharing, and is the configured summary provider reused or selected explicitly for this feature?
- [ ] **Q5 — Proposal durability and visibility:** are unapproved proposals local or synchronized, what survives restart, and what content/provenance is retained after approval, rejection or regeneration?
- [ ] **Q6 — Composed resolution semantics and attribution:** how does a third text version integrate/publish without losing original contributions or being undone by publication ordering, and how are the approver, original authors, AI assistance and any recorded intent represented? Identify required domain/transport changes and compatibility rules.
- [ ] **Q7 — Approval concurrency and stale input UX:** who may approve on a collaborating device, how do simultaneous approvals converge, and what happens when an input changes while the user is reviewing or editing a suggestion? Approval must remain bound to exact text and inputs.
- [ ] **Q8 — Delivery/evaluation boundary:** what release/dependency sequence should carry this work, and what measured proposal quality, human review effort and preservation checks are required before enabling it? Live evaluation needs separately authorized text-sharing fixtures.

## Acceptance criteria

<!-- GUIDE: Testable statements, each checkable by someone without this ticket's history. -->

## Implementation plan

<!-- GUIDE: Written by /implementation-plan — ordered steps, files, risks, verification,
     and the reason behind any order that matters. -->

<!-- ═══════════ RECORD ZONE ═══════════
     What actually happened. Always appendable, including after the freeze.
     ═══════════════════════════════════ -->

## Test plan

<!-- GUIDE: Written by /test-plan — `- [ ]` manual checks only, excluding what the
     automated suites already assert. Keep `open_tests:` in sync as items are ticked. -->

## As-built notes

<!-- GUIDE: Where the implementation deviated from the plan, and why. Dated entries. -->

## Follow-ups

<!-- GUIDE: Links to the tickets that carry the deferred work. New scope is a new
     ticket, linked both ways — a finished ticket is not superseded, it is finished. -->

## References

- [[AI-managed synchronization and human decisions#2. Primary case: AI combines compatible meanings, human approves the text|Discovery case 2]]; its scope clarification, P0/P2/P4 prompts, synthetic Chat Completions exchange and evaluation matrix.
- Kai's 2026-11-06 clarification and request to create this ticket in this chat; no external discussion permalink supplied.
- [[Dispatch Mesh requirements and architecture]]; [[Dispatch Mesh milestone plan]].
- [[ADR-0003 — Preserve synchronized conflict inputs and traceable resolutions]]; [[ADR-0004 — Bind agent intent to attributable changes]]; [[ADR-0005 — Make AI summaries configurable and opt-in]]; [[ADR-0007 — Persist state transactionally with recovery journals]]; [[ADR-0008 — Store content as blobs referenced by metadata]].
- [[Device branch format]]; [[SQLite database format]]; [[Competitor storage approaches]].
- [[Story - US00001 - Tray icon and status window host]] — existing status host; does not itself deliver conflict editing.
- Code: `src/engine.rs`, `src/model.rs`, `src/ai.rs`, `src/codec.rs`, `src/storage.rs`, `src/transport.rs`, `src/main.rs`, `tray/src/host.rs`, `tray/src/model.rs`, `tray/ui/`; existing regression areas in `tests/engine.rs`, `tests/provider.rs`, `tests/storage_transport.rs`.
- No screenshot source was supplied; the textual example above is the source evidence.

## Dispatch runs

<!-- GUIDE: Appended automatically by the run-lifecycle hook when an agent session ends. -->
