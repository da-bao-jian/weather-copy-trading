# Bug Investigation Template

Use this template when investigating a bug to find root cause, not just a workaround.

## Key Principle

**Avoiding vs Solving**: Workarounds feel like success but aren't solutions. This template ensures we find the actual root cause.

## Document Structure

```markdown
# [Bug Name] - Investigation

**Type**: Bug Investigation
**Date**: [date]
**Status**: [Investigating/Root Cause Found/Fixed]
**Severity**: [Critical/High/Medium/Low]

## Symptom

### Observed Behavior
[What actually happens?]

### Expected Behavior
[What should happen?]

### Reproduction Steps
1. [Step 1]
2. [Step 2]
3. [Observe: ...]

### Frequency
- [ ] Always reproducible
- [ ] Intermittent (conditions: ___)
- [ ] Rare (last seen: ___)

## Investigation

### Hypotheses
| # | Hypothesis | Evidence For | Evidence Against | Status |
|---|------------|--------------|------------------|--------|
| 1 | [Theory] | | | Testing/Confirmed/Ruled Out |
| 2 | [Theory] | | | |

### Code Path Trace
[Trace the execution path where bug occurs]

```
entry_point()
  → function_a()
    → function_b()  ← [Bug likely here because...]
      → function_c()
```

### Data Examined
- [Log entries]
- [Database state]
- [Request/response data]

### Timeline
- [Time]: [Event/observation]
- [Time]: [Event/observation]

## Root Cause

### The Actual Problem
[What is fundamentally wrong, not just what triggers it]

### Why It Wasn't Caught
[What allowed this bug to exist?]
- [ ] Missing test coverage
- [ ] Edge case not considered
- [ ] Incorrect assumption about [X]
- [ ] Race condition
- [ ] Other: ___

## Solutions

### Option A: Quick Fix
**Approach**: [Minimal change to fix symptom]

**Pros**:
- Fast to implement
- Low risk

**Cons**:
- May not address root cause
- [Other cons]

**Sacrifices**: [What we give up]

### Option B: Proper Fix
**Approach**: [Address root cause]

**Pros**:
- Fixes root cause
- Prevents similar bugs

**Cons**:
- More time to implement
- [Other cons]

**Sacrifices**: [What we give up]

### Option C: Preventive Fix
**Approach**: [Fix + prevent recurrence]

**Pros**:
- Fixes issue
- Adds safeguards
- Improves codebase

**Cons**:
- Most time
- [Other cons]

**Sacrifices**: [What we give up]

## Recommendation

**Chosen**: [Option]

**Reasoning**: [Why this approach]

## Fix Implementation

### Changes Required
- [ ] [File/change 1]
- [ ] [File/change 2]

### Test Plan
- [ ] [Test that reproduces bug - should fail before fix]
- [ ] [Test that verifies fix]
- [ ] [Regression tests]

## Prevention

### What Would Have Caught This?
- [ ] [Type of test]
- [ ] [Code review focus]
- [ ] [Monitoring/alerting]

### Follow-up Actions
- [ ] Add test coverage for [X]
- [ ] Document [Y]
- [ ] Consider [Z] for similar code
```

## Investigation Questions

### Symptom Understanding
1. When did this start happening?
2. What changed around that time?
3. Does it happen in all environments?
4. Who/what is affected?

### Hypothesis Generation
1. What could cause this exact symptom?
2. What assumptions might be wrong?
3. What edge cases exist?
4. What dependencies could have changed?

### Root Cause Validation
1. Can I prove this is the cause, not just correlated?
2. Does fixing this prevent all reproduction paths?
3. Why didn't existing tests catch this?
4. Are there other places with the same pattern?

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Fix the symptom | Bug returns or moves | Find root cause |
| "It works now" | No understanding why | Prove the cause |
| Shotgun debugging | Wastes time, may introduce bugs | Systematic hypothesis testing |
| Blame external | Misses internal issues | Investigate fully first |
