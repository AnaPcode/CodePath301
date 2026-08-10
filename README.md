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

No new automated unit tests were added. These changes are refactors with no new behavior, so rather than writing new unit test functions, correctness was verified through direct manual comparison of function output before and after each change (detailed under Manual Testing below).

### Integration Tests

Not applicable. These changes are isolated to individual functions within three toolchain/ Python files, with no interaction between subsystems (e.g., no build-system, Fortran, or cross-module integration to test).

### Manual Testing

gen_case_constraints_docs.py: ran the documentation-generation entry point against both the original (master) and modified code, using a git worktree to compare both versions side by side. Compared the two outputs with diff and confirmed they were identical; also confirmed with matching file hashes.

user_guide.py: ran the cluster-content generation function against both versions using the same approach. Outputs were identical, and all organizations still rendered the same color as before.

sched.py: since this function doesn't produce directly comparable output, verified correctness by value instead: confirmed the threshold values read from HEADLESS_THRESHOLDS (120, 600, 1800 seconds) matched the hardcoded values they replaced.

All three checks confirmed the refactors preserved existing behavior exactly, with no differences in output or underlying values.

<details>
<summary>Commands used for testing</summary>

​```bash
cd ~/MFC
mkdir -p ~/mfc-verify
git worktree add /tmp/original_master master
​```

**1. gen_case_constraints_docs.py**

​```bash
cd /tmp/original_master && python3 -c 'import sys; sys.path.insert(0,"toolchain")
from mfc.gen_case_constraints_docs import main; main()' > ~/mfc-verify/docs_before.txt

cd ~/MFC && python3 -c 'import sys; sys.path.insert(0,"toolchain")
from mfc.gen_case_constraints_docs import main; main()' > ~/mfc-verify/docs_after.txt

diff ~/mfc-verify/docs_before.txt ~/mfc-verify/docs_after.txt && echo "gen_case_constraints_docs.py: NO CHANGE"
​```

**2. user_guide.py**

​```bash
cd /tmp/original_master && python3 -c 'import sys; sys.path.insert(0,"toolchain")
from mfc.user_guide import _generate_clusters_content as f; print(f())' > ~/mfc-verify/guide_before.txt

cd ~/MFC && python3 -c 'import sys; sys.path.insert(0,"toolchain")
from mfc.user_guide import _generate_clusters_content as f; print(f())' > ~/mfc-verify/guide_after.txt

diff ~/mfc-verify/guide_before.txt ~/mfc-verify/guide_after.txt && echo "user_guide.py: NO CHANGE"
​```

**3. sched.py** — no output to compare, so check the numbers directly

​```bash
cd ~/MFC && python3 -c 'import sys; sys.path.insert(0,"toolchain")
from mfc.sched import HEADLESS_THRESHOLDS as H
print(H[0][0], H[1][0], H[2][0], "should be", 2*60, 10*60, 30*60)'
​```
Result:
​```
120 600 1800 should be 120 600 1800
​```

**Proof (hashes)**

​```bash
shasum ~/mfc-verify/*.txt
​```
Result:
​```
3016ca54d8cafc7fed2e41f49ac3fd4dd2cababd  ~/mfc-verify/docs_after.txt
3016ca54d8cafc7fed2e41f49ac3fd4dd2cababd  ~/mfc-verify/docs_before.txt
a22ed36dc99d289c79f9c9dae2486fe2b5a5f5d6  ~/mfc-verify/guide_after.txt
a22ed36dc99d289c79f9c9dae2486fe2b5a5f5d6  ~/mfc-verify/guide_before.txt
​```

**Cleanup**

​```bash
cd ~/MFC && git worktree remove /tmp/original_master
​```

</details>

Command Line Image:
<img width="836" height="554" alt="Screenshot 2026-08-09 at 9 23 20 PM" src="https://github.com/user-attachments/assets/d1e7f8f2-5c52-4e08-ba8d-f1ab91dc1e77" />

---

## Implementation Notes

### Week 10 Progress

Completed the full scope of this issue (items (c), (d), and (e); item (a) required investigation but no code changes). Work included locating each described duplication by inspecting the cited code, implementing the maintainer's suggested fix for each file, verifying behavior preservation for each change, and writing up commit messages and a pull request description.

A challenge was fp_stability.py (item (a)): the line numbers cited in the issue no longer matched the file, since the file changed significantly since the issue was filed. This required tracing the file's history to see if the described duplication had already been resolved elsewhere. I decided to make no changes to this file and to flag the finding to the maintainer.

A second decision point was item (e) (user_guide.py): the issue offered two valid fixes, deleting the redundant dictionary or assigning distinct colors to each organization. I chose deletion, since introducing new distinct colors is a user-facing design decision I wasn't in a position to make as a first-time contributor without more context on what those colors should represent.

I also worked out a verification approach for changes with no existing automated test coverage: using a git worktree to run the pre-change version of the code side by side with my working copy, so I could directly diff real output before and after each change rather than relying on assumptions.

