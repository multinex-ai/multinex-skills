---
name: MemQ Brain
description: >
  Use this skill whenever Claude should operate with the MemQ hosted
  intelligence fabric. Covers persistent memory, cross-session recall,
  the journal, associative recall, episode replay, reflection and
  consolidation tools, context hydration, and _commons governed shared
  knowledge.

  Trigger on: "remember this", "what did we decide", "save context",
  "recall from last session", "use MemQ", "brain discover", "load context
  from memory", "journal this", "commons search", "hydrate context", "checkpoint",
  "reflect", "consolidate", "reunion", "namespace info", any request for
  cross-session continuity, or any time the memq MCP server is connected.
---

# MemQ Brain — Comprehensive Claude Skill

---

## 1. How MemQ Behaves (What To Rely On)

MemQ is a hosted memory service. What you save is stored durably and recalled
by relevance to the current namespace and task, not by exact key.

- **Duplicates are handled for you.** Saving the same thing again reinforces
  the existing memory instead of creating a copy. Do not call `add_memory` in a
  loop expecting every entry to persist separately.
- **Consolidation also runs on the service.** Manual calls to `reflect_memory`
  or `brain_consolidate_sleep` are additive, not redundant.
- **Recall combines several signals.** Treat scores as relative within one
  result set, and verify anything important against current sources.
- **Access follows your sign-in.** You cannot see another tenant's data and
  cannot write to `_commons` directly through the hosted bridge.

---

## 2. Connection and Auth

**MCP endpoint:** `https://mcp.multinex.ai/mcp/v1`

**OAuth (interactive, default):** `mcp-remote@latest` will auto-discover the
OAuth server from the endpoint's `.well-known/oauth-protected-resource/mcp/v1`
metadata and redirect to `https://billing.multinex.ai` for the PKCE flow.
Complete once; the token is cached.

**API key (CI / service accounts):**
Set `MEMQ_API_KEY_AUTH_HEADER=Bearer <key>` in the environment.
Get a key at `https://billing.multinex.ai/dashboard`.

---

## 3. Tier and Namespace Model

| Tier | Namespace access |
|------|-----------------|
| `FREE` | `_commons` only — no private namespace |
| `BASE` | `_commons` + exactly one private namespace |
| `VERIFIED` | `_commons` + one authenticated private namespace |
| `TEAM` | `_commons` + team/org namespace and approved subteam scopes |

Rules that must never be violated:

- Private namespaces are locked. No other tenant can read them.
- `_commons` is governed shared knowledge, not a personal scratchpad.
- The hosted bridge cannot promote private memory into `_commons` directly.
  `commons_resonance` surfaces candidates; publication belongs to the core
  runtime governance path.
- `FREE` tier has no private write surface. All `add_memory` calls land in
  `_commons` scope only.

---

## 4. Session Opening Sequence (Always Run in Order)

```
1. reunion          — handshake, confirm protocol + auth posture + namespace
2. namespace_info   — verify active namespace, tier, journal footprint
3. mnemosyne_context (objective: "<current task>")
                    — hydrate relevant context before deep work
4. (optional) brain_recall_episode or brain_associate
                    — only if the task depends on a prior session or known failure
```

Do not skip `reunion`. It returns the protocol version, deployment mode,
access level, billing posture, and namespace status. Everything after it
depends on knowing the scope.

Do not skip `mnemosyne_context`. It compresses recent episodic, semantic,
procedural, and checkpoint memory into a token-efficient pack. Running it
before reasoning avoids re-reading the entire journal.

For research tasks, shared operational questions, public-source synthesis,
or cross-team pattern lookup, call `commons_search` alongside private namespace
retrieval. `_commons` is the only shared readable namespace; use it for
governed reusable knowledge, never for private tenant implementation details.

---

## 5. Memory Types — What Each Is For

Every write to MemQ requires a `memory_type`. Choosing the wrong type
degrades retrieval because filters and replay selection are type-aware.

| Type | Purpose | When to use |
|------|---------|-------------|
| `episodic` | What happened, in sequence | Events, observations, decisions with temporal order |
| `semantic` | Abstract facts, rules, concepts | Verified facts, domain knowledge, distilled patterns |
| `procedural` | How to do something | Step sequences, call orders, workflows, protocols |
| `checkpoint` | Resumable state snapshot | End of phase, before risky operation, session boundary |
| `hybrid` | Combined bridge records | bridge_sync, reflection_handoff — set automatically |
| `reflection` | Consolidation output | Set by reflect_memory/brain_consolidate_sleep — do not set manually |

