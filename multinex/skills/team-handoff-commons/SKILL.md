---
name: Team Handoff and Commons
description: >
  Start Claude Code with immediate MemQ handoff retrieval, then store resolved
  issue root cause and commons-candidate fixes as work completes.

  Trigger on: "resuming an inherited task", "picking up where we left off",
  "team handoff", "what did the last agent do", "shared namespace context",
  "leave behind a handoff", "publish to commons", "root cause for the team",
  collaborating with other agents, or fixing a problem that should become
  reusable team knowledge.
---

Use this skill whenever Claude Code is resuming an inherited task, collaborating with other agents, or fixing a problem that should become reusable team knowledge.

Immediate retrieval sequence:

1. Call `reunion` first. If the hosted runtime returns a startup `503`, retry after a short delay and keep journaling locally scoped progress instead of treating MemQ as missing.
2. Call `hybrid_retrieve` against `_commons` with the current objective, subsystem, and failure mode so Claude Code sees the strongest reusable fix patterns first.
3. Call `hybrid_retrieve` against the active shared namespace such as `org:{id}:shared` to pull the freshest operator handoff, task-specific checkpoints, and unresolved risks.
4. If a `plan_id` is known, call `plan_state_read` before planning or editing so Claude Code resumes the actual state machine, not an inferred one.

Resolved issue workflow:

1. Record the failure with `journal_record` using `type: "failure_record"`.
2. Include:
   - symptom
   - wrong assumption
   - trigger
   - scope
   - containment
3. After resolution, record `journal_record` using `type: "decision_point"` with:
   - root cause
   - corrective action
   - validation evidence
   - forward rule for avoiding recurrence
4. Write `add_memory` with a compact procedural pattern:
   - `Problem:`
   - `Root cause:`
   - `Fix:`
   - `Validation:`
   - `Reuse rule:`

Team handoff write-back:

1. Use `plan_state_write` in the shared namespace to persist current status, cleared blockers, pending checks, and the exact next step.
2. Use `reflection_handoff` to publish the human-readable handoff summary with tags including:
   - product tag
   - subsystem tag
   - `handoff`
   - `resolved`
   - `rootcause`
   - `commons-candidate`
3. Include metadata fields:
   - `root_cause`
   - `wrong_assumption`
   - `corrective_action`
   - `validation`
   - `promotable: true`
   - `promotion_target: "_commons"`
   - `promotion_state: "candidate"`

Governance rule:

- Shared team namespaces are the normal write path for corrective streams.
- `_commons` is for sanitized, reusable patterns only.
- If automatic commons publication is outside the current trust boundary, mark the record as a promotion candidate and keep the reusable analysis intact for later governed publication.

Claude Code operating rule:

- Before solving from scratch, retrieve both `_commons` and shared-namespace context.
- After any resolved issue, leave behind both a checkpointed handoff and a reusable root-cause record.
- Treat mistake analysis as part of done, not optional documentation.
