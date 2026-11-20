---
name: Commons Quality Issue Capture
description: >
  Record resolved commons-quality issues into MemQ consistently and durably
  using a standardized capture format. Ensures every resolved issue is
  journaled with tool/product context, root-cause analysis, resolution, and
  proof — forming a searchable, governed commons-quality grouping.

  Trigger on: "record this quality issue", "log what went wrong for the team",
  "capture root cause for commons", "this should be a commons quality record",
  "commons-quality capture", "journal this failure for the team", "resolved
  issue for commons", or any time a tool/product issue has been resolved and
  the resolution should be preserved for team reuse and commons promotion.
---

Use this skill whenever a Multinex product issue (MemQ, Shield, Legion, or any related tool) has been resolved and the fix should be captured durably for team reuse and potential commons promotion.

---

## Why This Skill Exists

Resolved issues vanish unless they are deliberately journaled with enough structure to be:
1. Found by future agents facing the same symptom
2. Promoted to `_commons` as governed reusable knowledge
3. Replayed during debugging to avoid repeating the same mistake

This skill enforces a consistent capture shape so all commons-quality records form a coherent grouping in the journal.

---

## When to Use

Use this skill when ALL of the following are true:

- A specific tool or Multinex product was involved (MemQ, Shield, Legion, or a related integration)
- An issue was encountered AND resolved (not just encountered)
- The resolution was proven — there is evidence it worked
- The fix is worth preserving for others or for future Claude sessions

Do NOT use this skill for transient issues that have no reuse value, for speculative root causes, or for unresolved blockers.

---

## Capture Protocol

### Step 1 — Record the failure entry

Call `journal_record` with the following shape:

```json
{
  "type": "failure_record",
  "content": "<concise description of the issue and its symptoms>",
  "tags": ["commons-quality", "<product-tag>", "failure_record", "rootcause"],
  "metadata": {
    "tool_or_product": "<e.g. memq | shield | legion | mcp-remote | billing>",
    "issue_summary": "<one sentence: what went wrong>",
    "root_cause": "<verified root cause — not a guess>",
    "resolution": "<exactly what was done to fix it>",
    "proof": "<how the fix was confirmed to work>",
    "promotable": true,
    "promotion_target": "_commons",
    "promotion_state": "candidate"
  }
}
```

**Product tag options** (use the nearest applicable):
`memq`, `shield`, `legion`, `billing-manager`, `mcp-remote`, `aieges-shield`, `legion-mcp`, `commons-explorer`

---

### Step 2 — Add a procedural memory for future retrieval

After the journal entry, call `add_memory` with this structured format:

```
Tool/product: <name>
Issue: <what went wrong — symptom visible to the user or agent>
Root cause: <verified root cause>
Resolution: <what was done>
Proof: <how it was confirmed>
Reuse rule: <one-line forward rule to avoid this class of issue>
```

Use:
- `memory_type: "procedural"`
- `tags: ["commons-quality", "<product-tag>", "rootcause", "resolved"]`

---

### Step 3 — Optional: surface for commons promotion review

After writing both records, optionally call `commons_resonance` with:
- `objective: "commons-quality <product-tag>"`
- `tag_filter: "commons-quality"`
- `min_support: 2`

This surfaces accumulated patterns from the namespace that may be ready for governed commons promotion. It does NOT publish to `_commons` — that belongs to the core runtime governance path.

---

## Consistency Rules

These rules make the `commons-quality` grouping coherent and searchable across sessions:

1. **Always use `tags: ["commons-quality", "<product-tag>"]`** — both tags, on every record. This is the retrieval anchor.
2. **Always use `type: "failure_record"`** for the journal entry — even when the issue has been resolved. The type describes what the entry is about, not whether it ended well.
3. **Never omit proof.** A resolved issue without proof is speculation. If you cannot state how the resolution was verified, do not write the record yet.
4. **One record per distinct root cause.** Do not collapse multiple issues with different root causes into one record. MemQ dedup handles true duplicates automatically.
5. **Keep `tool_or_product` a single canonical product name** from the approved list above. This enables `journal_search` filtering by product.

---

## Retrieval Pattern

To find all commons-quality records later:

```json
journal_search({ "tag": "commons-quality", "limit": 50 })
```

To find records for a specific product:

```json
journal_search({ "query": "<product-tag>", "tag": "commons-quality", "limit": 25 })
```

To check what's ready for commons promotion:

```json
commons_promotion_status({ "limit": 50 })
```

---

## Example Record

**Situation:** `mcp-remote` timed out on first call after session start with the MemQ plugin.

**journal_record call:**
```json
{
  "type": "failure_record",
  "content": "mcp-remote@latest timed out on first call to the MemQ MCP endpoint after session start. npx wrapper cold-start delay caused a 30s+ connect timeout. Retry succeeded on second call.",
  "tags": ["commons-quality", "memq", "mcp-remote", "failure_record", "rootcause"],
  "metadata": {
    "tool_or_product": "mcp-remote",
    "issue_summary": "First MCP call times out after session cold-start due to npx wrapper warmup",
    "root_cause": "npx wrapper initializes the mcp-remote package on first invocation; this takes 15-30s on cold systems and exceeds default MCP client timeout",
    "resolution": "Retry the first call after the timeout. Pre-warm by running a health_check call before any write operations.",
    "proof": "Second call succeeded within 2s. health_check pre-warm pattern tested across 3 sessions with no subsequent timeouts.",
    "promotable": true,
    "promotion_target": "_commons",
    "promotion_state": "candidate"
  }
}
```
