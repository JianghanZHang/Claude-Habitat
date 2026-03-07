---
name: seeit
description: Draw and fix TikZ diagrams by treating them as grid-routing (PacMan) problems. Use when creating, editing, or debugging any TikZ figure — especially block diagrams, flow charts, feedback loops, or any diagram with connecting lines. Also use when the user says lines are "crossing", "broken", "overlapping", or asks to "align" or "route" paths. Trigger on any TikZ/diagram layout task.
---

# SeeIt: TikZ as PacMan

## The Lesson

The goal of drawing a figure is not to write code that compiles — it's to **know what the output will look like before you compile**. TikZ path operators (`|-`, `-|`) compile fine but draw wrong because you can't predict the visual result by reading the code. A coordinate like `(A -| B)` forces you to mentally resolve which x and which y, and you will get it wrong in multi-segment paths.

The fix: make every point in the diagram a named location on a grid. When you read `(P1) -- (P2) -- (P3) -- (P4)`, you can point to each spot on the map and trace the path with your finger. If you can't do that, you don't know what you're drawing — and "compiles clean" means nothing.

## The Method

### 1. Place boxes on a grid

Define every node position with an explicit coordinate. Name the grid anchors.

```latex
% Grid anchors — the map
\coordinate (R1) at (0, 0);
\coordinate (R2) at (1.5, -2.0);
\coordinate (R3) at (3.0, -4.0);

\node[box] at (R1) (A) {Node A};
\node[box] at (R2) (B) {Node B};
\node[box] at (R3) (C) {Node C};
```

Staircase layouts (each row offset right) create natural lanes for feedback paths. The diagonal gap is your PacMan corridor.

### 2. Route lines as named waypoints

Every line is a sequence of named points. Only horizontal and vertical segments. No diagonals. No `|-` or `-|` in multi-segment paths.

```latex
% Name every turn in the path
\coordinate (W1) at ([yshift=-0.5cm]C.south);     % below C
\coordinate (W2) at ([xshift=-1cm]A.west |- W1);  % left margin, same y
\coordinate (W3) at ([yshift=1.5cm]W2);            % up at left margin
\coordinate (W4) at (W3 -| A.south);               % below A, same y as W3

% Draw: each segment is one direction only
\draw[dashed] (C.south) -- (W1) -- (W2) -- (W3) -- (W4) -- (A.south);
```

### 3. The PacMan rules

| Rule | Why |
|------|-----|
| Name every waypoint | You can debug by checking coordinates |
| One direction per segment | Horizontal OR vertical, never diagonal |
| `(A |- B)` = (A.x, B.y) | x from first, y from second |
| `(A -| B)` = (B.x, A.y) | x from second, y from first |
| Only use `|-`/`-|` to DEFINE coordinates, not in `\draw` paths | Avoids the swap bug |
| Enter boxes from the correct side | If approaching from below, target `node.south`; from left, `node.west` |
| Feedback paths go on the OUTSIDE | Route around, not through |

### 4. Avoiding crossings

Think of the diagram as a map with walls (the boxes) and corridors (the gaps between rows/columns).

**Forward flow**: left-to-right within rows, top-to-bottom between rows. These are the main streets.

**Feedback flow**: must use side streets — the margins and inter-row gaps.

Three routing strategies for feedback:
- **Left margin**: go down, left to outside all boxes, up along left edge, right back in
- **Right margin**: go right past all boxes, up along right edge, left back in
- **Top/bottom bypass**: go above or below a row to avoid crossing inline labels

When a line must cross multiple rows, create a **staircase feedback path** that mirrors the layout:

```
down → left → up (step 1) → right → up (step 2) → into target
```

### 5. Debugging checklist

When a line looks wrong:
1. Print the waypoint coordinates: add `\fill[red] (W1) circle(2pt);` at each waypoint
2. Check that consecutive waypoints share exactly one coordinate (x or y) — if they don't, you have a diagonal
3. Check entry direction matches anchor: vertical approach → `.south`/`.north`, horizontal → `.east`/`.west`
4. Check `|-` vs `-|` haven't been swapped (this is the #1 bug)

### 6. Operator reference card

```
(A |- B)  →  (A.x, B.y)    "A's column, B's row"
(A -| B)  →  (B.x, A.y)    "B's column, A's row"

\draw (A) |- (B);   →  vertical first, then horizontal
\draw (A) -| (B);   →  horizontal first, then vertical

[yshift=1cm]P   →  (P.x, P.y + 1)   move UP
[xshift=1cm]P   →  (P.x + 1, P.y)   move RIGHT
```

The `-|`/`|-` operators are FINE for two-point connections (`\draw (A) -| (B);`). They are DANGEROUS in multi-segment paths because you lose track of which coordinate is which. For anything with more than one turn: use named waypoints.

---

## Flow Diagrams — the simple case

Most diagrams are flow diagrams. They have exactly three ingredients:

1. **Layers** — horizontal rows, stacked top to bottom
2. **Blocks** — boxes within each layer, arranged left to right
3. **Connections** — arrows between blocks (forward = down/right, feedback = up/left)

That's it. If you can specify these three things, you can draw any flow diagram. The recipe:

### Step 1: Declare the layers

Each layer is a y-coordinate. Number them top to bottom.

```latex
% Layer positions (top to bottom)
\def\layerA{0}       % perception
\def\layerB{-2.0}    % planning
\def\layerC{-4.0}    % execution
```

### Step 2: Place blocks in each layer

Each block gets an (x, y) from its layer. Space them evenly within the layer.

```latex
% Layer A: perception
\node[box] at (0, \layerA)   (cam)  {Camera};
\node[box] at (3, \layerA)   (enc)  {Encoder};
\node[box] at (6, \layerA)   (det)  {Detector};

% Layer B: planning
\node[box] at (1.5, \layerB) (plan) {Planner};
\node[box] at (4.5, \layerB) (opt)  {Optimizer};

% Layer C: execution
\node[box] at (3, \layerC)   (ctrl) {Controller};
```

Now you have a mental picture: 3 boxes across the top, 2 in the middle, 1 at the bottom. You can see it.

### Step 3: Connect

**Forward connections** (same layer or downward) are trivial — straight lines or single-turn L-shapes:

```latex
% Within layer: horizontal
\draw[->] (cam) -- (enc);
\draw[->] (enc) -- (det);

% Between layers: vertical or single-turn
\draw[->] (enc.south) -- (plan.north);  % straight down
\draw[->] (det.south) -| (opt.north);   % L-shape, one turn — safe
```

**Feedback connections** (upward or leftward) need waypoints — use the PacMan method from above.

### Key insight

For forward flow, you barely need the PacMan rules — straight lines and single `-|` turns are enough. The complexity only appears in feedback paths, and those are rare. Most flow diagrams are 90% forward connections. Don't over-engineer the simple parts.

### Staircase variant

When layers have different widths or feedback paths cross many layers, offset each layer rightward:

```latex
\def\stepR{1.0}  % rightward offset per layer
% Layer A at x=0, Layer B at x=1.0, Layer C at x=2.0
```

This opens diagonal corridors on the left side — free lanes for feedback paths that never cross forward-flow arrows.

### Text labels are walls

Edge labels (`$q, \dot{q}$`, `$\phi$`, etc.) occupy space on the diagram just like boxes do. A line that passes through a text label looks broken — the reader can't tell if the line connects to the label or passes behind it.

Treat every text label as a invisible block with a bounding box. Route lines around them, not through them. If a label sits on an arrow between two layers, feedback paths must go *above* or *below* that label's row, never across it.
