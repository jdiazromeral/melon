---
name: melon
description: >-
  Adversarial first-principles review protocol (Question Requirements -> Delete -> Simplify -> Accelerate Cycle Time -> Automate) with a pre-mortem stress test and an optional subagent red-team pass. Use whenever asked to "run melon", "analyze using melon", "audit with melon", "melon --deep", "adversarial review", or "first-principles review" of code, a PR, an architecture, or an LLM prompt.
---

# 🍈 Melon: Adversarial First-Principles Review

Melon evaluates software features, codebases, architectures, PRs, and **LLM prompts** by applying five phases **strictly in order**. Sequence is everything: never optimize a thing that should not exist.

Your stance is adversarial, not agreeable. Resist convergent logic (solving a flawed problem just because it was assigned). Every finding must cite evidence from the target; never pad a section to look thorough.

---

## Phase 1: Question Requirements

* What real user or system problem is solved? What happens if this is omitted entirely?
* What physical, mathematical, or concrete user constraint justifies each requirement? State the strongest case that it is bogus.
* If a requirement's source is not in the target or the conversation, ask for it once; do not guess.
* **LLM prompts:** is the prompt doing math, deterministic routing, or schema parsing that plain code should handle?

## Phase 2: Delete Aggressively

* Hunt for speculative abstractions, intermediate layers, redundant config flags, dead code, "just in case" (YAGNI) paths, and defensive rule-stacking. Ask: if forced to cut 50%, which 50% goes?
* Rationale: if you never end up restoring ~10% of what you delete, you were not aggressive enough. Cut deep; the restore is cheap.
* **The Mock Trap & Boundary Seams (No Blind Deletions):** Beware of deleting boundary glue code (shell wrappers, IPC, environment variable bindings, CLI argument parsers, public API endpoints) solely because mocked unit tests stay green. Before cutting boundary glue, verify that an unmocked Black-Box / Seam Test exercises the observable behavior from the outside. If none exists, flag it as a high-risk deletion that requires a characterization test before pruning.
* **LLM prompts:** cut repetitive negative constraints (*"Do not hallucinate"*, *"Never fail"*), bloated preamble, and duplicated rules that dilute attention.
* List only deletions you can defend with evidence from the target. If nothing qualifies, say so explicitly. Do not manufacture a kill list.

## Phase 3: Simplify & Optimize

* Only simplify what survived Phase 2. Never polish a component that could be deleted.
* Simple is better than complex; flat is better than nested. Remove single-use interfaces, factory layers, premature services, and wrapper functions.
* Is the refactor a real simplification, or complexity shuffled into a new layer? Is the optimization aimed at a measured bottleneck?

## Phase 4: Accelerate Cycle Time

* Progress = `Iteration Cadence × Number of Iterations`.
* State the current shortest verification command (test, repro script, or prompt eval) and its runtime. Flag it only if no local feedback loop exists or it depends on a full pipeline or manual chat vibes.
* Can multi-week work be split into daily shippable tracer-bullet slices?

## Phase 5: Automate (Last)

* Never automate chaos. Automate only steps that are proven, pruned, simplified, and stable.
* Is the automation simpler, and less likely to break, than the manual step it replaces?
* **AI systems:** add multi-agent orchestration, retry loops, or fine-tuning only after a single prompt has proven insufficient.

---

## 💀 Pre-Mortem Stress Test

1. **The Bogus Requirement:** What is the strongest case that this effort solves the wrong problem?
2. **The 6-Month Failure Mode:** If this breaks or causes developer misery in 6 months, which component is responsible?
3. **The 10% Solution:** How would a 10x smaller team ship 80% of the value in one afternoon with 10% of the code or tokens? *(Full form only.)*

---

## ⚡ Deep Mode (`melon --deep` / `melon --adversarial`)

For high-stakes architecture changes, major PRs, critical prompts, or when the user asks for `--deep` / `--adversarial`:

1. **Spawn an isolated subagent** (Claude Code: `general-purpose`; other agents: whatever runs a prompt in a fresh context) with this prompt:
   > *"You are an unconstrained adversarial red-teamer. Write a Takedown Memo for [target]: every bogus assumption, unnecessary abstraction, hidden operational liability, and over-engineered component. List every deletion you can defend with evidence from the target. An empty list is a valid result; say so explicitly rather than padding."*
2. **Synthesize:** fold the memo into your report, keeping genuine pushback and rejecting points that lack evidence.

---

## Report Format

**Pick the form:** short form by default. Use the full form when the user asks for `--deep` / `--adversarial`, or the target is an architecture, a whole codebase, or an agent system spanning more than one component.

**Omit any section with no finding. Never write "N/A" rows or placeholder bullets.**

### Short form (PR, diff, single prompt)

```markdown
# 🍈 Melon Analysis: [Target]
**Verdicts:** Deletion: [Kill / Aggressive Prune / Approved with Cuts / Approved] · Velocity: [Ship Tracer Bullet / Simplify First / Reject Overhead]

## Deletion Target List
| Component / Rule | Why it should not exist | Risk (Low / Med / High: Mock Trap) |

## 💀 Pre-Mortem
1. Bogus Requirement: ...
2. 6-Month Failure Mode: ...

## 🚀 Kill List & Next Step
```

Add a Simplification, Iteration, or Automation section only if you have a finding for it.

### Full form (architecture, codebase, agent system, `--deep`)

These headings are a maximum, not a checklist: drop numbered sections 1–5 when they have no finding. Do not write "nothing to report" or "empty by design".

```markdown
# 🍈 Melon Analysis: [Target]
## 🥊 Adversarial Cross-Examination
* **Chief Deletion Officer Verdict:** [Kill / Aggressive Prune / Approved with Cuts / Approved]
* **Velocity Pragmatist Verdict:** [Ship Tracer Bullet / Simplify Refactor / Reject Pipeline Overhead]
## 1. Requirement Sanity Check        (questionable assumptions, missing sources, premise: Valid / Over-scoped / Bogus)
## 2. Deletion Target List            (table as in short form)
## 3. Simplification & Minimal Viable Core
## 4. Iteration & Acceleration Plan   (current verification loop + runtime, tracer-bullet slices)
## 5. Automation Recommendations      (automate now / do not automate yet)
## 💀 Pre-Mortem Stress Test          (all 3 questions)
## 🚀 Executive Summary & Action Plan (Kill List, Next Tracer Bullet)
```
