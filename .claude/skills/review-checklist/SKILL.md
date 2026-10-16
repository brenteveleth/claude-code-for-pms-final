---
name: review-checklist
description: Reviews a product brief against the PM's six-point checklist before it goes any further. Use when the user says "review this brief", "run the review checklist", "check this brief", or points at a brief file and asks if it is ready. Checks owner, success measure, scope match, problem before fix, sourced data, and definition of ready.
---

# Review checklist

Run the same six checks on any brief. Do not ask the user to explain the checks again.

## Input

- The user gives a file path, a pasted brief, or a wiki page.
- If no brief is named, ask once: "Which brief?" Then stop.
- Read the whole brief before you judge anything.

## The six checks

For each check, give a verdict: **PASS**, **FAIL**, or **UNCLEAR**. Quote the line that proves it. If you cannot quote a line, the verdict is FAIL.

1. **Owner.** The brief names who owns it. A named person or a clear role counts. "The team" does not count.
2. **Success measure.** The brief says how we will know it worked. Look for a metric, a baseline, a target, and a date or window. A metric with no baseline is UNCLEAR.
3. **Scope match.** The scope at the end matches the scope at the start.
   - List what the start promises.
   - List what the end delivers (phases, requirements, asks, appendix).
   - Name anything added, dropped, or renamed between the two.
4. **Problem before fix.** The brief explains the problem before it proposes a fix.
   - Find the first proposed solution.
   - Check that the problem statement comes earlier.
   - Check that the fix addresses that stated problem and not a different one.
5. **Evidence and sources.** Every assertion is backed by data, and the source is written down.
   - List each factual or causal claim (numbers, "since X", "because Y").
   - For each, mark: **sourced** (source named and findable), **unsourced** (no source), or **opinion** (a view, not data).
   - If the source is in the workspace (wiki page, table, file), open it and check the claim. Say if it does not match.
   - Do not fix a gap with a guess. Mark it as a gap.
6. **Definition of ready.** The brief has a "definition of ready": a short list of checkable criteria that show the requirements are met.
   - Each criterion must be testable by someone other than the author.
   - Criteria must cover the requirements in the brief. Name any requirement with no criterion.

## Output

Use this shape, every time:

```
# Review: <brief title>
Result: <n> of 6 pass. <READY | NOT READY | READY WITH FIXES>

| # | Check | Verdict | Evidence (quote or "none found") |
|---|-------|---------|----------------------------------|
| 1 | Owner | ... | ... |
| ... |

## Fix before it goes further
1. <the most important fix, one line>
2. ...

## Claims table (check 5)
| Claim | Status | Source |
|-------|--------|--------|
```

Rules for the result line:

- **READY**: all six PASS.
- **READY WITH FIXES**: no FAIL, but at least one UNCLEAR.
- **NOT READY**: any FAIL.

## Rules

- Review only. Do not rewrite the brief unless the user asks.
- Keep "Fix before it goes further" to 5 items or fewer, most important first.
- Say what you verified and what you did not. Do not call a claim sourced because it sounds right.
- Use the same six checks and the same output shape on every brief. Do not add or drop checks. If you see another problem, add one line under the table, labeled "Also noticed".
- End with one next action the user can do in two minutes.
