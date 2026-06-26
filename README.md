# 4D Cube Rotation Playground & Match Game

**Status:** active sketch. Working 4D viz + diagnostic projections + match game with leveled curriculum.

## What this is

A web tool for **building 4D-rotation intuition** by interacting with a rotating tesseract. The premise: humans recover 3D shape from 2D images by predicting how motion will continue. Maybe the same prediction machinery can learn 4D shape if you let people drive the rotation themselves and watch what happens. The tool gives the user:

- Direct control over which 2-plane the cube rotates in (any of `C(4,2) = 6` planes, or simultaneous Clifford pairs)
- A *generic* 2D projection of the whole 4D cube ("main view")
- Six *coordinate-plane* projections that each pick out 2 of the 4 axes (the "tiles") — diagnostic windows that freeze under some rotations and move under others
- Depth-based color coding so the kernel of the projection (the 2 dimensions you can't see directly) still becomes visible
- A match-the-target game with a curriculum that progressively removes the scaffolding

## Layout

```
~/Projects/Ideas/4DCubeVizGame/
├── README.md                    ← this file
├── 4d_cube.html                 ← THE TOOL. Open it in a browser. Self-contained.
├── 3d_tetrahedron_webgl.html    ← stepping-stone, never woven in
└── key_press_test.html          ← input scaffold from the early 2023 iter sessions
```

Everything that matters is in `4d_cube.html`. ~983 lines, no build step, no dependencies. Open via `python3 -m http.server` (or any static server) and point a browser at it. (Don't use `file://` — the Claude-in-Chrome MCP rewrites it.)

## The interface — a tour

The page is two columns:

### Left column

1. **Main view** (320 × 320 SVG) — the tesseract under a *generic* `2 × 4` projection that mixes all 4 axes. Captures every kind of rotation visibly, but loses 2 dimensions in a way that's hard to reason about analytically.
2. **Target view** (shown only when the match game is enabled) — same projection, applied to the target cube the user is trying to align with. Dashed border.
3. **Depth legend** — a horizontal `hsl(240,…) → hsl(0,…)` gradient bar with tick marks at `0`, `√2 ≈ 1.41` (the depth every axis-aligned vertex has at rest), and `2` (max). Tells you what the line colors mean.
4. **6 coordinate-plane tiles** in a 1 × 6 strip — labeled `(0,1), (0,2), (0,3), (1,2), (1,3), (2,3)`. Tile `(i,j)` projects directly onto coordinates `i` and `j`, ignoring the other two. Diagnostic properties:
   - Rotation in plane `(i,j)`: tile `(i,j)` moves, tile `(k,l)` (the disjoint partner) is **completely frozen** (both position and color), the four mixed tiles change partially.
   - Tiles are themselves clickable — click a tile to select its plane.

### Right column (panel)

1. **State card** — text describing the current rotation: e.g. `+(0,1) ⊕ −(2,3)  →  Clifford [right-isoclinic]`.
2. **Axis-pair selector** — a drawing of `K₄` (the complete graph on 4 nodes labeled 0–3). The 6 edges of K₄ map bijectively onto the 6 elements of `𝔰𝔬(4)`'s basis. Click an edge to select that rotation plane. Two disjoint edges = a Clifford pair (perfect matchings of K₄ are exactly the 3 Clifford pairings). Plus a speed slider, default 1.5°/frame.
3. **User rotation matrix** — live 4 × 4 `R_user` rendered as a color-tinted grid. Blue tint = positive entry, red tint = negative, opacity = magnitude.
4. **Match game card** — see below.
5. **Command box** — text-driven entry: `01 right`, `0213 left`, `stop`, `reset`. Useful for scripting / repeat-experiments.

## Input model

The user has **two state surfaces**:

- **Persistent selection** — set via mouse (K₄ edge or tile click). Stays put. Carries signs for each plane (positive = solid line, negative = dashed). Stored as `selOrder` (axes in click order) + `edgeSigns`.
- **Momentary override** — set via held digit keys `0`–`3` (or backtick `` ` `` which also maps to 0 for keyboards where it's positioned there). Overrides the persistent selection while held, snaps back when released. Held nodes glow yellow in the K₄ graph in real time.

Direction comes from holding arrow keys `←` / `→`. Tab cycles linearly through the 6 planes. Esc clears the persistent selection. Sign negation is via Shift (mouse) or the `-` key (held during any select-or-hold action).

Why split persistent + momentary? Because in two-handed play the natural ergonomics is: set up a complex (possibly multi-plane, mixed-sign) configuration with the mouse and then "sample" different planes by holding digit keys with your other hand. The persistent state stays as scaffolding while the override lets you ask "what if?"

## The math behind the diagnostic

For each projection `P : ℝ⁴ → ℝ²` and vertex `v`:

- **2D position** is `Pv` (drawn at scaled `+ offset`).
- **Depth** is `√(|v|² − |Pv|²)` — the distance from `v` to the 2D viewing plane in 4-space. Encoded as line color via `hsl(240 · (1 − depth/2), 85%, 48%)`. Per-edge linear gradients interpolate color from one endpoint to the other.

A rotation in plane `(i,j)` preserves any quantity built from coords other than `i, j`. For coord-plane tile `(i,j)`, depth = `√(v[k]² + v[l]²)` where `{k,l} = {0,1,2,3} \ {i,j}`. So depth in tile `(i,j)` is preserved exactly under rotations in `(i,j)` OR `(k,l)` — the same diagnostic invariant as position, generalized to color. The four mixed tiles see depth shift continuously during any rotation that touches their axes.

For the generic main view, depth is not a coord-plane sum but `|v|² − |Pv|²`. Still preserved under rotations in the 2-plane orthogonal to `P`'s row span (the "blind spot" — see *Known limits* below).

## Match game with leveled curriculum

A 7-level progression that gradually removes the diagnostic crutches. Enable via the "match game" toggle.

| # | Kind | Tiles | Matrix |
|---|------|-------|--------|
| 1 | single plane | ✓ | ✓ |
| 2 | single plane | ✓ | ✗ |
| 3 | single plane | ✗ | ✗ |
| 4 | Clifford     | ✓ | ✓ |
| 5 | Clifford     | ✓ | ✗ |
| 6 | Clifford     | ✗ | ✗ |
| 7 | generic SO(4)| ✓ | ✓ |

Each level: hit 3 matches to auto-advance. Match condition: Frobenius distance `‖R_user − R_target‖ < 0.10`. Bonus stats per attempt: par (min degrees needed), your degrees, efficiency (`par / your`) with `★`, `★★`, `★★★` tiers at 25 / 50 / 80%.

## Known limits

- **The 2D viewing plane has a 2-dimensional blind spot.** Rotations entirely in the plane orthogonal to the main projection's row span are invisible — same position, same depth color. This is mathematically unavoidable for a single 2D projection. The coord-plane tiles cover the gap (any 2-plane rotation touches at least one tile non-trivially); the matrix display shows the rotation analytically. On no-tiles, no-matrix levels (3, 6), you fall back to probe-and-feedback against the distance bar.
- **Par for generic targets is approximate.** Single-plane par = the rotation angle (achievable); Clifford par = sum of two angles (achievable). Generic par is `0.55 × sum-of-component-angles`, which is a heuristic. Real par would need the SO(4) canonical-form decomposition.
- **No N-dim generalization yet.** Hardcoded for 4D throughout (16 vertices, 32 edges, 6 planes, K₄ graph). Generalizing would require: dynamic vertex/edge generation, a Stiefel-manifold projection chooser, and `K_n` instead of `K_4` for plane selection.

## Possible next moves

- **Peek mode** — briefly flash the tiles or matrix at the cost of a small score penalty, on demand.
- **Identify mode** — replace the match game's user-controlled solver with "name the rotation": show the rotated target alongside an axis-aligned cube, ask the user to click the K₄ edge that produced it. Pure deduction.
- **5D, 6D, …** — generalize. Beyond 4D, K_n has `n(n−1)/2` edges; the tile strip grows quadratically; the K₄ graph needs a different visual layout.
- **VR / WebXR** — render the tesseract into a stereo pair with the user's head pose as the third (depth-direction) constraint. Then the main view actually has access to 3 of the 6 𝔰𝔬(4) directions visibly.
- **Tetrahedron stepping-stone** — `3d_tetrahedron_webgl.html` is still sitting there from the 2023 iter sessions. If we ever go WebGL for higher-N, that's the starting point.

## Gotchas

- **Vimium (and similar extensions) swallow digit keys.** If keybindings for selecting axes (0–3 etc.) aren't working, check for browser extensions that intercept keyboard shortcuts (e.g. Vimium, Vimium C, Surfingkeys, or other vim/emacs-style extensions). Disable them for this domain or pause the extension on the page. Mouse clicks and arrow keys should continue to work as a fallback.
- The in-game Help panel (click "help & math" or press H / ?) also reminds users of this.
- **Don't open via `file://`** — at least when using the Claude-in-Chrome MCP, the URL bar's protocol-rewrite breaks the load. Serve via `python3 -m http.server 8765` from this directory.

## Why "Ideas" and not "Active"

It's a tab that's gotten substantial code but still doesn't have a *use case* beyond personal exploration. If the no-tile levels turn out to actually build 4D intuition that transfers (e.g., to thinking about quaternions, or to other geometric/physics contexts), it could justify a polish pass and a public web home. For now it lives here as a thing to come back to.

— *README last updated 2026-05-17, after adding leveled curriculum, K₄ graph + tile selectors, per-edge sign control, depth-color gradients, depth legend, Tab cycling, and momentary override.*
