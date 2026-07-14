# Contribution #4098: Add LinearSystemLoop.atReference() function

**Contribution Number:** 4098  
**Student:** Kashish Saini
**Issue:** [GitHub issue link](https://github.com/wpilibsuite/allwpilib/issues/4098)
**Status:** Phase II In Completed

---

## Why I Chose This Issue

I chose this issue because it's a small, well-bounded feature addition rather than an open-ended refactor — LinearSystemLoop exposes a getError() method but has no built-in way to check whether the system is "at reference," unlike other WPILib controllers. Before starting implementation, I verified the issue's own premise against the current codebase rather than taking it at face value: the issue states LinearSystemLoop "takes a vector of acceptable error values," implying a tolerance already exists — but as of current main, no such field exists anywhere in the class (Java, C++, or the Python binding config). So this isn't wiring together two existing pieces; it requires adding a new tolerance field and setter, then the comparison method, in three language surfaces.

I also had to update my template list: the issue and prior research pointed to HolonomicDriveController.atReference() and RamseteController.atReference() as precedent, but both classes were removed in WPILib's 2027 refactor (called out explicitly in CONTRIBUTING.md as intentionally-removed "opaque black-box" abstractions). The patterns that do still exist and are usable as templates are PIDController.atSetpoint() (scalar tolerance, defaults to a nonzero value) and LTVUnicycleController.atReference()/setTolerance() (structured tolerance object, strict < comparison, per-axis check) — I'm using the latter as the primary template since its tolerance is a compound value like the one I need to add here.

I ran this through the Phase I 6-check selection criteria and it passed 5.5/6 — the only partial gap being that the issue's linked Discord discussion isn't accessible to me, which I addressed by relying on in-repo precedent instead, and by treating the "per-element vs. combined-norm" tolerance question as an open design question to flag on the issue rather than assume.

---

## Understanding the Issue

### Problem Description

LinearSystemLoop (a WPILib class combining a controller, observer, and feedforward for full state-feedback control) exposes a .getError() method (the difference between the reference r and the observer's state estimate x-hat) but has no way to check whether that error is within an acceptable tolerance. Other WPILib controllers already have this exact convenience method (atSetpoint() on PIDController, atReference() on LTVUnicycleController and LTVDifferentialDriveController), so this is a consistency gap, not a new concept — but unlike those controllers, LinearSystemLoop currently has no tolerance concept at all to build on.

### Expected Behavior

Calling a new atReference() method on LinearSystemLoop should return true if the system's current error (from getError()) is within a configured tolerance for every state, and false otherwise — mirroring LTVUnicycleController.atReference()'s per-element, strict-less-than comparison.

### Current Behavior

No tolerance field and no atReference() method exist on LinearSystemLoop, in the Java implementation (org.wpilib.math.system.LinearSystemLoop), the C++ implementation (wpi::math::LinearSystemLoop), or the Python (RobotPy) semiwrap binding config. A caller who wants this behavior currently has to manually call getError() and compare it against tolerances they track themselves.

### Affected Components

- wpimath/src/main/java/org/wpilib/math/system/LinearSystemLoop.java (Java)
- wpimath/src/main/native/include/wpi/math/system/LinearSystemLoop.hpp (C++, header-only)
- wpimath/src/main/python/semiwrap/LinearSystemLoop.yml (Python/RobotPy binding config — not mentioned in the original issue, but required per CONTRIBUTING.md's rule that new user-facing methods need wrapper configs updated)
- wpimath/src/test/java/org/wpilib/math/controller/LinearSystemLoopTest.java and its C++ equivalent

---

## Reproduction Process

### Environment Setup

Repo builds via Gradle; no devcontainer present. Cloned fork already had a working main; found local main was 43 commits behind upstream/main (added upstream remote pointing at wpilibsuite/allwpilib, fast-forwarded, pushed the sync to my fork) before branching, to avoid basing work on stale code.

### Steps to Reproduce

1. Read wpimath/src/main/java/org/wpilib/math/system/LinearSystemLoop.java and wpimath/src/main/native/include/wpi/math/system/LinearSystemLoop.hpp directly.
2. Confirmed getError()/Error() exist (return reference r - observer x-hat), but grepped both files for atReference/tolerance/Tolerance — zero matches in either language.
3. Also checked wpimath/src/main/python/semiwrap/LinearSystemLoop.yml (RobotPy binding config) — no tolerance/atReference entries either, confirming this is a three-language gap (Java, C++, Python), not two as the issue implies.

### Reproduction Evidence

- My findings: The issue's premise that LinearSystemLoop "already stores a tolerance vector" is inaccurate as of the current main — there is no tolerance field at all. The class only exposes getError(). So the fix isn't just "combine two existing things" — it requires adding a new tolerance field and setter, then a comparison method, in Java, C++, and the Python wrapper config.
- Branch: fix-issue-4098 (based on upstream/main @ bf5f00490).

---

## Solution Approach

### Analysis

Two live precedents exist for this exact pattern: PIDController.atSetpoint() (single scalar, default nonzero tolerance) and LTVUnicycleController.atReference()/setTolerance() (structured tolerance object, uninitialized-by-default field, strict < comparison). The two "template" classes cited in the original issue — HolonomicDriveController and RamseteController — were both removed from the codebase in the 2027 refactor, so they're not available as references.

### Proposed Solution

Add a Matrix<States, N1> m_tolerance field (Java) / StateVector m_tolerance field (C++) to LinearSystemLoop, defaulted to a zero vector (unlike LTVUnicycleController's uninitialized field, to avoid a null/undefined-behavior footgun on a generic-typed vector). Add setTolerance(...) and atReference()/AtReference() that checks, per-element, abs(getError(i)) < tolerance.get(i) — strict less-than, matching LTVUnicycleController's convention. Mirror the change into the LinearSystemLoop.yml semiwrap config so the Python (RobotPy) binding picks it up.

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
