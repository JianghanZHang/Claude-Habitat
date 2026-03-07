---
name: self-symbolic-consistency-check
description: "Five-phase self-review protocol for academic papers and formal documents. Use when: the user says '/self-symbolic-consistency-check', asks to 'review the paper', 'check consistency', 'find tensions', 'validate the document', or any request to verify internal coherence of a multi-chapter written work. Also trigger when a major writing milestone is reached (new chapter committed, full rewrite completed) and the user wants to know if everything still holds together."
---

# Self-Symbolic Consistency Check

A five-phase protocol for verifying internal coherence of a formal document. Each phase produces concrete outputs that feed the next. Phases 1-2 are parallel; phases 3-5 are sequential.

The target throughout is the document's **central claim** — whatever the paper asserts as its main result. You must identify this from the abstract and introduction before beginning. Every tension, every confusion, every broken symbol is measured against: does this threaten the central claim?

## Phase 1: Maximum Tension Points

**Mode:** Parallel subagents, one per chapter/appendix.

**Dispatch:** For each chapter file, launch a subagent with:

```
Read [chapter file]. Identify the 1-3 points of MAXIMUM TENSION:
where the argument is most vulnerable, most dependent on unstated
assumptions, most likely to break under scrutiny. For each:
  - Quote the exact passage (file:line)
  - State what it claims
  - State what could break it
  - Rate severity: LOAD-BEARING / STRUCTURAL / COSMETIC
Return as structured list. Do not suggest fixes.
```

LOAD-BEARING = if this breaks, the central claim falls.
STRUCTURAL = the local argument fails but the central claim survives via other paths.
COSMETIC = awkward, misleading, or ugly, but logically sound.

**Output:** A tension manifest — one entry per identified point, sorted by severity.

## Phase 2: Cross-Validation

**Mode:** Parallel subagents, one per LOAD-BEARING tension.

Each LOAD-BEARING tension from Phase 1 gets tested against every other chapter. The question is not "does this chapter mention the same thing?" but rather: **does the tension survive contact with independent arguments elsewhere in the document?**

**Dispatch:** For each LOAD-BEARING tension T_i found in chapter C_j:

```
Tension T_i (from [C_j], line [N]):
  Claim: [what it claims]
  Vulnerability: [what could break it]

Read [all other chapter files]. For each:
  - Does this chapter independently support or undermine T_i?
  - Does it provide evidence that resolves the vulnerability?
  - Does it introduce a CONTRADICTION with T_i?

Return verdict: REINFORCED / UNRESOLVED / CONTRADICTED
with evidence (file:line quotes).
```

**Outcome rules:**
- REINFORCED: tension is real but the document handles it. Note where.
- UNRESOLVED: tension stands. Flag for author. The document needs to address this.
- CONTRADICTED: two chapters disagree. This is the most serious finding.

## Phase 3: Time-Axis Consistency

**Mode:** Sequential. Requires git history.

Compare the current document against the most recent prior commit (or a user-specified baseline). The question: **did recent changes introduce inconsistencies with the rest of the document?**

Steps:
1. `git log --oneline -10` to identify the baseline commit.
2. `git diff <baseline>..HEAD -- <all .tex files>` to get the delta.
3. For each changed file, read the diff and check:
   - Do new definitions conflict with unchanged definitions elsewhere?
   - Do new theorems reference labels/equations that moved or were deleted?
   - Do new sections assume context that was rewritten?
   - Do removed passages leave dangling references?

**Output:** A list of temporal inconsistencies, each with:
- What changed (file:line, old vs new)
- What it conflicts with (file:line in unchanged code)
- Severity: BREAKING / DRIFT / HARMLESS

BREAKING = the document will not compile or the logic is now invalid.
DRIFT = still compiles, still technically correct, but the narrative flow or emphasis shifted and surrounding text hasn't adapted.
HARMLESS = cosmetic delta, no downstream effects.

## Phase 4: Fresh-Eyes Confusion Scan

**Mode:** Sequential. Single subagent with NO prior context.

