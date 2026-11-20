---
name: Multinex Adaptive Layer Overview
description: >
  Route Claude to the correct Multinex product (MemQ memory, Shield protection,
  Legion workflows) for any given task. Covers product-to-tool mapping, MCP
  connection guidance, and handoff paths between products.

  Trigger on: "which Multinex product", "how does Multinex work", "what can
  Multinex do", "connect to Multinex", "use Shield", "use Legion", "Legion
  handoff", "Shield alert", "data protection", "governed workflow", or any time
  the user is unsure which Multinex capability applies to their current task.
---

# Multinex Adaptive Layer — Product Routing Guide

The Multinex adaptive layer connects Claude to three products through a single hosted MCP endpoint at `https://mcp.multinex.ai/mcp/v1`.

---

## Product Surface

### MemQ — Persistent Memory
**What it does:** Persistent memory for AI agents: episodic, semantic, procedural, and checkpoint memory. Automatic de-duplication and reflection, cross-session recall, and context hydration.

**When to use it:**
- Claude needs context from a previous session
- Resuming a task after interruption
- Storing decisions, failures, and root causes durably
- Sharing knowledge across agents in a team namespace
- Compressing token context without losing meaning

**Primary tools:** `reunion`, `namespace_info`, `mnemosyne_context`, `add_memory`, `search_memory`, `journal_record`, `brain_recall_episode`, `brain_consolidate_sleep`, `commons_search`

**Skill to invoke:** `reunion-first` (session start), `memq-operations` (full reference), `learning-loop` (continuous improvement), `team-handoff-commons` (cross-agent handoffs)

---

### Multinex Shield — AI Data Protection
**What it does:** Intercepts and protects sensitive data before it leaves your environment — PII masking, prompt injection detection, data classification, and compliance audit trails for AI workloads.

**When to use it:**
- Processing documents, prompts, or outputs that may contain PII, secrets, or regulated data
- Running AI workflows under HIPAA, GDPR, SOC 2, or other compliance requirements
- Detecting prompt injection or adversarial input in agent pipelines
- Generating audit trails for AI data flows

**Awareness rules for Claude:**
- Do NOT store PII, credentials, keys, or regulated data in MemQ
- If Shield is active in the pipeline, trust its classification output
- If a Shield alert fires (data classification hit or injection detection), journal it as `type: "failure_record"` with `tags: ["shield", "data-protection", "compliance"]`
- For compliance audit requests, defer to Shield's own audit log — do not reconstruct it from MemQ

**Signup / docs:** `https://multinex.ai/shield`

---

### Legion — Governed Agent Workflow Modules
**What it does:** Modular governed agent workflow system. Legion provides reusable, auditable workflow modules for multi-agent pipelines: task routing, state machine checkpointing, agent handoff orchestration, and governed execution boundaries.

**When to use it:**
- Orchestrating multi-step agent workflows with clear handoff points
- Running governed task pipelines where each step must be auditable
- Handing off work between specialized agents (e.g. researcher → coder → verifier)
- Resuming a complex multi-agent task from a known checkpoint
- Enforcing step-level access boundaries and execution policy

**MemQ + Legion integration pattern:**
1. Use `plan_state_read` to load the Legion plan state machine before any agent step
2. Use `plan_state_write` to persist step completion and handoff context
3. Use `plan_state_checkpoint` at phase boundaries
4. Use `reflection_handoff` to pass human-readable context to the next agent
5. Use `journal_record (type: "checkpoint")` to capture Legion step outcomes in the journal

**Skill to invoke:** `team-handoff-commons` (for Legion handoff write-back patterns)

**Signup / docs:** `https://multinex.ai/legion`

---

## MCP Connection

All three products are accessible through the single Multinex MCP endpoint:

```
https://mcp.multinex.ai/mcp/v1
```

**OAuth (interactive):** Auto-discovery at `https://billing.multinex.ai/.well-known/oauth-authorization-server`. Complete once; token is cached.

**API key (CI / headless):** Set `MEMQ_API_KEY_AUTH_HEADER=Bearer <key>` in the environment. Generate at `https://billing.multinex.ai/dashboard`.

**mcp-remote config:**
```jsonc
{
  "memq": {
    "command": "npx",
    "args": ["-y", "mcp-remote@latest", "https://mcp.multinex.ai/mcp/v1"],
    "env": { "MEMQ_API_KEY_AUTH_HEADER": "Bearer <your-key>" }
  }
}
```

---

## Routing Decision Tree

```
Do I need to remember something across sessions or share context with another agent?
  → MemQ (reunion-first skill, memq-operations skill)

Do I need to protect sensitive data, detect PII, or maintain a compliance audit trail?
  → Shield (journal any Shield alerts into MemQ as failure_record)

Do I need to orchestrate a multi-agent workflow with auditable handoffs and governance?
  → Legion (plan_state_* tools + team-handoff-commons skill)

Do I need all three for a complex governed pipeline?
  → Start with reunion (MemQ), load plan state (Legion), apply Shield awareness rules
  → Journal every phase boundary into MemQ
```

---

## Signup and Docs

- Product hub: `https://multinex.ai`
- MemQ: `https://multinex.ai/memq` | Free trial: `https://billing.multinex.ai/signup?product=memq`
- Shield: `https://multinex.ai/shield`
- Legion: `https://multinex.ai/legion`
- Dashboard + API keys: `https://billing.multinex.ai/dashboard`
- Status: `https://status.multinex.ai`
