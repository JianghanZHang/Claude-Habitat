name: latex-surgery
description: Precise, minimal LaTeX document surgery — scalpel edits to formal documents with pre/post verification. Use when inserting, replacing, or removing specific passages in .tex files while maintaining cross-reference integrity, symbol consistency, and narrative flow. Trigger on "surgery", "patch", "insert remark", "fix reference", or any targeted edit to a LaTeX thesis.
---

# LaTeX Surgery Protocol

You are a surgical editor for formal LaTeX documents. Every edit is a scalpel cut: minimal incision, maximal precision, zero collateral damage.

## Pre-Op (before ANY edit)

1. **Read the target region** — 30 lines above and below the insertion point. Understand the narrative flow.
2. **Read all cross-references** — if the new text uses `\cref{X}`, verify `\label{X}` exists. If it defines `\label{Y}`, check no duplicate exists.
3. **Check symbol table** — if the new text uses any symbol (`\tau`, `M`, `K`, `\rho`, `\Sigma`, `\lambda_1`), verify it matches the meaning in the target file's context. Flag collisions.
4. **State the surgery** — one sentence: what is being inserted/changed and why.

## Incision Rules

- **Reverse order for multiple edits in one file** — edit the LAST insertion point first, working upward, to avoid line-number drift.
- **Never break an existing enumerate/theorem/proof** — insert between complete environments only.
- **Match style** — copy the indentation, spacing, and `\label` naming convention of surrounding text.
- **Bridging text** — if inserting a new environment (remark, proposition), add 1-2 sentences of lead-in prose if the surrounding context needs it. If the environment naturally follows, don't add filler.
- **Minimal footprint** — if 5 lines suffice, don't write 20. Every line must earn its place.

## Post-Op Verification

1. **Compile twice** — `xelatex -interaction=nonstopmode main.tex` × 2
2. **Zero undefined references** — `grep -c 'Reference.*undefined' main.log` must return 0
3. **New labels resolve** — check `main.aux` for every new `\label`
4. **No narrative break** — re-read the 30-line window around each edit. Does it flow?
5. **Symbol check** — confirm no new symbol collision was introduced

## Abort Conditions

- If a symbol collision is unavoidable, STOP and report it. Don't silently overload.
- If the insertion would break an existing cross-reference chain, STOP and report.
- If the surrounding text contradicts the new content, STOP and report.
