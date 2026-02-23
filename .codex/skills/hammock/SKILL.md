---
name: Hammock
description: |
  Deep design and planning methodology based on Rich Hickey's Hammock Driven Development.
  Use when: designing new features, investigating complex bugs, making architecture decisions,
  planning refactors, or when user says "design this", "plan this", "think through",
  "let's think deeply about", "hammock time". Guides through problem understanding,
  research, tradeoffs analysis, and suggests hammock time for subconscious processing.
---

# Hammock Driven Development

A methodology for solving hard problems through deliberate thinking. Most bugs come from misconception, not typos. The cheapest place to fix bugs is during design.

## Interactive Process

Use `AskUserQuestion` tool throughout this process to gather information efficiently. Batch related questions together (max 4 per call). This creates a structured dialogue rather than free-form back-and-forth.

## Quick Start

Use AskUserQuestion to determine mode and project type:

```
Question 1: "What type of design work is this?"
Header: "Project type"
Options:
- New project: "Starting something from scratch"
- New feature: "Adding new capability to existing system"
- Bug investigation: "Finding root cause of an issue"
- Architecture decision: "Significant structural change"
- Refactor: "Improving existing code structure"
- Plan review: "Review an existing design or proposal"

Question 2: "How deep should we go?"
Header: "Mode"
Options:
- Quick: "Problem → Understand → Tradeoffs (time pressure)"
- Standard (Recommended): "Full process without hammock pauses"
- Deep: "Include hammock time prompts for major decisions"
```

## The Process

### Phase 1: State the Problem

Use AskUserQuestion:

```
Question: "What problem are we actually solving? (Describe the pain, not the feature)"
Header: "Problem"
Options:
- [Let user type - use "Other" flow]

Question: "How will we know this is solved?"
Header: "Success"
Options:
- [Let user type - use "Other" flow]
```

If user gives a feature instead of a problem, probe deeper:
```
Question: "You mentioned [feature]. What pain or issue does this address?"
Header: "Root problem"
```

Write the problem statement explicitly before proceeding.

### Phase 2: Understand the Problem

First, explore the codebase silently using Glob, Grep, Read tools. Then use AskUserQuestion to fill gaps:

```
Question: "What constraints must we respect?"
Header: "Constraints"
Options:
- Performance critical: "Has latency/throughput requirements"
- Security sensitive: "Handles auth, PII, or secrets"
- Backward compatible: "Must not break existing clients"
- Time constrained: "Has a deadline"
multiSelect: true

Question: "What's unclear that we should research?"
Header: "Unknowns"
Options:
- [Based on exploration findings, offer specific unknowns discovered]
- [Or let user type]
```

Document findings in categories:
| Category | What to Capture |
|----------|-----------------|
| **Facts** | Requirements, specs, existing behavior |
| **Context** | Codebase patterns, tech stack, team constraints |
| **Constraints** | Performance limits, security needs, compatibility |
| **Unknowns** | Knowledge gaps to research (mark these!) |

### Phase 3: Gather Input

Explore codebase and external sources. Then confirm with user:

```
Question: "I found these relevant patterns/solutions. Which should we consider?"
Header: "Patterns"
Options:
- [Pattern A found]: "[Brief description]"
- [Pattern B found]: "[Brief description]"
- [External approach]: "[Brief description]"
- Research more: "Need to look at additional solutions"
multiSelect: true
```

### Phase 4: Analyze Tradeoffs

After generating at least 2 solutions, use AskUserQuestion:

```
Question: "I've identified these approaches. Which factors matter most for your decision?"
Header: "Priorities"
Options:
- Simplicity: "Easiest to understand and maintain"
- Performance: "Speed and resource efficiency"
- Flexibility: "Easy to change later"
- Speed to implement: "Fastest to build"
multiSelect: true

Question: "Initial recommendation is [Option X]. Does this direction feel right?"
Header: "Direction"
Options:
- Yes, proceed: "Develop this approach further"
- Explore alternatives: "Look at other options more"
- Combine approaches: "Mix elements from multiple options"
- Rethink problem: "Step back and reconsider the problem"
```

### Phase 5: Hammock Time (Deep mode only)

Use AskUserQuestion:

```
Question: "This is a significant decision. Would you like to take hammock time before finalizing?"
Header: "Hammock"
Options:
- Continue now: "I'm ready to proceed"
- Sleep on it: "Save state, I'll return tomorrow"
- Think break: "Give me the summary, I'll think and return"
```

If they choose to pause, save the design document as draft with status "Hammock Time".

Prompt:
> Your background mind will process this. Return with `/hammock continue` when ready.

### Phase 6: Capture & Implement

Use AskUserQuestion for implementation planning:

```
Question: "How should we break down the implementation?"
Header: "Approach"
Options:
- Single PR: "All changes in one pull request"
- Phased: "Multiple PRs in sequence"
- Feature flag: "Behind a flag for gradual rollout"
```

Then generate the implementation plan with specific steps.

## Output Document

Generate design document at `[project]/.claude/designs/[name].md`:

```markdown
# [Problem Name] - Design Document

**Mode**: [Quick/Standard/Deep]
**Date**: [date]
**Status**: [Draft/Hammock Time/Final]

## Problem Statement
[What we're solving - not features, the actual problem]

## Understanding

### Facts
- ...

### Context
- ...

### Constraints
- ...

### Unknowns (Resolved)
- [x] [Unknown] → [What we learned]

### Unknowns (Open)
- [ ] [Still unknown]

## Research & Input
[Patterns found, solutions studied, what we learned]

## Solutions Considered

### Option A: [Name]
**Approach**: ...
**Pros**: ...
**Cons**: ...
**Sacrifices**: ...

### Option B: [Name]
**Approach**: ...
**Pros**: ...
**Cons**: ...
**Sacrifices**: ...

## Tradeoffs Matrix

| Criterion | Option A | Option B |
|-----------|----------|----------|
| ... | ... | ... |

## Recommendation
[Chosen approach]

**Reasoning**: [Why this over alternatives]

## Implementation Plan
1. ...
2. ...

## Open Questions
[Things that might make us reconsider]
```

## Templates

For specific scenarios, see:
- [references/templates/new-project.md](references/templates/new-project.md) - Starting a new project from scratch
- [references/templates/new-feature.md](references/templates/new-feature.md) - New feature design
- [references/templates/bug-investigation.md](references/templates/bug-investigation.md) - Bug root cause analysis
- [references/templates/architecture.md](references/templates/architecture.md) - Architecture decisions
- [references/templates/refactor.md](references/templates/refactor.md) - Refactoring plans
- [references/templates/plan-review.md](references/templates/plan-review.md) - Review existing designs/plans

## Key Principles

1. **Problem vs Feature**: Solve problems, don't just build features
2. **At Least Two**: Never evaluate one solution in isolation
3. **Know Your Unknowns**: Document what you don't know
4. **Tradeoffs Exist**: Every choice sacrifices something
5. **You Will Be Wrong**: Plan for iteration, embrace it

## AskUserQuestion Guidelines

- Batch related questions (up to 4 per call)
- Use multiSelect when choices aren't mutually exclusive
- Put recommended option first with "(Recommended)" suffix
- Keep option labels short (1-5 words)
- Use descriptions to explain implications
- For open-ended input, design options that encourage "Other" selection
