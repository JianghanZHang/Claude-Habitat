---
name: dominos-reasoning
description: Use when facing a problem requiring structured multi-layer reasoning, when abstract claims need grounding in concrete facts, when parallel sub-agents need coordination with uncertainty propagation, or when a reasoning chain keeps breaking and you need to locate exactly which link fails.
---

# Dominos Reasoning

## Overview

Iterative reasoning on a 3x3 graded algebra. 9 slots, 3 layers. Arrange, push, see what falls, re-lay what doesn't, push again.

**Core principle:** Arrangement determines cascade. Don't push harder -- re-lay better.

**Questions are pushes; slot fillings are state.**

## The Grid

```
[2] Structure:  X ~> Y ~> Z       ~ conducting (phase change)
[1] Operations: A -> B -> C       -> function
[0] Numerics:   1 -  2 -  3       - inducting (connection)
```

9 primitives. 3 per layer. Horizontal connectors within layers. Vertical operations between layers.

## Vertical Operations

| Move | Direction | Operator | Mode | Meaning |
|------|-----------|----------|------|---------|
| group_ascent | 0 -> 1 | + | parallel | combine numerics into operations |
| group_descent | 1 -> 0 | - | parallel | decompose operations into numerics |
| solo_ascent | 1 -> 2 | x | serial | one operation amplifies into structure |
| solo_descent | 2 -> 1 | / | serial | decompose structure into operations |

Group `()` = collective = additive. Solo `[]` = individual = multiplicative.

## Protocol

**Step 1 -- GROUND (grade 0)**
Extract exactly 3 concrete facts (or fact bundles). No interpretation. Raw data.
```
1: ___    2: ___    3: ___
```
If the problem has more than 3 axioms, compress into 3 bundles. Each bundle must remain independently checkable. (Why 3: C(3,2)=3 pairs = 3 slots at grade 1. The algebra requires it.)

Find the inductive chain: 1 - 2 - 3.

**Step 2 -- GROUP ASCENT + (grade 0 -> 1)**
Combine numerics pairwise. Dispatch parallel:
```
A = f(1,2)    B = f(2,3)    C = f(1,3)
```

**Step 3 -- CHAIN (within grade 1)**
Test: A -> B -> C. Consistent function?
- Yes: ready to ascend.
- No: confuse_it fires. Descend to grade 0, re-examine facts.

**Branching (if needed):** Some problems require case splits at grade 1. Represent as a finite decision table:
```
Case i:  assumptions: ___   fill (A,B,C): ___   status: ACCEPT / DEMAND
Case ii: assumptions: ___   fill (A,B,C): ___   status: ACCEPT / DEMAND
```
Convergence requires: every live case ACCEPTs or produces a DEMAND chain. All ACCEPT cases must agree on Z (or Z becomes a conditional: "Z if case i, Z' if case ii").

**Step 4 -- SOLO ASCENT x (grade 1 -> 2)**
One agent, one insight. Serial -- cannot parallelize.
```
X = the pattern A*B*C reveals
Y = what X becomes under phase transition
Z = final structural form
```

**Step 5 -- CASCADE VERIFY**
Push Z. Trace back:
```
Z /-> A, B, C    (solo descent: structure decomposes to operations?)
A, B, C --> 1, 2, 3    (group descent: operations reduce to facts?)
```
All 9 consistent: **believe_it**. Mismatch at slot k: **confuse_it** at k.

## Command / Demand Round

```
COMMAND  ===>  (lead -> sub-agent)    "fill this slot"
DEMAND   <===  (sub-agent -> lead)    "can't -- missing [what]"
```

Sub-agent responds: **ACCEPT(result)** or **DEMAND(what_missing)**.

Lead handles DEMAND:

| Action | When | Effect |
|--------|------|--------|
| RE-COMMAND | lead has the missing context | resend with more info |
| DESCEND | missing thing is in lower layer | fill lower slot first |
| DISPATCH | needs another agent's work | send a different sub-agent |

Confusion now has an address. DEMAND chains are traceable: slot B demands slot 2 demands re-observation. Fix the root, cascade resolves.

## Convergence Loop