### Code Changes

- **Files modified:**
  toolchain/mfc/gen_case_constraints_docs.py
  toolchain/mfc/sched.py
  toolchain/mfc/user_guide.py
- **Key commits:**
  Hoist level emoji map to LEVEL_EMOJI constant: https://github.com/MFlowCode/MFC/commit/718ef21fb2bfb9ae20cc460622190fdffabf2e46
  Collapse schema-name getters into _named helper: https://github.com/MFlowCode/MFC/commit/212f2de1b68d980f0c029bf2795ba19bb52f769d
  Read threshold seconds from HEADLESS_THRESHOLDS: https://github.com/MFlowCode/MFC/commit/1d5406eec4e750d75a6c7198fc8460a8b044eeb4
  Delete ORG_COLORS and inline "yellow": https://github.com/AnaPcode/MFC/pull/1/changes/065e86f28280e13c90dc05e2e519541f1be84269
- **Approach decisions:**
  Followed the maintainer's suggested fix for each file as written, rather than proposing alternative designs, since the issue already specified the intended solution shape for each item.
  Split gen_case_constraints_docs.py into two separate commits (one per distinct duplication) rather than one combined commit, since the emoji-map fix and the getter-collapse fix are logically independent and easier to review separately.
  Chose not to implement distinct organization colors for item (e), deferring that as a possible future feature request rather than a mechanical dedup.
  Chose not to run the full Fortran build/test suite, since none of the modified files affect build or simulation behavior; verified correctness directly through output comparison instead.

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:**

## Description

Addresses items (a), (c), (d), (e) from issue 1512, following the specific fixes the issue suggested for each file except for (a).

(a) fp_stability.py - no changes made. The file seems to have changed since the issue was filed (now 697 lines), and the cited line numbers (970-983, 1322-1335) no longer correspond to the described code. I found a _blank_result(name, ...) -> dict helper already present, which may have resolved this.

(c) gen_case_constraints_docs.py - two separate dedups:

The level emoji map was duplicated in render_playbook_card (line 159) and generate_playbook (line 267). Moved it to a LEVEL_EMOJI constant, placed after PlaybookEntry, and pointed both call sites at it.

get_model_name, get_riemann_solver_name, and get_time_stepper_name (lines 133-151) had byte-identical bodies modulo the schema key string. Replaced all three with a single _named(param, value) helper and updated the three call sites: line 170 (model_eqns), line 222 (riemann_solver), and line 226 (time_stepper).

(d) sched.py - the seconds were hardcoded at each comparison in notify_long_running_threads, while only the message text indexed the HEADLESS_THRESHOLDS table. Comparisons now read the seconds from the table too.

(e) user_guide.py - all 8 entries in ORG_COLORS mapped to "yellow", and the lookup defaulted to "yellow" anyway, so every org rendered yellow regardless. Deleted the dictionary and inlined "yellow" directly at the lookup site. Chose not to give orgs distinct colors, since that's a user-facing change and I could not identify what information/benefit color-coding would add across the 9 orgs. May be worth its own feature-request issue if that's actually wanted.

All changes are mechanical and behavior-preserving, no functional or output changes are intended.


### Type of change

- Refactor


## Testing

These changes are cleanups in the toolchain Python files, with no build-system or Fortran impact, so I verified behavior preservation directly rather than through the test suite. For two of the files, gen_case_constraints_docs.py and user_guide.py, I compared generated output before and after my changes. To do this I created a temporary copy of the original master branch alongside my working copy, so I could run both versions side by side. sched.py is time-dependent, so I verified it by value instead.

gen_case_constraints_docs.py - I ran main(), the entry point that generates the case constraint documentation, and compared outputs from both the original file and the file with changes. The outputs were identical. That path covers both changes in this file: it calls render_playbook_card(), which uses _named, and both it and generate_playbook() use LEVEL_EMOJI.

user_guide.py - same comparison on _generate_clusters_content(). Outputs were identical, and all orgs still render yellow.

sched.py - prints only after minutes of real elapsed time, so I compared values rather than output. The table holds 120, 600, and 1800 seconds, the same numbers as the 2 * 60, 10 * 60, 30 * 60 literals they replaced.

I also used file hashes (a way to confirm two files are byte-for-byte identical) as an extra check on top of the direct comparisons for gen_case_constraints_docs.py and user_guide.py where both matched.

<details>
<summary>Commands used for testing</summary>

​```bash
cd ~/MFC
mkdir -p ~/mfc-verify
git worktree add /tmp/original_master master
​```

**1. gen_case_constraints_docs.py**

​```bash
cd /tmp/original_master && python3 -c 'import sys; sys.path.insert(0,"toolchain")
from mfc.gen_case_constraints_docs import main; main()' > ~/mfc-verify/docs_before.txt

cd ~/MFC && python3 -c 'import sys; sys.path.insert(0,"toolchain")
from mfc.gen_case_constraints_docs import main; main()' > ~/mfc-verify/docs_after.txt

diff ~/mfc-verify/docs_before.txt ~/mfc-verify/docs_after.txt && echo "gen_case_constraints_docs.py: NO CHANGE"
​```

