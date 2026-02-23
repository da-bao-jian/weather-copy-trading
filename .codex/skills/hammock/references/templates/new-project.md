# New Project Template

Use this template when starting a new project or system from scratch.

## Key Principle

**Start with Why**: Before any technical decisions, deeply understand the problem space. New projects have the most freedom but also the most risk of building the wrong thing.

## AskUserQuestion Flow

### Initial Discovery
```
Question 1: "What problem does this project solve?"
Header: "Problem"
Options: [Open-ended - encourage "Other"]

Question 2: "Who is the primary user/audience?"
Header: "Users"
Options:
- Internal team: "Used by your organization"
- External customers: "Used by customers/clients"
- Developers: "API/tool for other developers"
- Yourself: "Personal project"

Question 3: "What's the timeline reality?"
Header: "Timeline"
Options:
- Exploration: "Learning/experimenting, no deadline"
- Flexible: "Want it done but timeline is soft"
- Weeks: "Need something working in weeks"
- Days: "Urgent, need MVP fast"

Question 4: "What's the expected lifespan?"
Header: "Lifespan"
Options:
- Throwaway: "Prototype, will rewrite"
- Short-term: "Months, then replaced"
- Long-term: "Years of maintenance expected"
- Unknown: "Not sure yet"
```

### Technical Direction
```
Question 1: "Any technology constraints or preferences?"
Header: "Tech stack"
Options:
- Flexible: "Open to recommendations"
- Existing stack: "Must fit current tech"
- Specific tech: "Have requirements (specify)"
- Learning goal: "Want to learn specific tech"

Question 2: "What's the deployment target?"
Header: "Deploy"
Options:
- Local only: "Runs on local machine"
- Cloud hosted: "Deploy to cloud service"
- Self-hosted: "On own infrastructure"
- Multiple: "Multiple environments"
```

## Document Structure

```markdown
# [Project Name] - Project Design

**Type**: New Project
**Date**: [date]
**Status**: [Discovery/Design/Ready to Build]

## Vision

### Problem Statement
[What problem exists in the world that this project addresses?]

### Target Users
[Who will use this? What do they care about?]

### Success Looks Like
[Paint a picture of the solved state]

### Non-Goals
[What this project explicitly won't do]

## Discovery

### User Research
- **Who**: [User types]
- **Pain points**: [What frustrates them today]
- **Current solutions**: [How do they solve this now]
- **Why those fail**: [Gaps in current solutions]

### Domain Understanding
[Key concepts, terminology, domain rules]

### Assumptions
- [ ] [Assumption 1] - [How to validate]
- [ ] [Assumption 2] - [How to validate]

## Requirements

### Must Have (MVP)
1. [Requirement] - [Why essential]
2. [Requirement] - [Why essential]

### Should Have (V1)
1. [Requirement] - [Value added]
2. [Requirement] - [Value added]

### Could Have (Future)
1. [Requirement] - [Nice to have]

### Won't Have (Out of Scope)
1. [Explicitly excluded] - [Why excluded]

## Technical Design

### Architecture Overview
```
[High-level diagram or description]
```

### Key Technical Decisions

#### Decision 1: [Topic]
**Options considered**:
- Option A: [Description]
- Option B: [Description]

**Chosen**: [Option]
**Reasoning**: [Why]
**Tradeoff**: [What we sacrifice]

#### Decision 2: [Topic]
[Same structure]

### Technology Stack
| Layer | Choice | Rationale |
|-------|--------|-----------|
| Frontend | | |
| Backend | | |
| Database | | |
| Infrastructure | | |

### Data Model
[Key entities and relationships]

### External Dependencies
- [Dependency 1]: [What it provides, risk level]
- [Dependency 2]: [What it provides, risk level]

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| [Risk 1] | | | |
| [Risk 2] | | | |

## Implementation Plan

### Phase 1: Foundation - [Timeframe]
**Goal**: [What this achieves]
- [ ] [Task 1]
- [ ] [Task 2]

**Milestone**: [How we know this phase is done]

### Phase 2: Core Features - [Timeframe]
**Goal**: [What this achieves]
- [ ] [Task 1]
- [ ] [Task 2]

**Milestone**: [How we know this phase is done]

### Phase 3: Polish & Launch - [Timeframe]
**Goal**: [What this achieves]
- [ ] [Task 1]
- [ ] [Task 2]

**Milestone**: [How we know this phase is done]

## Open Questions

- [Question 1]: Blocks [what], needs resolution by [when]
- [Question 2]: Can proceed without, revisit at [phase]

## Assumptions That Might Be Wrong

- **[Assumption]**: If wrong, we would [action]
- **[Assumption]**: If wrong, we would [action]
```

## Guiding Questions

### Problem Validation
1. Is this a real problem or a perceived one?
2. How do people solve this today?
3. Why haven't existing solutions won?
4. What would make someone switch to this?

### Scope Control
1. What's the smallest thing that delivers value?
2. What can we cut and still have something useful?
3. What are we explicitly NOT building?
4. What would V2 look like vs V1?

### Technical Choices
1. What's the simplest architecture that works?
2. Where do we need flexibility vs where can we be rigid?
3. What decisions are reversible vs irreversible?
4. What will we regret in a year?

### Risk Assessment
1. What's the biggest unknown?
2. What could kill this project?
3. What dependencies are we taking on?
4. What's the cost of being wrong?

## Anti-Patterns for New Projects

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| Build first, validate later | Waste effort on wrong thing | Validate problem before building |
| Premature optimization | Complexity without users | Build simple, optimize when needed |
| Technology-driven | Cool tech, no problem fit | Start with problem, pick tech to fit |
| Feature creep | Never ships | Ruthless MVP scoping |
| Perfect architecture | Paralysis | Good enough for current scale |
| No constraints | Analysis paralysis | Set artificial constraints |