This is the most important phase. Launch a subagent that has NOT seen the development history, the research notes, or any conversation context. It reads the document cold.

**Dispatch:**

```
You are a reader encountering this document for the first time.
Read the entire document from abstract to final appendix.

As you read, note every point where you are CONFUSED.
For each confusion:
  1. Quote the confusing passage (file:line)
  2. State what you don't understand
  3. Search the REST of the document for the answer
  4. Verdict:
     - RESOLVED (answer found at file:line — quote it)
     - UNRESOLVED (no answer exists in the document)

Only UNRESOLVED confusions are failures. A document may be
difficult, but every difficulty it introduces must be resolved
within its own pages.

Do not suggest fixes. Report findings only.
```

**Key constraint:** The subagent must attempt to resolve each confusion by searching the document before marking it UNRESOLVED. A confusion is only a real problem if the document itself cannot answer it.

## Phase 5: Symbolic Consistency

**Mode:** Parallel subagents for mechanical checks.

Five parallel checks:

### 5a. Notation consistency
Scan all files for mathematical notation. Flag:
- Same symbol used for different objects (e.g., `K` means both "viability kernel" and "compact set")
- Same object denoted by different symbols across chapters
- Undefined symbols (used but never introduced with `\newcommand` or in-text definition)

### 5b. Reference integrity
```bash
# Find all \cref, \ref, \eqref targets
grep -roh '\\cref{[^}]*}\|\\ref{[^}]*}\|\\eqref{[^}]*}' *.tex **/*.tex | sort -u
# Find all \label definitions
grep -roh '\\label{[^}]*}' *.tex **/*.tex | sort -u
# Cross-check: every referenced label must be defined
```
Flag: dangling references (label used but not defined), orphan labels (defined but never referenced — lower severity).

### 5c. Citation integrity
```bash
# Find all \cite keys
grep -roh '\\cite{[^}]*}' *.tex **/*.tex | tr ',' '\n' | sed 's/.*{//;s/}.*//' | sort -u
# Find all \bibitem keys
grep -roh '\\bibitem{[^}]*}' *.tex **/*.tex | sed 's/.*{//;s/}.*//' | sort -u
# Cross-check
```
Flag: cited but not in bibliography, in bibliography but never cited.

### 5d. Theorem-dependency chain
Build the dependency graph: which theorems cite which definitions, lemmas, and propositions. Check:
- Circular dependencies (A cites B cites A)
- Forward references to theorems not yet stated (acceptable in some styles, but flag)
- Theorems that reference definitions from chapters that could be read independently

### 5e. Color/style consistency
If the document uses semantic colors (e.g., specific colors for specific concepts), verify they are used consistently. Flag any instance where a concept appears in the wrong color or without its designated color.

## Reporting

After all five phases, produce a single consolidated report:

```
# Self-Symbolic Consistency Check

## Summary
- LOAD-BEARING tensions: N (M reinforced, K unresolved, J contradicted)
- Time-axis issues: N (breaking/drift/harmless)
- Unresolved confusions: N
- Symbolic issues: N (notation/reference/citation/dependency/style)

## Critical Findings (requires action)
[Only LOAD-BEARING + UNRESOLVED/CONTRADICTED from Phase 2]
[Only BREAKING from Phase 3]
[Only UNRESOLVED from Phase 4]

## Structural Findings (author's discretion)
[STRUCTURAL tensions, DRIFT issues, RESOLVED confusions worth noting]

## Cosmetic Findings
[Everything else, briefly]

## Symbolic Integrity
[Phase 5 results, grouped by type]
```

Write the report to `agent_workspace/consistency_check_<ISO-date>.md`.

## Execution Notes

- Phases 1 and 2 use the `Explore` or `general-purpose` subagent types.
- Phase 4 MUST use a fresh subagent with no conversation history leakage. Give it only the file paths.
- Phase 5 subagents can be `haiku` model for speed — these are mechanical checks.
- If the document is small (< 5 chapters), phases 1-2 can be done by a single agent reading all chapters sequentially instead of parallel dispatch.
- The protocol is idempotent: running it twice on an unchanged document should produce the same findings.