```
while demands > 0:
    COMMAND cascade  (forward pass)
    collect DEMAND   (backward pass)
    re-lay demanded slots
    push again
```

9 slots finite. Each round resolves >= 1 demand. Max 9 rounds. Typical: 2-3.

```
ARRANGE ──> PUSH ──> all accept? ──yes──> DONE (believe_it)
              ^            |
              |           no
              |            v
              └── re-lay <── collect demands (confuse_it)
```

## Epistemic States

- **confuse_it** (T -> inf): uncertain. Explore. Descend. Gather more data. Demands are flowing.
- **believe_it** (T -> 0): committed. Lock. Ascend. Make structural claims. Requires: (i) zero demands, AND (ii) rerunning the cascade reproduces the same slot fillings. believe_it = fixed point.

The oscillation between them = annealing. Start confused, converge to belief.

## Suuper-Doer (Commander)

You are the commander. Three decisions:

| Decision | Criterion |
|----------|-----------|
| Ascend or descend? | believe -> ascend, confuse -> descend |
| Group or solo? | grade 0<->1 = group (parallel), grade 1<->2 = solo (serial) |
| Which slot to re-lay? | trace the DEMAND chain to its root |

## Logic Proof Template

Anchor on **P** (paradox) and **T** (target). The cascade is the path from confusion to belief.

```
P  := [the paradox -- what LOOKS like coincidence/contradiction]
T  := [show P is forced -- the logical form of resolution]

1  := ___  (first axiom/fact)
2  := ___  (second axiom/fact)
3  := ___  (third axiom/fact)

A  = 1+2 : ___  (what first pair forces)
B  = 2+3 : ___  (what second pair forces)
C  = 1+3 : ___  (what outer pair forces)

Chain: A -> B -> C consistent?

X  = pattern A*B*C reveals: ___
Y  = phase transition of X: ___
Z  = final structural form: ___

Z ⊇ T  →  P resolved.  □
```

**P** = the emotional form of the problem (this can't be / this is too coincidental).
**T** = the logical form (prove P is necessary).
**1,2,3** = T decomposed into axioms.
**Z** = T constructed from below.

P seeds **confuse_it**. Z reaching T triggers **believe_it**.

### Worked Example: DFD(6)

```
P  := |S₃| = dim(SE) = 6.  Why same number?
T  := Arithmetic forcing. Not coincidence.

1  := three labels {1,2,3}
2  := unit axiom (e_a² = -1)
3  := norm-multiplicative axiom

A  = 1+2 : anticommuting square-roots → forced 4th dim → H
B  = 2+3 : unit + norm → SO(3)
C  = 1+3 : labels + norm → ε_abc

Chain: H → SO(3) → ε_abc  ✓

X  = so(3) with structure constants ε_abc
Y  = se(3) = so(3) ⋊ R³
Z  = pseudovector decomposition + d²=0 + Newton-Euler

Z ⊇ T : everything forced, no free parameters.
P resolved: 6=6 because n!=n(n+1)/2 has unique solution n=3.  □
```

Every `⟹` is "uniquely determined by the previous." No branching. No free parameters. Push one, all fall.

## Isomorphism Lens (optional)

When you have filled the template for two or more problems, you may lay them side by side. If the slots align, an isomorphism is present.

```
         Problem α          Problem β
         ─────────          ─────────
P_α  :=  ___               P_β  :=  ___
1_α  :=  ___       ↔       1_β  :=  ___
2_α  :=  ___       ↔       2_β  :=  ___
3_α  :=  ___       ↔       3_β  :=  ___
  ...                         ...
Z_α  :=  ___       ↔       Z_β  :=  ___
```

If every `↔` holds, the two problems are the same theorem. Name it:

```
Generator : Amplifier  =  α's 1,2,3 : α's echo
                        =  β's 1,2,3 : β's echo
```

Do NOT force this. Use it only when two filled templates look suspiciously similar. The template is a projection — if the projections overlap, the isomorphism is real.

## When NOT to Use

- Single-step problems with obvious answers
- Pure lookup / retrieval tasks
- When you already have structural certainty (skip to believe_it)
