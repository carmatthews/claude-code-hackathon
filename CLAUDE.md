# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Scenario

**Scenario 4: Data & Analytics — "40 Dashboards, One Metric, Four Answers"**

One metric is calculated four different ways depending on who you ask. The goal: one authoritative definition, a versioned calculation engine with an API, a single dashboard that replaces the 40, and an NL query layer over the semantic layer.

## Repo Structure

- `README.md` — team submission doc
- `CLAUDE.md` — this file; update as conventions are established
- `presentation.html` — HTML deck built with Claude Code, required for submission
- `decisions/` — ADRs go here (judges read these)
- `data/` — raw source data (realistic noise: gaps, mislabeled categories, timezone mismatches, duplicates)
- `engine/` — metric calculation as code with an API; each result tagged with definition version
- `semantic/` — semantic layer definitions and MCP server (`get_metric`, `list_definitions`, `explain_calculation`, `compare_periods`)
- `dashboard/` — the single dashboard replacing the 40
- `evals/` — golden question set, reconciliation table, CI harness

## The Metric Definition Is First-Class

Every assumption, edge case, and "what counts" lives in `decisions/metric-definition.md`. Replace all vague modifiers with numeric thresholds — "recent" → "within the last 14 calendar days," "significant" → specific numeric cut. Every downstream result must be tagged with the definition version that produced it.

The reconciliation table (`evals/reconciliation.md`) — rows are edge cases, columns are the five competing definitions, cells show what each returns — is the artifact that wins the stakeholder room. Prioritize it over dashboard polish.

## Three-Level CLAUDE.md Pattern

- **User-level** (`~/.claude/CLAUDE.md`) — personal preferences
- **Project-level** (this file) — shared conventions in VCS
- **Directory-level** (e.g. `engine/CLAUDE.md`, `semantic/CLAUDE.md`) — per-module stack and boundary rules

Add a directory-level `CLAUDE.md` for `engine/` and `semantic/` once those modules have their own conventions.

## Key Techniques for This Scenario

**Prompt engineering:** All metric definitions use explicit criteria with boundary examples — no "material," "significant," or "recent" without a concrete threshold. Few-shot examples for the NL query layer must include at least one question the system should *refuse* because the data genuinely can't answer it.

**MCP server over the semantic layer:** The NL query layer should stay thin. A fresh Claude session must reach for `get_metric`, `list_definitions`, `explain_calculation`, or `compare_periods` on the first try — write tool descriptions that make this unambiguous.

**Hooks vs prompts:** `PostToolUse` hook to deterministically redact PII in drill-down row results. System-prompt instructions for probabilistic preferences. Document the distinction in an ADR.

**Evals (The Scorecard):** Golden set covering normal queries + adversarial + refusal cases. Metrics: accuracy, refusal accuracy, false-confidence rate (confident-and-wrong), stratified across question types. Runs in CI so quality numbers move with every semantic-layer change.

**Agentic variance explanation (The Panel):** When the metric moves unexpectedly, spin up parallel Task subagents (geography / product / time). Each returns a structured finding with evidence. Coordinator surfaces the best explanation *and* the losing theories. Pass context explicitly in every Task prompt — subagents don't inherit coordinator context.

## Judging Priorities

Claude reads `README.md`, `presentation.html`, and `CLAUDE.md` first. Keep them current throughout — don't leave them to the end.

## Submission Checklist

- [ ] `README.md` filled from template (participants, scenario, what was built, how to run)
- [ ] `decisions/metric-definition.md` — the authoritative definition with explicit thresholds
- [ ] `evals/reconciliation.md` — edge-case table comparing all five definitions
- [ ] `CLAUDE.md` reflects actual conventions used
- [ ] `presentation.html` generated via Claude Code
- [ ] No client or internal data in the repo
