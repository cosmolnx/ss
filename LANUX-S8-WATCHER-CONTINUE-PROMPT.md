We are continuing the LANUX S.8 WATCHER follow-up integration from an
in-progress, uncommitted implementation done in a previous Claude chat.

I have uploaded a continuation package ZIP containing:
- `LANUX-S8-WATCHER-INTEGRATION-HANDOFF.md` — full technical handoff
- `LANUX-S8-WATCHER-CURRENT-STATE.md` — exact current state/test results
- `modified_files/` — the 7 files already changed/created, exact current content
- this prompt

FIRST: read the handoff and current-state documents fully, and inspect
every file under `modified_files/` before doing anything else. Do not
restart or redesign the implementation — it is substantially complete.
Do not re-derive the architecture from scratch.

DO NOT:
- restart the implementation
- redesign the locked architecture
- re-litigate the specialist-role-set question (explicitly separate, unresolved workstream)
- commit, push, merge, switch branches, reset, or rebase
- introduce a new numbered phase label (this is S.8 follow-up hardening, not "S.9")

==================================================
LOCKED ARCHITECTURE (do not reopen)
==================================================

- WATCHER is a verification layer/capability living at the
  `ToolRegistry.invoke()` execution seam. It is NOT a routable specialist.
- No `AgentRole.WATCHER`. No `WatcherAgent`. No `ModelRegistry` entry for
  WATCHER. No `Router` changes to dispatch WATCHER.
- The master orchestrator is referred to as **"Lanux s5"** (exact
  capitalization) throughout.
- Verification scope is exactly two tools: `filesystem.write_file` and
  `filesystem.edit_file`. Do not add verification to any other tool.
- `ToolSpec.verifier` is the declarative extension point (same pattern as
  the pre-existing `sensitive_args` field). It uses a two-phase
  ("pre"/"post") calling convention — this is documented precisely in the
  handoff (§5) and is the smallest change that satisfies EDIT_FILE's
  genuine need to read content before the mutating handler runs. Do not
  remove this two-phase design and do not generalize it into a broader
  hook/workflow system.
- `VerificationState` has exactly four values, unchanged from S.8:
  `NOT_ATTEMPTED`, `VERIFIED`, `FAILED`, `INCONCLUSIVE`. Their retry/audit/
  security semantics are documented in full in the handoff (§6-9) and are
  locked.
- `ToolResult.verification_state` is set ONLY by `ToolRegistry.invoke()`,
  never by a tool handler.

==================================================
WHAT'S ALREADY DONE
==================================================

Read the handoff's §4 for the full precise diff description. In short:
`agent/core/result.py`, `agent/core/tool_registry.py`,
`agent/core/model_agent.py`, `agent/core/specialists/_common.py`,
`agent/core/controller.py`, and `agent/core/task_manager.py` were all
modified; `tests/test_task_verification_s8.py` was rewritten (30 original
tests preserved with an updated, more robust fault-injection mechanism,
plus 13 new tests proving the real specialist-routed path). All of this
already passed:
- Dedicated file: 43/43 passed.
- An 18-file S.0–S.8 regression subset: 509/509 passed.
- Full repository suite: 1217 passed, 1 skipped, **1 failed**.

==================================================
YOUR TASK
==================================================

1. Apply the 7 files from `modified_files/` onto the actual current
   repository (confirm first that it is genuinely at the expected
   checkpoint — branch `s8-integration`, commit `c47e46a`, clean working
   tree — before touching anything; if it differs, stop and report rather
   than guessing).
2. Inspect the resulting diff to confirm these 7 files are the only
   changes, and that nothing in the "strict scope exclusions" list below
   was introduced.
3. Fix the ONE known, understood, remaining regression — detailed exactly
   in the handoff's §13 — in
   `tests/test_security_boundaries.py::test_invoke_always_calls_permission_gateway_check`.
   Its hardcoded "2 gateway checks expected" assumption is now stale
   because a verified `write_file` call legitimately triggers one
   additional internal `PermissionGateway.check()` call (for its
   read-back). Update the test's expectation to reflect this real,
   intentional behavior — do not weaken what the test is actually
   checking (that every single tool execution, including internal
   verification ones, passes through the gateway).
4. Run the dedicated test file, the S.0–S.8 regression subset, and the
   full repository suite. Confirm the full suite now reports **zero
   failures** (1187-passed / 0-failures / 0-errors / 0-skips was the
   original reference baseline before this integration began — note the
   real observed skip count and reconcile deviations by investigating,
   never by adjusting the target to match).
5. If a `PytestUnhandledThreadExceptionWarning` still appears in
   `test_endpoint_gating_s6.py`, spend a little time determining whether
   it's related to this work or pre-existing/unrelated, and say which —
   don't leave it uncommented on either way.
6. Inspect the final `git status`/diff and confirm scope: only the 7
   files (plus your one fix to `test_security_boundaries.py`, if needed)
   changed.
7. Do NOT commit, push, merge, switch branches, reset, or rebase. Leave
   everything as uncommitted working-tree changes.
8. Stop after reporting verified, real test results. Do not proceed into
   any further architectural workstream (role-roster reconciliation,
   new specialists, fleet/routing work, etc.) after this.

==================================================
STRICT SCOPE EXCLUSIONS (do not implement any of these)
==================================================

New specialist roles; `AgentRole.WATCHER`; `WatcherAgent`; VECTOR; MEMORY;
HACKATHON; SCOUT/FORGE/ARCHITECT/ARTISAN/SENTINEL/PILOT naming migration;
specialist roster migration; model naming migration; fallback chains;
cost-aware routing; priority queues; resource budgeting; fleet execution;
parallel multi-agent execution; UI; voice; browser/computer-control
expansion; self-improvement; autonomous learning; authentication
redesign; authorization redesign; audit schema redesign; a new numbered
phase.

Report results only when done — files changed, exact test results for
all three levels (dedicated / regression / full suite), confirmation of
no scope creep, and explicit confirmation that nothing was committed,
pushed, merged, or branch-switched.
