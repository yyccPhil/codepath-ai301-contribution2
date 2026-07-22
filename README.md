# Contribution #3446: MathTable — First and Last Column Are Truncated

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

*(Tentative — to be confirmed during Phase II reproduction.)*
Likely the `Table` / `MathTable` implementation in manim's mobject layer (probably `manim/mobject/table.py`), specifically the logic that applies `col_widths`, `h_buff`, and `include_outer_lines` when arranging cells. The exact file and function will be pinned down once I reproduce the bug and trace the layout code.

---

## Reproduction Process

_⏳ Phase II — not started yet._

### Environment Setup

[Notes on setting up your local development environment — challenges faced, how you solved them. manim uses Poetry and requires LaTeX + ffmpeg; fill in once set up.]

### Steps to Reproduce

1. [Step 1 — e.g., install manim (current version) in a fresh environment]
2. [Step 2 — run the minimal scene from the issue]
3. [Observed result — first/last columns truncated]

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

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [Confirm whether it still reproduces on the current version, and on which version]

---

## Solution Approach

_⏳ Phase II — not started yet._

### Analysis

[Root-cause analysis — what in the width/buffer calculation causes the outer columns to be clipped?]

### Proposed Solution

[High-level description of the fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Fixed `col_widths` are not honored for the first and last columns of a `MathTable`; they are truncated at the table edges.

**Match:** [What similar patterns/solutions exist in the codebase? e.g., how row heights or inner columns are handled correctly.]

**Plan:**
1. [Modify file X to do Y]
2. [Add / adjust function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist — does it follow CONTRIBUTING.md / the project's style?]

**Evaluate:** [How will you verify it works — rerun the repro scene and compare output?]

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
