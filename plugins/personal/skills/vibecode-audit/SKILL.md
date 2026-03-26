---
name: vibecode-audit
description: Audit AI-generated ("vibe-coded") repositories for the specific anti-patterns, security holes, and quality issues that LLM coding agents systematically produce. Use when reviewing any codebase built with Cursor, Claude Code, Copilot, or other AI coding tools — or when the user asks to audit, review, or harden a vibe-coded project.
argument-hint: Optional path to repo root or specific directory to audit (defaults to cwd)
---

# Vibe Code Audit — AI-Generated Codebase Review

Systematically detect the anti-patterns, security holes, and quality issues that AI coding agents produce. Based on empirical research from CodeRabbit (470 PR study), METR (246 real-world tasks), Wiz (5,600 app scan), Variant Systems (production audit findings), and documented incidents (Moltbook, Enrichlead, Lovable, Base44, Tea App).

## When to Use

- Auditing any repo built substantially with AI coding tools
- Pre-launch security review of a vibe-coded application
- Periodic quality gate for AI-assisted development
- After extended AI coding sessions (context rot risk)
- When inheriting or reviewing someone else's AI-generated code

## Audit Protocol Overview

The audit runs in 5 phases. Detection commands for each phase are in [references/detection-patterns.md](references/detection-patterns.md).

### Phase 1: Codebase Fingerprinting

Determine the AI footprint by checking git logs for AI co-author signatures, AI scaffolding markers (`.cursorrules`, `CLAUDE.md`), and AI contribution ratios. Classify the repo:

- **Fully vibe-coded**: No human-written code → Full audit required
- **AI-assisted**: Human architecture, AI implementation → Targeted audit
- **AI-augmented**: Mostly human, AI for boilerplate → Spot-check audit

### Phase 2: The 17 Vibe Code Anti-Patterns

Scan for each pattern. Report findings with severity and file:line references.

| # | Pattern | Severity | Category |
|---|---------|----------|----------|
| 1 | Phantom Validation | CRITICAL | Security — types as runtime validation |
| 2 | Optimistic Auth | CRITICAL | Security — auth gaps on later routes |
| 3 | Assertion Mirage | HIGH | Test Quality — existence-only assertions |
| 4 | God Prompt Pattern | MEDIUM | Maintainability — monolith components |
| 5 | Secret Sprinkle | CRITICAL | Security — hardcoded secrets |
| 6 | Duplicate Divergence | MEDIUM | Maintainability — divergent duplicates |
| 7 | Missing Middle | HIGH | Error Handling — only 200/500 status codes |
| 8 | Orphan Migration | HIGH | Data Integrity — schema drift |
| 9 | Console.log Observatory | MEDIUM | Observability — no structured logging |
| 10 | Flat Auth | CRITICAL | Security — authn without authz (IDOR) |
| 11 | Silent Failures | HIGH | Reliability — swallowed exceptions |
| 12 | Tests That Lie | HIGH | Test Quality — zero mutation scores |
| 13 | Lazy Output / Code Truncation | CRITICAL | Data Loss — placeholder deletions |
| 14 | Scope Contamination | MEDIUM | Maintainability — out-of-scope changes |
| 15 | Over-Engineering | MEDIUM | Maintainability — premature abstractions |
| 16 | Regression Cascades | HIGH | Stability — fix-fix-fix cycles |
| 17 | Security Removal | CRITICAL | Security — AI deletes security controls |

### Phase 3: Slopsquatting Check

Verify all dependencies actually exist on their registries. 21.7% of AI-suggested package names are hallucinations (Trend Micro).

### Phase 4: Context Rot Indicators

Detect artifacts left when AI agents lose project context during long sessions, after context window compaction, or across multiple coding sessions. Factory.ai benchmarks (36,611 messages) show compaction preserves only 2.19-2.45/5.0 artifact trail fidelity — agents systematically forget what they've already built.

