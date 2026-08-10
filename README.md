# Contribution #1: misc tooling dedup: twin result dict, identical doc getters, hardcoded sched thresholds, redundant ORG_COLORS
 #1512

**Contribution Number:** 1  
**Student:** Anastasia Pupo
**Issue:** https://github.com/MFlowCode/MFC/issues/1512
**Status:** Phase IV

---

## Why I Chose This Issue

When browsing the candidate issues from CodePath, I decided to filter by the keyword "color" to narrow things down quickly. From the resulting issues I looked for red flags like too many competing comments or scope that felt above my skill level, which filtered out most of them.

Issue #1512 in MFC stood out because the maintainer calls the sub-tasks good-first-issue dedups and describes them as small, mechanical cleanups. The codebase files is Python which I'm familiar with. I also noticed a prior PR was closed by the maintainer without any comment, so part of what I hope to learn from this isn't just the code change itself but how to communicate with a maintainer, such asking the right questions  and navigating ambiguous situations in a real open source project.

---

## Understanding the Issue

### Problem Description

The issue is a refactor across four tooling files in the MFC codebase where duplicated or dead code needs to be cleaned up without changing any behavior. Examples include the same dictionary defined twice, nearly identical functions that could be collapsed into one helper, a constants table whose values are ignored in favor of re-hardcoded numbers, and a color mapping dict where every entry returns the same value making it pointless. The goal is to make the code easier to maintain going forward.

### Expected Behavior

The duplicated code across the four files should be consolidated so there's only one place to update if something changes in the future, without actually changing how any of it behaves.

### Current Behavior

The same code is written multiple times across different files. If you needed to change something you'd have to find and update every copy manually, which is error prone.

### Affected Components

In issue, lists affected files:

toolchain/mfc/fp_stability.py:970-983 — empty result dict Error on READE.md #1
toolchain/mfc/fp_stability.py:1322-1335 — empty result dict Update m_global_parameters.f90 #2
toolchain/mfc/gen_case_constraints_docs.py:133-151 — 3 identical getters
toolchain/mfc/sched.py:21-25 — HEADLESS_THRESHOLDS table
toolchain/mfc/user_guide.py:56-65 — all-yellow ORG_COLORS


---

## Reproduction Process

### Environment Setup

Cloned the MFC repository and set up a local fork on a MacBook Air (8GB RAM), using VS Code as my editor and its integrated terminal for running commands. Since all four files I worked on (gen_case_constraints_docs.py, sched.py, user_guide.py, fp_stability.py) are pure Python under toolchain/, no full MFC build (Fortran compilation) was required to reproduce or verify the issue. I could import and run the relevant functions directly with Python. This was a helpful constraint to discover early, since a full ./mfc.sh build would have been considerably more demanding on my machine's limited RAM.

### Steps to Reproduce

1. Cloned MFlowCode/MFC in VS Code and checked out master at the commit the issue was filed against.
2. For each file named in the issue, opened the cited line ranges in VS Code and manually inspected the code to confirm the described duplication existed as written.
3. For gen_case_constraints_docs.py: located get_model_name, get_riemann_solver_name, and get_time_stepper_name at lines 133-151, and compared their bodies. Confirmed they were byte-identical apart from the schema key string, exactly as described. Also located the level to emoji map duplicated at lines 159 and 267.
4. For sched.py: located the HEADLESS_THRESHOLDS table (lines 21-25) and the notify_long_running_threads consumer, and confirmed the seconds were separately hardcoded in the comparisons rather than read from the table.
5. For user_guide.py: located ORG_COLORS (lines 56-65) and confirmed all 8 entries mapped to "yellow", with the lookup also defaulting to "yellow". This meant every org rendered the same color regardless of the dictionary's contents.
6. For fp_stability.py: attempted to locate the described duplicated 13-key result dict at lines 970-983 and 1322-1335. Found the file had since grown to 697 lines, so the cited line numbers no longer corresponded to the described code. The underlying issue this item described appeared to already be resolved elsewhere.

### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/AnaPcode/MFC/tree/master
- **Screenshots/logs:** not applicable. Verification was done by direct code inspection in VS Code, not by running a program that produces visible output or errors.
- **My findings:** Three of the four items, (c), (d), and (e), matched the issue's description exactly. The duplication was real, unambiguous, and located at the cited lines. The fourth item (a) did not reproduce: the file had changed substantially since the issue was filed, and a helper function already present in the code appeared to address the same duplication independently.

---

## Solution Approach

### Analysis

This issue is not a functional bug, but a code quality / maintainability issue: several files in the toolchain contained duplicated logic that should have been written once and reused. The root cause across all items is the same underlying pattern: when a value, dictionary, or formula is needed in more than one place, copying it inline at each call site instead of extracting it into a single shared definition creates a maintenance hazard. If the value ever needs to change, a developer has to remember to update every copy; missing one introduces silent inconsistencies. The maintainer's own framing in the issue (a "grab-bag of small cleanups") confirms this is a refactor, not a bug fix: none of the described behavior is wrong, the code just repeats itself more than it should.

