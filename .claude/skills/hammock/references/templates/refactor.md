# Refactoring Plan Template

Use this template when planning a significant refactoring effort.

## Key Principle

**Problem vs Feature**: Refactoring should solve a real problem, not just "improve" code. What pain is this addressing?

## Document Structure

```markdown
# [Refactor Name] - Plan

**Type**: Refactoring
**Date**: [date]
**Status**: [Draft/In Progress/Complete]
**Scope**: [Files/modules affected]

## Problem Statement

### The Pain
[What's actually wrong? Be specific.]

Examples:
- "Adding a new payment method requires changes in 7 files"
- "Tests take 10 minutes because of X"
- "New developers can't understand the auth flow"

### Impact
- **Development velocity**: [How does this slow us down?]
- **Bug frequency**: [Does this cause bugs?]
- **Maintenance cost**: [Time spent working around this?]

### Why Now
[What triggered this refactoring?]

## Current State

### Code Analysis
[Describe the current structure]

```
current/
├── [file/module 1] - [what it does, what's wrong]
├── [file/module 2] - [what it does, what's wrong]
└── [file/module 3] - [what it does, what's wrong]
```

### Problems Identified
1. **[Problem 1]**: [Description, location]
2. **[Problem 2]**: [Description, location]
3. **[Problem 3]**: [Description, location]

### Dependencies
[What depends on this code? What will break?]

## Target State

### Desired Structure
```
refactored/
├── [new structure]
├── [new structure]
└── [new structure]
```

### Goals
- [ ] [Goal 1]: [How we'll know it's achieved]
- [ ] [Goal 2]: [How we'll know it's achieved]

### Non-Goals
- [What we're explicitly NOT doing]
- [What's out of scope]

## Approaches

### Option A: [Name] - Incremental
**Approach**: Small changes over time, never breaking main

**Steps**:
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Pros**:
- Low risk per change
- Can stop partway if needed
- Continuous integration

**Cons**:
- Takes longer overall
- Intermediate states may be awkward
- Harder to maintain momentum

**Sacrifices**: [Speed, possibly cleanliness of final result]

### Option B: [Name] - Big Bang
**Approach**: Complete rewrite, swap in when ready

**Steps**:
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Pros**:
- Clean final result
- Faster total time
- No awkward intermediate states

**Cons**:
- High risk
- Long time before value delivered
- Merge conflicts during development

**Sacrifices**: [Safety, ability to course-correct]

### Option C: [Name] - Strangler Pattern
**Approach**: Build new alongside old, gradually migrate

**Steps**:
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Pros**:
- New code is clean
- Gradual migration
- Can run both in parallel

**Cons**:
- Two systems to maintain temporarily
- Need routing/switching logic
- May never fully complete

**Sacrifices**: [Short-term simplicity]

## Recommendation

**Chosen**: [Option]

**Reasoning**: [Why this approach for this refactor]

## Implementation Plan

### Prerequisites
- [ ] [Test coverage in place for affected code]
- [ ] [Documentation of current behavior]
- [ ] [Team alignment on approach]

### Phase 1: [Name] - [Timeframe]
**Goal**: [What this achieves]

Changes:
- [ ] [Change 1]
- [ ] [Change 2]

Verification:
- [ ] [How to verify this phase worked]

### Phase 2: [Name] - [Timeframe]
[Same structure]

### Phase 3: [Name] - [Timeframe]
[Same structure]

### Cleanup
- [ ] [Remove old code]
- [ ] [Update documentation]
- [ ] [Remove feature flags]

## Risk Management

### Risks
| Risk | Mitigation |
|------|------------|
| Break existing functionality | [Tests, feature flags, gradual rollout] |
| Take too long | [Timeboxes, stop points, scope reduction] |
| Scope creep | [Explicit non-goals, regular check-ins] |

### Rollback Plan
If things go wrong:
1. [Rollback step 1]
2. [Rollback step 2]

### Stop Points
We should pause/reconsider if:
- [Condition 1]
- [Condition 2]

## Success Criteria

### Must Have
- [ ] [Criterion 1]
- [ ] [Criterion 2]

### Nice to Have
- [ ] [Criterion 1]
- [ ] [Criterion 2]

## Assumptions That Might Be Wrong

- **[Assumption 1]**: If wrong, we would [action]
- **[Assumption 2]**: If wrong, we would [action]
```

## Refactoring Questions

### Is This Necessary?
1. What's the actual pain this addresses?
2. How often do we hit this pain?
3. What's the cost of NOT refactoring?
4. Is there a simpler fix?

### Scope Control
1. What's the minimum change that solves the problem?
2. What are we explicitly NOT changing?
3. Where do we draw the boundary?
4. How do we prevent scope creep?

### Safety
1. What test coverage exists?
2. What could break?
3. How would we know if something broke?
4. How do we rollback if needed?

### Timing
1. Why now and not later?
2. What other work conflicts with this?
3. Do we have the bandwidth to finish?
4. What's the cost of stopping partway?

## Anti-Patterns

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Refactor while adding features | Two risks at once, hard to debug | Separate the work |
| No tests before refactoring | No safety net | Add characterization tests first |
| Scope creep | Never finishes | Explicit scope, non-goals |
| Perfectionism | Good enough is often fine | Focus on pain reduction |
| Big bang rewrites | High risk, often fail | Incremental or strangler |