| Sub-Pattern | Signal | Severity |
|-------------|--------|----------|
| 4a. Contradictory Implementations | Multiple approaches to same concern in same codebase | HIGH |
| 4b. Lost Context Artifacts | Dead imports, orphaned functions, unused parameters | MEDIUM |
| 4c. Style Drift | Naming convention or code style shifts mid-file or across timeline | MEDIUM |
| 4d. Incomplete Refactors | Old and new pattern coexisting when migration should be complete | HIGH |
| 4e. Session Boundary Artifacts | Re-implementations of existing code, sudden convention changes in git timeline | MEDIUM |

### Phase 5: Cognitive Debt Assessment

Find files with high complexity but no explanatory comments — code that runs but nobody understands (Simon Willison's "cognitive debt").

## Execution Guidance

**Recommended order** (highest-signal first):
1. **Phase 1** (Fingerprinting): ~30s. Determines audit depth.
2. **Phase 3** (Slopsquatting): Run early — supply chain risk. May be slow due to network calls.
3. **Phase 2** (Anti-Patterns): Run CRITICAL patterns first (#1, #2, #5, #10, #13, #17), then HIGH, then MEDIUM.
4. **Phase 4** (Context Rot): Expanded — grep-based plus git timeline analysis. Sub-patterns 4a and 4d are highest signal.
5. **Phase 5** (Cognitive Debt): Last — least actionable.

**Large repos (>1000 files)**: Focus greps on `src/` or `app/` rather than repo root. Use `--include` filters aggressively. Skip Phase 5 unless specifically requested.

**Language-specific notes**: Patterns #1-#10 include both JS/TS and Python detection. For Go, Rust, or Java codebases, adapt the grep patterns to equivalent constructs (e.g., Go's `http.HandleFunc` for routing, Rust's `actix_web::web` for handlers).

## Report Format

```markdown
# Vibe Code Audit Report

**Repository**: [name]
**AI Footprint**: [Fully vibe-coded / AI-assisted / AI-augmented]
**Audit Date**: [date]
**Auditor**: Claude Code

## Executive Summary

| Severity | Count | Top Issue |
|----------|-------|-----------|
| CRITICAL | X | [worst finding] |
| HIGH     | X | [worst finding] |
| MEDIUM   | X | [worst finding] |
| LOW      | X | [worst finding] |

**Overall Risk**: [CRITICAL / HIGH / MEDIUM / LOW]
**Ship-Ready**: [YES / NO — with conditions]

## Findings by Anti-Pattern

### 1. [Pattern Name] — [SEVERITY]
**Status**: FOUND / NOT FOUND
**Files affected**: [count]
**Details**: [specific findings with file:line references]
**Fix**: [actionable remediation]

[Repeat for all 17 patterns + slopsquatting + context rot + cognitive debt]

## Remediation Priority

1. [CRITICAL items — fix before any deployment]
2. [HIGH items — fix within current sprint]
3. [MEDIUM items — schedule for next sprint]
4. [LOW items — address during regular maintenance]

## Metrics

| Metric | Value | Threshold | Status |
|--------|-------|-----------|--------|
| Weak assertion ratio | X% | <40% | PASS/FAIL |
| Console.log ratio | X:Y | >1:1 structured:unstructured | PASS/FAIL |
| Status code diversity | X codes | >4 distinct codes | PASS/FAIL |
| Auth coverage | X% routes | 100% non-public routes | PASS/FAIL |
| Hallucinated packages | X | 0 | PASS/FAIL |
| Silent catch/except blocks | X | 0 | PASS/FAIL |
| Assertion:test ratio | X:Y | >1:1 | PASS/FAIL |
| Code truncation markers | X | 0 | PASS/FAIL |
| High-churn files (2wk) | X | <3 files with >5 commits | PASS/FAIL |
| Single-impl abstractions | X | 0 unnecessary | PASS/FAIL |
| Contradictory pattern pairs | X | 0 | PASS/FAIL |
| Dead imports / orphaned functions | X | <5% of files | PASS/FAIL |
| Incomplete refactor layers | X | 0 | PASS/FAIL |
```

## References

- [Detection Patterns](references/detection-patterns.md) — All bash commands for the 17 anti-patterns and 5 audit phases
- [Metrics and Sources](references/metrics-and-sources.md) — Research statistics and source links