**Critical:** `reflection` memories are written by the system. Writing your
own text with `memory_type: "reflection"` does not trigger consolidation — it
just creates a record tagged as a reflection. Use `reflect_memory` or
`brain_consolidate_sleep` to trigger actual consolidation.

---

## 6. Tool Selection Decision Tree

```
Do I need to START a session?
  → reunion → namespace_info → mnemosyne_context

Do I need RECENT activity (what just happened)?
  → recent_memory

Do I need RELEVANT knowledge (what do I know about X)?
  → search_memory / query_memory (private)
  → commons_search (shared)
  → brain_discover (with AI synthesis)

Do I need to TRACE context around a known memory or cue?
  → brain_associate

Do I need to REPLAY a session timeline in order?
  → brain_recall_episode

Do I need COMPRESSED context to save tokens?
  → mnemosyne_context

Do I need to WRITE a decision, fact, or failure?
  → add_memory (for recallable facts)
  → save_context (for episodic/checkpoint framing with observation)
  → journal_record (for typed, auditable journal entries)

Do I need to REFLECT and consolidate at a session boundary?
  → brain_consolidate_sleep (with persist: true)
  → brain_reinforce
  → reflect_memory (for a targeted reflection pass)

Do I need to PREDICT next actions or risks?
  → brain_predict

Do I need to see PROMOTION CANDIDATES for _commons?
  → commons_resonance
  → commons_promotion_status

Do I need to CHECK system health or quota?
  → health_check
  → memory_status

Do I need to VERIFY namespace and tier?
  → namespace_info

Do I need to CHUNK a large document?
  → slicer_slice (before add_memory)
```

---

## 7. Session Patterns

### New task in a continuing project
```
reunion
namespace_info
mnemosyne_context (objective: "<task name>")
search_memory (query: "<relevant aspect>")
commons_search (query: "<shared pattern if applicable>")
→ begin work
journal_record (type: "decision_point", key decisions as they happen)
save_context (kind: "checkpoint", at phase boundaries)
→ complete work
brain_reinforce
brain_consolidate_sleep (persist: true)
save_context (kind: "checkpoint", final state summary)
```

### Resuming after interruption
```
reunion
namespace_info
recent_memory (limit: 20)
brain_recall_episode (session_id: "<prior session>", include_related: true)
mnemosyne_context (objective: "<resumption task>")
→ continue from checkpoint
```

### Debugging a regression
```
reunion
namespace_info
journal_search (query: "<error or component>", tag: "failure_record")
brain_associate (cue: "<error message or component>", top_k: 10)
search_memory (query: "<related system>", filters: { memory_type: "procedural" })
commons_search (query: "<known patterns for this class of failure>")
→ root cause analysis
journal_record (type: "failure_record", content: root cause + resolution)
brain_reinforce (trigger_reflection: true)
```

### Session closing sequence (always)
```
brain_reinforce (trigger_reflection: false)
brain_consolidate_sleep (limit: 100, persist: true, include_commons_candidates: true)
save_context (kind: "checkpoint", content: compact summary of what changed,
              what was verified, what remains uncertain, key risk)
```

---

## 8. Operating Constitution

1. **Session start: run the opening sequence.** `reunion` → `namespace_info`
   → `mnemosyne_context`. Every time, without exception.

2. **Read before deciding.** Call `recent_memory` or `search_memory` before
   any decision where prior context could change the outcome.

3. **Search commons when work touches shared knowledge.** Use `commons_search`
   before designing any architecture, API contract, or cross-team workflow.

4. **Write only useful outcomes.** Do not dump raw transcripts. Persist
   decisions, verified facts, reusable procedures, failure root causes, and
   resumable checkpoints.

5. **Tag everything consistently.** Tags are the primary filter surface.

6. **Namespace boundaries are hard.** Never store private data (keys, PII,
   credentials) in MemQ. Never claim commons publication happened through the
   hosted bridge.

7. **Close sessions.** Always run the closing sequence.

---

## 9. Signup and Docs

- Free account: `https://billing.multinex.ai/signup?product=memq`
- Dashboard + API keys: `https://billing.multinex.ai/dashboard`
- MCP endpoint: `https://mcp.multinex.ai/mcp/v1`
- Status: `https://status.multinex.ai`
