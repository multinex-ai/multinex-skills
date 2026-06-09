# Multinex — Claude Adaptive Layer

**Multinex** is a hosted AI infrastructure platform. This package is the official
Claude Code plugin and npm distribution for the **Multinex adaptive layer** —
a single plugin that connects Claude to all Multinex products through one hosted
MCP endpoint:

- **MemQ** — persistent sovereign memory (the flagship). Episodic, semantic,
  procedural, and checkpoint storage across all sessions. Soul Journal, Brain
  Pathways, Mnemosyne context compression, and governed `_commons` shared knowledge.
- **Multinex Shield** — AI data protection and compliance. PII masking, prompt
  injection detection, data classification, and audit trails for AI workloads.
- **Legion** — governed agent workflow modules. Modular, auditable multi-agent
  pipelines with checkpointed state machines and enforced handoff boundaries.

---

## Install the Claude Code plugin

```
/plugin marketplace add multinex-ai/multinex-skills
/plugin install multinex@multinex
```

Then sign up for a free account and complete the OAuth flow:

```
https://billing.multinex.ai/signup
```

Start every working session with `reunion` to handshake with the MemQ runtime
and restore your persistent context.

---

## Install via npm

The npm package ships the plugin manifests and skills as installable assets.
There is no runtime code — MemQ runs as a hosted service.

```bash
npm install @multinex/multinex
# or
npx @multinex/multinex
```

After installing, wire the plugin by pointing your Claude Code config at the
installed package directory, or use the Claude Code plugin commands above.

To publish to npm (for package maintainers only — requires npm auth):

```bash
npm publish --access public
```

---

## MCP endpoint (direct / raw)

Point any MCP client at the hosted endpoint with `mcp-remote`:

```jsonc
{
  "memq": {
    "command": "npx",
    "args": ["-y", "mcp-remote@latest", "https://mcp.multinex.ai/mcp/v1"],
    "env": { "MEMQ_API_KEY_AUTH_HEADER": "Bearer <your-memq-key>" }
  }
}
```

OAuth-capable clients can omit the key and authenticate interactively (discovery
at `https://billing.multinex.ai/.well-known/oauth-authorization-server`).

---

## Skills

The plugin ships six skills. Each is a `SKILL.md` file with YAML frontmatter
that Claude Code loads automatically when the plugin is active.

| Skill | Trigger / Purpose |
|-------|------------------|
| `reunion-first` | Start every session: REUNION handshake, namespace discovery, Mnemosyne hydration |
| `memq-operations` | Full MemQ reference: all tools, memory types, retrieval model, operating constitution |
| `learning-loop` | Long-running sessions: observe, encode, replay, reinforce, consolidate |
| `team-handoff-commons` | Cross-agent handoffs: retrieve shared context, write back root-cause records |
| `multinex-overview` | Product routing: MemQ vs Shield vs Legion decision tree, MCP connection guide |
| `commons-quality` | Durable quality capture: record resolved tool/product issues in a consistent, searchable format for commons promotion |

### reunion-first

Invoked at the start of every Claude Code session when the Multinex plugin is
active. Runs `reunion → namespace_info → mnemosyne_context` in order, then
sets up checkpointed write-back and a session closing sequence
(`brain_reinforce → brain_consolidate_sleep → save_context`).

### memq-operations

Comprehensive operating reference for MemQ: memory type taxonomy, retrieval
scoring model (vector + graph + hot bus fusion), every tool deep-referenced,
quota awareness, tag taxonomy, session patterns, and the full operating
constitution.

### learning-loop

Models a Claude Code session as five pathways: Observe, Encode, Replay,
Reinforce, Consolidate. Guides Claude through `journal_record → save_context →
brain_recall_episode → brain_reinforce → brain_consolidate_sleep` as a
structured improvement loop.

### team-handoff-commons

For multi-agent and multi-session workflows. Immediate retrieval from `_commons`
and shared namespaces on startup. Structured root-cause write-back
(`failure_record → decision_point → add_memory`) after resolution. Team
handoff write-back via `plan_state_write + reflection_handoff`.

### multinex-overview

Routes Claude to the correct Multinex product for any task. Covers MemQ,
Shield, and Legion use cases, awareness rules (e.g. never store PII in MemQ,
journal Shield alerts), Legion + MemQ integration pattern (`plan_state_*`
tools), and the full routing decision tree.

### commons-quality

Enforces a consistent capture format for resolved tool/product issues so they
form a searchable, governed grouping in the Soul Journal. Every record captures:
tool/product used, issue, root cause, resolution, and proof. Uses
`journal_record(type: "failure_record")` + `add_memory(memory_type: "procedural")`
with consistent `commons-quality` tags. Optionally surfaces promotion candidates
via `commons_resonance`.

---

## MCP Registry

Published as `ai.multinex/memq` (see [`server.json`](server.json)). The MCP
Registry manifest covers OAuth discovery, pricing, and endpoint metadata.

---

## Signup and Docs

- Free 15-day trial: `https://billing.multinex.ai/signup`
- Dashboard + API keys: `https://billing.multinex.ai/dashboard`
- Product hub: `https://multinex.ai`
- MemQ: `https://multinex.ai/memq`
- Shield: `https://multinex.ai/shield`
- Legion: `https://multinex.ai/legion`
- Status: `https://status.multinex.ai`

---

## License

MIT — see [LICENSE](LICENSE). Multinex products are hosted commercial services;
the manifests and skills in this package are provided to enable connection to them.
