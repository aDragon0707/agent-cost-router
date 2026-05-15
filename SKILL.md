---
name: agent-cost-router
description: Lightweight routing skill for choosing the cheapest safe execution path before using other skills or models. Use when the user asks about saving tokens/cost, model routing, OpenAI vs DeepSeek choice, cheap vs strong model split, prompt caching, evidence receipts, probes, long-context reduction, or which skill/model should handle a task.
---

# Agent Cost Router

## Core Rule

Before solving, choose the cheapest route that can still pass the task safely.

Do not become a giant workflow skill. Route first, then hand off:

```text
agent-cost-router = choose route, model, context budget, probe
token-prompt-compiler = compile messy input into packet/receipt
audit-evolution = after-action receipt, continuity, clean state
```

## Route Output

For nontrivial tasks, produce or silently apply:

```text
route:
model_choice:
token_policy:
skill_chain:
next_action:
stop_rule:
```

Keep it under 120 words unless the user asks for details.

## Skill Chain

Use this lightweight handoff:

```text
before work: agent-cost-router
messy input or worker prompt: token-prompt-compiler
after meaningful work: audit-evolution
```

Do not load all three skill bodies by default. Route first; load the specialist only when its job is needed.

Interaction contract:

- To `token-prompt-compiler`: send goal, scope, evidence budget, packet tier, and stop rule.
- From `token-prompt-compiler`: receive Tiny Packet, Standard Packet, Worker Packet, Evidence Receipt, or Micro Receipt.
- To `audit-evolution`: send nearest evidence, result path, score/cost deltas, and one next action.
- From `audit-evolution`: receive Tiny Audit, Evidence Receipt, or Clean-State Packet.

## Decision Rules

- Simple extraction, classification, formatting, translation, or schema filling -> cheap/no-thinking route.
- Ambiguous strategy, architecture, business judgment, safety, or high-stakes verification -> strong model after a compact receipt.
- Large documents, logs, long history, or many files -> cheap evidence receipt first, strong model only for final judgment.
- Small already-scoped task -> skip full packet; use direct execution or Tiny Packet.
- Multi-file/tool/worker task -> use token-prompt-compiler to create Standard or Full Packet.
- Finished meaningful work -> use audit-evolution Tiny Audit, not a long report.

## Context Policy

```text
stable_short_prefix -> micro/evidence receipt -> current task -> latest evidence tail
```

- Cache helps repeated long prefixes become cheaper.
- Micro Receipt avoids sending the long prefix at all.
- Prefer receipt over huge cached context when the next decision only needs facts.
- Keep full logs, diffs, traces, and raw outputs on disk; pass paths and key numbers.
- Do not judge savings from visible answer length; use provider usage/cost when available.

## Probe Policy

Use a small probe before an expensive run when:

- model behavior is unknown;
- provider usage fields are uncertain;
- a cheap model may be enough;
- strict JSON/schema/tool behavior matters.

Probe output should answer only:

```text
can_do:
usage_fields_available:
risk:
route:
```

## Model Split

Default split:

```text
cheap model: extract, filter, summarize evidence, build receipt
strong model: decide, verify, architect, debug hard failures
```

Avoid using a strong model to read the world. Use it to judge the already-compressed world.

## Stop Rules

Stop routing and ask or run a probe if:

- missing cost/quality evidence could flip the route;
- the task may spend money, publish, or mutate external systems;
- provider usage fields are unavailable and the user asks for exact cost proof.
