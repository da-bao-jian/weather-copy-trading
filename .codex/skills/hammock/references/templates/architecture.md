# Architecture Decision Template

Use this template for significant architectural decisions that will be hard to reverse.

## When to Use

- Introducing new technology/framework
- Changing data models or storage
- Modifying system boundaries
- Defining new patterns for the codebase
- Decisions that affect multiple teams/components

## Key Principle

**Hammock Time Critical**: Major architectural decisions benefit most from background processing. Consider sleeping on these decisions.

## Document Structure

```markdown
# [Decision Name] - Architecture Decision Record

**Type**: Architecture Decision
**Date**: [date]
**Status**: [Proposed/Hammock Time/Accepted/Superseded]
**Deciders**: [Who is involved]

## Context

### Current State
[How does the system work today?]

### Problem/Opportunity
[What's driving this decision?]

### Drivers
- [Driver 1]: [Why it matters]
- [Driver 2]: [Why it matters]

### Constraints
- **Technical**: [Must work with X, can't use Y]
- **Business**: [Timeline, budget, skills]
- **Organizational**: [Team structure, ownership]

## Decision

### Chosen Approach
[Brief statement of what we're doing]

### Key Principles
1. [Principle 1]
2. [Principle 2]

## Options Considered

### Option A: [Name]
**Description**: [What is this approach?]

**Architecture**:
```
[Diagram or description of structure]
```

**Pros**:
- [Pro 1]
- [Pro 2]

**Cons**:
- [Con 1]
- [Con 2]

**Risks**:
- [Risk 1]: Mitigation: [X]

**Effort**: [Low/Medium/High]

**Sacrifices**: [What we give up]

### Option B: [Name]
[Same structure as Option A]

### Option C: [Name]
[Same structure as Option A]

## Tradeoffs Analysis

### Comparison Matrix

| Criterion | Weight | Option A | Option B | Option C |
|-----------|--------|----------|----------|----------|
| Simplicity | | | | |
| Performance | | | | |
| Scalability | | | | |
| Maintainability | | | | |
| Team familiarity | | | | |
| Migration effort | | | | |
| Reversibility | | | | |

### What We're Sacrificing
By choosing [Option], we are giving up:
- [Sacrifice 1]
- [Sacrifice 2]

This is acceptable because: [Reasoning]

## Consequences

### Positive
- [Consequence 1]
- [Consequence 2]

### Negative
- [Consequence 1]
- [Consequence 2]

### Risks
| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| [Risk 1] | | | |
| [Risk 2] | | | |

## Implementation

### Migration Path
1. [Phase 1]: [Description]
2. [Phase 2]: [Description]
3. [Phase 3]: [Description]

### Rollback Plan
If this fails, we can:
- [Rollback option 1]
- [Rollback option 2]

### Success Criteria
- [ ] [Metric 1]
- [ ] [Metric 2]

## Open Questions

- [Question 1]: Needs resolution by [date/phase]
- [Question 2]: Acceptable to defer because [reason]

## Assumptions That Might Be Wrong

- **[Assumption 1]**: If wrong, impact is [X], we would [action]
- **[Assumption 2]**: If wrong, impact is [X], we would [action]

## Related Decisions

- [Link to related ADR 1]
- [Link to related ADR 2]
```

## Architecture Questions

### Problem Framing
1. What problem are we actually solving?
2. Is this really an architecture problem or a feature problem?
3. What's the cost of not deciding now?
4. Who needs to be involved in this decision?

### Option Discovery
1. What's the simplest thing that could work?
2. What would we do with unlimited time/resources?
3. How have others solved this?
4. What would we do differently if starting from scratch?

### Consequence Analysis
1. What does this make easier?
2. What does this make harder?
3. What doors does this close?
4. What will we regret in 2 years?

### Reversibility Check
1. How hard is it to change this later?
2. What would migration look like?
3. Can we make a reversible version of this decision?
4. What's the blast radius if this is wrong?

## Hammock Prompts for Architecture

> "This decision will be hard to reverse. Before committing, consider taking time to let your background mind process the tradeoffs."

> "Architecture decisions compound over time. A day or week of thinking now could save months of rework later."

> "Sleep on it. If you wake up with doubts, that's important information."
