# Contribution #2: MathTable — First and Last Column Are Truncated

**Contribution Number:** 2  
**Student:** Yuan Yuan  
**Issue:** https://github.com/ManimCommunity/manim/issues/3446  
**Status:** Phase I — Complete

---

## Why I Chose This Issue

I chose issue #3446, "0.17.3 MathTable - first and last column of MathTable are truncated" in ManimCommunity/manim, because it is a well-scoped, reproducible layout bug that matches my Python skills and my goal of learning how to locate and fix a real bug inside an unfamiliar codebase. The issue is labeled `good first issue`, is unassigned, and ships with minimal reproduction code plus before/after screenshots, so what "fixed" looks like is unambiguous — which makes it a realistic first contribution rather than an open-ended project.

I'm interested in this because:
1. **It's pure Python.** The bug lives in how the `Table` / `MathTable` class computes column widths and buffers, so I can work on it without touching the rendering internals or LaTeX itself.
2. **The scope is contained and testable.** I can run the provided snippet, watch the first and last columns get clipped, fix the width/buffer logic, and confirm the result visually — a clear, bounded piece of work.
3. **There is concrete context to start from** (reproduction code, a stated expected behavior, and comparison images), which should make Phase II faster.
4. **I want to learn** how a mature open-source project structures its layout logic, and how to trace a visual bug back to the arithmetic that causes it.

*Community engagement:* _[To fill in after Step 8 — e.g., "Left a comment on the issue on <date> introducing myself and confirming intent to work on it; asked whether it still reproduces on the latest release."]_

---

## Understanding the Issue

### Problem Description

When a `MathTable` is given fixed column widths (via `arrange_in_grid_config={"col_widths": [...]}`), the widths are not correctly applied to the **first and last columns**. Those two outer columns get truncated at the left and right edges of the table instead of honoring the requested width. Inner columns render as expected.

### Expected Behavior

Every column, including the first and last, should render at the width specified in `col_widths`, independent of the cell contents.

### Current Behavior

The first and last columns are clipped at the table's outer edges, so their contents/width do not match the specified value (see the reproduction screenshots in the issue thread).

### Affected Components

Confirmed during reproduction (manim v0.20.1): the base `Table` class in `manim/mobject/table.py` (inherited by `MathTable`). Specifically:
- `Table._add_vertical_lines()` — the outer vertical lines are anchored to the **content** bounding box of the outer columns, not the column slots:
  - left line: `self.get_columns()[0].get_left()[0] - 0.5 * self.h_buff` (~line 370)
  - right line: `self.get_columns()[-1].get_right()[0] + 0.5 * self.h_buff` (~line 376)
- `Table.get_cell()` — cell corners use the same content-based anchors: `col.get_left()[0] - self.h_buff / 2` and `col.get_right()[0] + self.h_buff / 2` (~lines 791–806).

The column widths themselves are applied earlier via `arrange_in_grid(..., **self.arrange_in_grid_config)` (~line 295), so the cells *are* placed in correctly-sized slots — but the line/cell boundaries are drawn from content extents rather than those slots.

---

## Reproduction Process

Reproduced successfully on **manim v0.20.1** (current release; the issue was originally filed against v0.17.3). The buggy code path in `Table._add_vertical_lines()` is unchanged in the latest source, so the bug is still present.

### Environment Setup

Setup on macOS. The issues I hit and how I solved them:

- **System dependencies:** installed via Homebrew — `brew install py3cairo pango pkg-config scipy ffmpeg` (needed for `pycairo`/`manimpango` and rendering).
- **LaTeX:** MacTeX (`mactex`) is ~6.9 GB; a lighter alternative is BasicTeX. LaTeX is only required to render `MathTable`; I confirmed the bug with a LaTeX-free `Table` variant (`repro_text.py`) that exercises the same layout code.
- **`ERROR: editable mode requires a setuptools-based build`:** caused by an old pip (21.x) that can't do editable installs of pyproject-only projects → fixed by upgrading pip.
- **`Package 'manim' requires a different Python: 3.9.6 not in '>=3.11'`:** the system Python was too old → installed Python 3.12 via `brew install python@3.12`.
- **`error: externally-managed-environment`:** Homebrew blocks installing into its Python system-wide → fixed by creating a virtual environment: `python3.12 -m venv .venv && source .venv/bin/activate`, then `pip install -e .` inside it.
- Result: `import manim` reports **0.20.1** — matching the version I traced the root cause in.

### Steps to Reproduce

1. In a fresh environment with manim installed, create a file `repro.py` with the scene below (the exact snippet from the issue).
2. Render a still image: `manim -s -ql repro.py Test` (the `-s` flag saves the last frame as a PNG; no ffmpeg needed).
3. Open the generated PNG under `media/images/repro/`.
4. **Observed result:** the first and last columns are visibly narrower than the four inner columns — the outer columns are truncated at the table's left/right edges and do not honor `col_widths=[3]*6`.

**Quantified measurement** (from reading the vertical-line x-positions on v0.20.1, table scaled to width 10): inner columns ≈ 1.85 wide and equal, but the **first column is only ~1.00 (≈54% of an inner column)** and the **last column ~1.60 (≈87%)**. The asymmetry matches the root cause: the more narrow a column's *content*, the more that outer column is truncated (the first column holds the narrow `"1"`/`"0"`; the last holds the wide `"100000"`).