**2. user_guide.py**

​```bash
cd /tmp/original_master && python3 -c 'import sys; sys.path.insert(0,"toolchain")
from mfc.user_guide import _generate_clusters_content as f; print(f())' > ~/mfc-verify/guide_before.txt

cd ~/MFC && python3 -c 'import sys; sys.path.insert(0,"toolchain")
from mfc.user_guide import _generate_clusters_content as f; print(f())' > ~/mfc-verify/guide_after.txt

diff ~/mfc-verify/guide_before.txt ~/mfc-verify/guide_after.txt && echo "user_guide.py: NO CHANGE"
​```

**3. sched.py** — no output to compare, so check the numbers directly

​```bash
cd ~/MFC && python3 -c 'import sys; sys.path.insert(0,"toolchain")
from mfc.sched import HEADLESS_THRESHOLDS as H
print(H[0][0], H[1][0], H[2][0], "should be", 2*60, 10*60, 30*60)'
​```
Result:
​```
120 600 1800 should be 120 600 1800
​```

**Proof (hashes)**

​```bash
shasum ~/mfc-verify/*.txt
​```
Result:
​```
3016ca54d8cafc7fed2e41f49ac3fd4dd2cababd  ~/mfc-verify/docs_after.txt
3016ca54d8cafc7fed2e41f49ac3fd4dd2cababd  ~/mfc-verify/docs_before.txt
a22ed36dc99d289c79f9c9dae2486fe2b5a5f5d6  ~/mfc-verify/guide_after.txt
a22ed36dc99d289c79f9c9dae2486fe2b5a5f5d6  ~/mfc-verify/guide_before.txt
​```

**Cleanup**

​```bash
cd ~/MFC && git worktree remove /tmp/original_master
​```

</details>


Part of issue 1512


**Maintainer Feedback:**
Maintainer has not yet responded.

**Status:** Awaiting Review

---

## Learnings & Reflections

### Technical Skills Gained

The main new technical skill was learning to use git worktree to check out a second copy of a branch (in my case, the original master) alongside my working copy, without disturbing my active changes. This let me run the pre-change and post-change versions of the same function side by side and directly diff their output, which became my main strategy for verifying that a refactor didn't change behavior when no existing automated tests covered the code.

This issue also taught me good contribution practices I hadn't done deliberately before: splitting work into separate, logically independent commits rather than one large commit, and writing commit messages that actually follow a project's contribution guidelines (concise imperative summary, blank line, detailed explanation, issue reference). Working through this also meant making real decisions rather than just following instructions exactly, such as how to scope commits, how much detail to put in a PR description, and how to phrase testing claims accurately rather than overstating what I'd actually verified.

### Challenges Overcome

Working with limited RAM (8GB, MacBook Air) meant I had to think carefully about what setup was actually necessary before diving in. Since the full MFC build compiles Fortran across multiple configurations, I had to figure out whether I needed to build the project at all for this issue, and realized that since all four affected files were pure Python under toolchain/, I could verify my changes by running the relevant functions directly, without a full build.

Working in a new, large codebase without physics domain knowledge also slowed me down. Understanding what a file or function was actually responsible for, especially in files adjacent to physics/simulation logic, sometimes took longer than making the actual code change, since I had to be careful not to assume behavior I didn't fully understand.

Deciding how thoroughly to test was also harder than expected. Some of these refactors felt intuitively safe, for example, reading a value from a dictionary instead of hardcoding it seems like it obviously shouldn't change behavior, so I kept second-guessing whether setting up a full before/after comparison for each file was overkill. In the end, I still went through with the more rigorous verification, and it turned out to be a good learning opportunity: it forced me to actually prove behavior was unchanged rather than relying on intuition, and gave me a reusable process (the git worktree comparison) I can apply to future refactors where "this should obviously be fine" isn't good enough justification on its own.

### What I'd Do Differently Next Time

I'd try to finish an issue in a shorter overall timeframe when possible, to reduce the risk of the target files changing. This is exactly what happened with item (a) (fp_stability.py): the file matched the issue's description when I first looked at it, and I had already worked out a solution locally, but I waited too long before implementing it. By the time I came back, the file had changed enough that my proposed solution for that file no longer applied.

---

## Resources Used

MFC Contributing Guide - covered commit message format, coding standards, testing expectations, CI pipeline structure, and PR submission process

MFC Getting Started Guide - covered build environment setup, running the test suite, and toolchain usage

Issue #1512 - the original issue describing all five tooling dedup tasks, including the maintainer's specific suggested fix for each file

Claude (subscription provided by CodePath) - used throughout this project for help understanding the codebase, code suggestions, testing strategy, commit messages, the pull request description, and this read.me
