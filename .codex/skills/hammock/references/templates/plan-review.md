# Design & Plan Review Template

Use this template when reviewing an existing design, plan, or proposal to find gaps, risks, and improvements.

## Key Principle

**Steel Man First**: Understand the design's intent before critiquing. Find what's good before finding what's wrong. The goal is to make the plan better, not to prove it wrong.

## AskUserQuestion Flow

### Review Setup
```
Question 1: "What's your relationship to this design?"
Header: "Role"
Options:
- Author: "I wrote it, want feedback"
- Reviewer: "Someone else wrote it, I'm reviewing"
- Inheritor: "I need to implement/maintain this"
- Stakeholder: "This affects my work"

Question 2: "What kind of review do you need?"
Header: "Review type"
Options:
- Sanity check: "Quick gut check, obvious issues"
- Deep review: "Thorough analysis of approach"
- Risk assessment: "Focus on what could go wrong"
- Improvement ideas: "Make a good plan better"

Question 3: "What's the decision timeline?"
Header: "Timeline"
Options:
- Urgent: "Deciding today"
- Soon: "Deciding this week"
- Planning: "Early stage, time to iterate"

Question 4: "Any specific concerns to focus on?"
Header: "Focus areas"
Options:
- Technical feasibility: "Will this actually work?"
- Scalability: "Will it handle growth?"
- Maintainability: "Can we live with this long-term?"
- Completeness: "What's missing?"
multiSelect: true
```

## Document Structure

```markdown
# [Design Name] - Review

**Type**: Design Review
**Date**: [date]
**Reviewer**: [who]
**Original Design**: [link/reference]
**Review Type**: [Sanity Check/Deep Review/Risk Assessment/Improvement]

## Summary

### Design Intent
[What is this design trying to achieve? Steel man the approach.]

### Overall Assessment
- **Recommendation**: [Approve / Approve with changes / Needs rework / Reject]
- **Confidence**: [High/Medium/Low]
- **Key insight**: [One sentence summary of most important finding]

## What's Good

[Don't skip this section - acknowledge strengths before weaknesses]

- **[Strength 1]**: [Why this is good]
- **[Strength 2]**: [Why this is good]
- **[Strength 3]**: [Why this is good]

## Critical Issues

Issues that must be addressed before proceeding.

### Issue 1: [Title]
**Severity**: Critical
**Category**: [Feasibility/Correctness/Security/Performance/Maintainability]

**Problem**: [What's wrong]

**Impact**: [What happens if not addressed]

**Suggested Fix**: [How to address]

### Issue 2: [Title]
[Same structure]

## Concerns

Issues that should be addressed but aren't blockers.

### Concern 1: [Title]
**Severity**: Medium
**Category**: [Category]

**Problem**: [What's concerning]

**Risk**: [What could go wrong]

**Suggestion**: [How to mitigate]

### Concern 2: [Title]
[Same structure]

## Questions

Things that need clarification or aren't addressed.

| # | Question | Why It Matters | Blocking? |
|---|----------|----------------|-----------|
| 1 | [Question] | [Impact on design] | Yes/No |
| 2 | [Question] | [Impact on design] | Yes/No |

## Missing Elements

What the design should address but doesn't.

- [ ] **[Missing element]**: [Why it's needed]
- [ ] **[Missing element]**: [Why it's needed]

## Alternative Approaches

Other options worth considering.

### Alternative 1: [Name]
**Approach**: [Description]
**Tradeoff vs current**: [What's better, what's worse]
**When to consider**: [Conditions that favor this approach]

### Alternative 2: [Name]
[Same structure]

## Risk Assessment

### Identified Risks

| Risk | Likelihood | Impact | In Design? | Mitigation Adequate? |
|------|------------|--------|------------|---------------------|
| [Risk 1] | H/M/L | H/M/L | Yes/No | Yes/No/Partial |
| [Risk 2] | H/M/L | H/M/L | Yes/No | Yes/No/Partial |

### Unaddressed Risks
- **[Risk]**: [Why it matters, suggested mitigation]

## Assumptions Check

| Assumption in Design | Valid? | If Wrong... |
|---------------------|--------|-------------|
| [Assumption 1] | Yes/No/Uncertain | [Consequence] |
| [Assumption 2] | Yes/No/Uncertain | [Consequence] |

## Implementation Concerns

If this design is implemented as-is:

- **Easy parts**: [What will go smoothly]
- **Hard parts**: [What will be challenging]
- **Unknowns**: [What we'll discover during implementation]

## Recommendations

### Must Do (Before Proceeding)
1. [Action item]
2. [Action item]

### Should Do (Before or During Implementation)
1. [Action item]
2. [Action item]

### Consider (For Future Iterations)
1. [Action item]
2. [Action item]

## Reviewer Confidence

| Aspect | Confidence | Notes |
|--------|------------|-------|
| Problem understanding | H/M/L | [Any gaps in context] |
| Technical assessment | H/M/L | [Areas of uncertainty] |
| Alternative awareness | H/M/L | [Other approaches I might not know] |
```

## Review Checklist

### Problem & Requirements
- [ ] Is the problem clearly stated?
- [ ] Are success criteria defined?
- [ ] Are requirements complete?
- [ ] Are non-goals explicit?
- [ ] Is scope appropriate?

### Solution Design
- [ ] Does the solution address the problem?
- [ ] Are there at least 2 options considered?
- [ ] Are tradeoffs explicit?
- [ ] Is the recommendation justified?
- [ ] Is the architecture appropriate for scale?

### Risks & Edge Cases
- [ ] Are failure modes considered?
- [ ] Is there a rollback plan?
- [ ] Are security implications addressed?
- [ ] Are performance implications addressed?
- [ ] What happens at 10x scale?

### Implementation
- [ ] Is the plan actionable?
- [ ] Are dependencies identified?
- [ ] Are milestones clear?
- [ ] Is testing strategy defined?
- [ ] Are unknowns acknowledged?

### Meta
- [ ] Is the design appropriately detailed for the stage?
- [ ] Are assumptions stated?
- [ ] Is it clear who decides?
- [ ] Is the timeline realistic?

## Review Mindsets

### Devil's Advocate
Ask: "What would make this fail?"
- How could this go wrong?
- What's the worst case scenario?
- What assumptions are we betting on?

### User Advocate
Ask: "Does this serve users?"
- Will users actually want this?
- Is the UX considered?
- What's the user's journey?

### Future Maintainer
Ask: "Will we regret this in 2 years?"
- Can someone else understand this?
- What technical debt are we taking on?
- How hard is this to change later?

### Operations
Ask: "What happens in production?"
- How do we deploy this?
- How do we monitor it?
- How do we debug issues?
- What's the blast radius of failures?

## Anti-Patterns in Reviews

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Nitpicking | Misses forest for trees | Focus on what matters |
| Bike-shedding | Easy stuff over hard stuff | Prioritize critical issues |
| Rejecting without alternatives | Unconstructive | Offer better approaches |
| Rubber stamping | Provides no value | Find at least one improvement |
| Not understanding intent | Critiques wrong thing | Steel man first |
| Perfectionism | Nothing ships | Good enough for current stage |
