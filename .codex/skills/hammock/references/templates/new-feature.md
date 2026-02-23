# New Feature Design Template

Use this template when designing a new feature or capability.

## Document Structure

```markdown
# [Feature Name] - Design Document

**Type**: New Feature
**Date**: [date]
**Status**: [Draft/Hammock Time/Final]

## Problem Statement

### User Problem
[What problem does the user face?]

### Business Context
[Why does this matter to the business now?]

### Success Metrics
- [ ] [Metric 1]
- [ ] [Metric 2]

## Understanding

### User Requirements
- [Requirement 1]
- [Requirement 2]

### Technical Context
- **Codebase area**: [Where does this fit?]
- **Related features**: [What existing features touch this?]
- **Data models**: [What data is involved?]

### Constraints
- **Must have**: [Non-negotiable requirements]
- **Performance**: [Latency, throughput requirements]
- **Compatibility**: [What must it work with?]

### Unknowns
- [ ] [Unknown 1] - [How to resolve]
- [ ] [Unknown 2] - [How to resolve]

## Research

### Existing Patterns
[What patterns exist in the codebase we can reuse?]

### Similar Features
[How do similar features work? What can we learn?]

### External Solutions
[How do other systems solve this? Pros/cons?]

## Solutions

### Option A: [Name]
**Approach**: [Description]

**Implementation**:
1. [Step 1]
2. [Step 2]

**Pros**:
- [Pro 1]

**Cons**:
- [Con 1]

**Sacrifices**: [What we give up]

### Option B: [Name]
**Approach**: [Description]

**Implementation**:
1. [Step 1]
2. [Step 2]

**Pros**:
- [Pro 1]

**Cons**:
- [Con 1]

**Sacrifices**: [What we give up]

## Tradeoffs

| Criterion | Option A | Option B |
|-----------|----------|----------|
| Complexity | | |
| Performance | | |
| User Experience | | |
| Maintenance | | |
| Time to implement | | |

## Recommendation

**Chosen**: [Option]

**Reasoning**: [Why this over alternatives]

## Implementation Plan

### Phase 1: [Name]
- [ ] [Task 1]
- [ ] [Task 2]

### Phase 2: [Name]
- [ ] [Task 1]
- [ ] [Task 2]

### Verification
- [ ] [How to test this works]

## Open Questions

- [Question 1]: If [condition], we would [action]
- [Question 2]: If [condition], we would [action]
```

## Guiding Questions for New Features

### Problem Discovery
1. Who specifically has this problem?
2. How do they work around it today?
3. What's the cost of not solving it?
4. Is this a real problem or a perceived one?

### Solution Scoping
1. What's the minimum viable version?
2. What would the ideal version look like?
3. What can we defer to later iterations?
4. What would make this fail?

### Technical Fit
1. Does this fit the existing architecture?
2. What would we have to change if it doesn't?
3. Are there existing patterns we can follow?
4. What new patterns would this introduce?
