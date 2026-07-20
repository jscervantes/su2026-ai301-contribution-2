# Contribution 2: LinExp is broken for audio input parameters

**Contribution Number:** 2
**Student:** Jose Cervantes
**Issue:** [[GitHub issue link] ](https://github.com/supercollider/supercollider/issues/7556) 
**Status:** Phase 1 - Complete

---

## Why I Chose This Issue

Supercollider is a project I have had my eye on as a contribution candidate for a while now. This project falls at the intersection of technology and music, which I find very fascinating. This issue is tagged 'Good First Issue', which makes it a perfect issue to hop into the project!

---

## Understanding the Issue

### Problem Description

SuperCollider's LinExp UGen produces garbage output when the two members of a bounds pair (srclo/srchi or dstlo/dsthi) have mismatched signal rates, such as a scalar dstlo with an audio-rate dsthi. The dispatch function LinExp_SetCalc picks an audio-rate calc function if either member of a pair is audio-rate, but that calc function then iterates both members per-sample. Since a control-rate or scalar input only owns a 1-sample buffer, the loop reads past the end of it, feeding uninitialized memory into the output. The bug also affects .exprange, which is built on LinExp.

### Expected Behavior

LinExp.ar should compute the documented mapping dstlo * (dsthi/dstlo) ^ ((in - srclo) / (srchi - srclo)) correctly for any combination of input rates. Concretely, {LinExp.ar(DC.ar(0.5), -1.0, 1.0, 1.0, DC.ar(2.0))}.plot should output a constant ~1.6818, matching the scalar reference 0.5.linexp(-1.0, 1.0, 1.0, 2.0). Every rate combination of the four bounds should produce the same values as the all-control-rate equivalent (with audio-rate bounds simply updating per-sample instead of per-block).

### Current Behavior

When the two members of a bounds pair have different rates, the output "goes off the rails" (per the issue's plots): wildly wrong values, NaNs, or zeros, changing unpredictably because they depend on whatever memory lies past the 1-sample input buffer. The same expression works correctly if the mismatched bound is changed to control rate (e.g. DC.kr(2.0) instead of DC.ar(2.0)), which is what confirms the dispatch/iteration mismatch. The sclang-side checkInputs doesn't catch this because it only validates the first input's rate, and no error is raised — users just get silently corrupted audio.

### Affected Components

- server/plugins/LFUGens.cpp (server plugin, C++) — the core fix. Specifically LinExp_SetCalc (lines ~2320–2359, the faulty || rate dispatch) and the calc functions LinExp_next_aa / _ak / _ka (lines ~2276–2317, which unconditionally iterate all pair members). The fix implements the missing mixed-rate cases, per maintainer guidance in the issue, using per-input strides rather than 16 specialized functions.
- SCClassLibrary/Common/Audio/Line.sc (sclang class library) — LinExp class definition and its checkInputs; relevant for understanding why the bad combination reaches the server unchecked, though no change is required there under the chosen approach.
- SCClassLibrary/Common/Audio/UGen.sc — .exprange, which builds on LinExp and inherits the bug; fixed transitively.
- testsuite/classlibrary/ — new regression test comparing mixed-rate LinExp output against scalar .linexp reference values (alongside the existing TestCoreUGens.sc server-rendering patterns).

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

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
