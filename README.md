# Contribution #1: misc tooling dedup: twin result dict, identical doc getters, hardcoded sched thresholds, redundant ORG_COLORS
 #1512

**Contribution Number:** 1  
**Student:** Anastasia Pupo
**Issue:** https://github.com/MFlowCode/MFC/issues/1512
**Status:** Phase II

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

[Notes on setting up your local development environment - challenges you faced, how you solved them]
Utilized instructions from https://mflowcode.github.io/documentation/getting-started.html. Ran the program on a MacBook Air with operating system macOS 26.3.
First, cloned the repository for local development. The utilized homebrew to install the packages.
Challeges: While installing packages with homebrew, there was a error with qt. Will monitor if the issue affects any part of the program.
<img width="909" height="717" alt="Screenshot 2026-06-14 at 11 59 18 PM" src="https://github.com/user-attachments/assets/62409c38-2816-44a3-87fb-5b0d70c8f014" />


Error when building MFC:
<img width="1071" height="109" alt="Screenshot 2026-06-15 at 12 19 39 AM" src="https://github.com/user-attachments/assets/8c35a8db-dc97-4406-9808-e9cf671c7796" />


### Steps to Reproduce

1. Run MFC

The issue is a refactor so need to run program to view how program currently runs.

### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/AnaPcode/MFC/tree/master
- **Screenshots/logs:** not applicable
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]
This issue is a refactor.

### Proposed Solution

[High-level description of your fix approach]
Maintener has outlined what needs to be refactored.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

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