> **LaTeX-free variant:** if you don't have LaTeX installed, replace `MathTable` with `Table` and pass the numbers as strings (e.g. `Table([["1","10",...],["0",...]], ...)`). This exercises the *same* `Table` layout code and reproduces the identical truncation, since `MathTable` only overrides how cell contents are built, not the line/cell geometry.

Reference reproduction code from the issue:

```python
class Test(Scene):
    def construct(self):
        mytable = MathTable(
            [[1, 10, 100, 1000, 10000, 100000], [0, 0, 0, 0, 0, 0]],
            h_buff=.1,
            v_buff=.1,
            include_outer_lines=True,
            arrange_in_grid_config={"col_widths": [3] * 6, "col_alignments": ["c"] * 6},
        ).scale_to_fit_width(10)
        self.add(mytable)
```

### Reproduction Evidence

- **Branch in my fork:** https://github.com/yyccPhil/manim/tree/yyccphil-fix-issue-3446 (contains `repro.py`, `repro_text.py`, and `repro_table.png`)
- **Commit showing reproduction:** commit `c3adf52e` on the branch above
- **Screenshots/logs:** `repro_table.png` (rendered on v0.20.1) — first and last columns clearly truncated; committed to the branch above.
- **My findings:**
  - The bug still reproduces on the current release (v0.20.1), not just the reported v0.17.3.
  - Root cause traced to `Table._add_vertical_lines()` and `Table.get_cell()` in `manim/mobject/table.py`: line/cell boundaries are anchored to each column's **content** extent (`get_columns()[i].get_left()/get_right()`) plus half of `h_buff`, instead of to the arranged **column slots** defined by `col_widths`.
  - Because it depends on content width, the truncation is asymmetric: narrow-content outer columns shrink more than wide-content ones.
  - `git log` on `table.py` shows no commit addressing column-width truncation, consistent with the bug still being open.

---

## Solution Approach

### Analysis

When `col_widths` is supplied, `arrange_in_grid` places each cell centered inside a fixed-width slot, so cells whose content is narrower than their slot no longer touch the slot edges. However, the table's grid lines and cell boundaries are computed from the **content** bounding boxes of the columns, not from those slots:

- `_add_vertical_lines()` draws the outer lines at `get_columns()[0].get_left() - 0.5*h_buff` and `get_columns()[-1].get_right() + 0.5*h_buff`.
- `get_cell()` builds cell corners the same way, from `col.get_left()/get_right()` ± `h_buff/2`.

For inner columns this looks roughly correct (inner lines sit between adjacent columns' content), but for the first and last columns the outer line ends up only half a buffer outside the *content* instead of half a slot outside the column — so the outer columns render narrower than `col_widths` requests. The effect scales with how narrow the content is, which explains the asymmetry measured during reproduction.

### Proposed Solution

_(Direction to validate and implement in Phase III — not final.)_

Anchor the outer vertical lines (and, for consistency, `get_cell`'s outer edges) to the **column slot boundaries** rather than the content extent. Concretely, the boundary between two columns should sit midway between their slot centers, and the outer boundaries should extend half a slot-width (not half an `h_buff`) beyond the first/last column centers. The slot geometry is already known at arrange time (from `col_widths` + `h_buff`), so the fix is to derive line/cell x-positions from that arranged column pitch instead of from `get_columns()[i].get_left()/get_right()`. The default case (no `col_widths`, content fills the slot) must remain visually unchanged.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Fixed `col_widths` are not honored for the first and last columns of a `Table`/`MathTable`; the outer columns are truncated because grid/cell boundaries are drawn from column *content* extents rather than the column *slots*.

**Match:** Compare how inner vs. outer lines are anchored in `_add_vertical_lines()`; check how row heights / `v_buff` are handled for the analogous vertical case; look for any existing helper that exposes per-column slot positions from the `arrange_in_grid` result.

**Plan:**
1. In `manim/mobject/table.py`, change the outer-line anchors in `_add_vertical_lines()` so the first/last boundaries reflect the column slot width, not `get_columns()[0/-1].get_left()/get_right() ± 0.5*h_buff`.
2. Apply the same correction to `get_cell()` so a cell's polygon matches its rendered column.
3. Verify the default path (no `col_widths`) is unchanged.
4. Add/extend tests (see Testing Strategy) and update docstrings/examples if needed.

**Implement:** [Link to your feature branch/commits during Phase III.]

**Review:** Follow `CONTRIBUTING.md` — run the project's linters/formatters and the existing `tests/` for tables; confirm no visual regression in existing table examples.

**Evaluate:** Re-render the reproduction scene and re-measure the vertical-line x-positions; all six columns should be equal width (≈1.67 at width 10) with `col_widths=[3]*6`, and unchanged output when `col_widths` is not set.

---

## Testing Strategy

_⏳ Phase III — not started yet._

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

_⏳ Phase II–III — not started yet._

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

_⏳ Phase IV — not started yet._

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description — much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

_⏳ To be completed as the contribution progresses._

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- manim community docs: https://docs.manim.community/
- manim CONTRIBUTING guide: https://docs.manim.community/en/stable/contributing.html
- The issue thread: https://github.com/ManimCommunity/manim/issues/3446
- [Add tutorials, Stack Overflow posts, or related issues/discussions as you use them]
