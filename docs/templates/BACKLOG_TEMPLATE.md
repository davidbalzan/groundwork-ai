# BACKLOG — <project>

David's inbound task queue. Phases are planned via `/plan-phase` and appear here as single items
referencing the full task breakdown. Ad-hoc tasks (bug fixes, improvements) are added directly by
David. The coordinator drains the top unblocked item and checks it off with a PR ref.

**Write rule (single-writer-per-region):**

- **`## Queue` = David's region.** Add / reorder / remove / edit items. Set priority (`P1`/`P2`/`P3`;
  top-to-bottom breaks ties). Add constraints, acceptance criteria, or refs inline.
- **`## Done` = coordinator's region, append-only.** On completion the coordinator appends a line
  with the PR ref. David prunes satisfied items from Queue.

## Queue

- [ ] (P1) Phase 1: [Phase Name] — see [[phases/phase1/PHASE1_TASKS]] · acceptance: all Phase 1 success criteria met

## Done

<!-- coordinator appends here: - [x] <task> — owner/repo#N · YYYY-MM-DD -->