### Proposed Solution

The maintainer has already outlined the fix for each file directly in the issue, assigning each item a letter (a) through (e) along with the specific extraction or dedup expected. My approach will be to implement each of the maintainer's suggested fixes as written, rather than propose an alternative design, since the issue already specifies the intended solution shape for each file.

(a) fp_stability.py: since the cited lines no longer match the file and the issue appears already resolved, I will make no changes to this file and will flag this to the maintainer rather than guess at intent.
(c) gen_case_constraints_docs.py: two separate duplications will be addressed with the maintainer's suggested extractions. The three getter functions with identical bodies (differing only in the schema key they look up) will be collapsed into a single _named(param, value) helper, and the level-to-emoji mapping that is written out twice will be hoisted into one shared module-level constant.
(d) sched.py: a table of named thresholds exists, but the actual comparison logic re-types the same numeric values by hand instead of reading them from that table. The comparisons will be updated to pull their values directly from the table, so the numbers are defined in exactly one place.
(e) user_guide.py: a color-mapping dictionary is entirely redundant, every entry maps to the same value, and the lookup already defaults to that same value. The maintainer's suggested fix offers two options, delete the dictionary or give organizations distinct colors. I will choose to delete it, since assigning meaningful distinct colors is a user-facing design decision outside the scope of a mechanical dedup and I think better suited to its own discussion.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The issue requires four small cleanups across three different files ((c), (d), (e), with (c) containing two separate cleanups), all mechanical and behavior-preserving, suitable for a "good first issue." A fifth item (b) is a larger structural refactor that the maintainer has explicitly asked to be scoped as a separate pull request. A fourth file (a) was also named in the issue, but investigation during reproduction found it already resolved on master, so no changes were made there. This plan covers the three files actually modified: (c), (d), and (e).

**Match:** The codebase already contains examples of the pattern each fix should follow. Helper functions that take parameters and return a shared result are a common pattern throughout the toolchain, matching the shape of the _named(param, value) helper needed for (c). Module-level constants are already used elsewhere in the codebase for shared, unchanging values, which is the same pattern needed to fix the duplicated emoji map in (c).

**Plan:**
1, Create a feature branch to hold changes for the good-first-issue tasks, since the maintainer has requested two separate pull requests: one covering (a), (c), (d), (e), and a second, separate one for (b).
2. Modify gen_case_constraints_docs.py (c) with two separate commits, since this file has two distinct cleanups: first, add a LEVEL_EMOJI module-level constant and update both locations that previously duplicated the emoji map to reference it instead. Second, add a _named(param, value) helper function and update the three getter call sites (model_eqns, riemann_solver, time_stepper) to use it.
3. Modify sched.py (d) in a single commit: update the comparisons in notify_long_running_threads to read threshold values directly from the HEADLESS_THRESHOLDS table instead of re-hardcoding them.
4. Modify user_guide.py (e) in a single commit: remove the redundant ORG_COLORS dictionary and inline the single color value it always resolved to at the lookup site.
5. Verify each change preserves existing behavior by comparing actual output (or underlying values, where output isn't directly comparable) before and after each modification, since these are refactors and no new automated tests are being added for behavior that is not changing.
6. Push the branch and open the pull request covering (c), (d), and (e), noting that (a) required no changes.

**Implement:**
Link to branch: https://github.com/AnaPcode/MFC/tree/feature/tooling-dedup
Link to 4 commits:
Hoist level emoji map to LEVEL_EMOJI constant: https://github.com/MFlowCode/MFC/commit/718ef21fb2bfb9ae20cc460622190fdffabf2e46
Collapse schema-name getters into _named helper: https://github.com/MFlowCode/MFC/commit/212f2de1b68d980f0c029bf2795ba19bb52f769d
Read threshold seconds from HEADLESS_THRESHOLDS: https://github.com/MFlowCode/MFC/commit/1d5406eec4e750d75a6c7198fc8460a8b044eeb4
Delete ORG_COLORS and inline "yellow": https://github.com/AnaPcode/MFC/pull/1/changes/065e86f28280e13c90dc05e2e519541f1be84269

**Review:**
Yes, this work follows the project's contribution guidelines. Commit messages follow the required format: a summary under 50 characters, a blank line, a detailed explanation, and an issue reference. Since these are toolchain/ Python changes, the applicable hard rules are the Python-specific ones rather than the Fortran-specific ones, which don't apply here.
The pull request will follow the same standard: it will group the four cleanups into a single PR, following the guideline of one PR per logical change and matching the issue's own scope recommendation to file (a), (c), (d), (e) together and (b) separately. It will use the required template structure, describe what was tested, and reference the issue as Part of issue 1512.

**Evaluate:**
I will verify each change is behavior-preserving by comparing the program's actual output or values before and after my edits, rather than relying on the full build/test suite, since these changes don't affect simulation or build behavior.

---

## Testing Strategy

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

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
