# Detailed Phase Instructions

## Phase 1: State the Problem - Deep Dive

### The Anti-Pattern
Users often come with feature requests, not problems:
- "Add a cache" (feature) vs "The page loads too slowly" (problem)
- "Add authentication" (feature) vs "Unauthorized users can access data" (problem)

### Questions to Ask
1. What behavior exists today that shouldn't?
2. What behavior is missing that should exist?
3. Who experiences this problem?
4. What happens if we don't solve it?
5. How will we know when it's solved?

### Output Format
```
PROBLEM: [One sentence]
CONTEXT: [Why this matters now]
SUCCESS CRITERIA: [Measurable outcomes]
```

---

## Phase 2: Understand the Problem - Deep Dive

### Facts Gathering
Explore codebase for:
- Current implementation (if exists)
- Related functionality
- Data models involved
- API contracts

### Context Mapping
- What patterns does this codebase use?
- What's the team's familiarity with relevant tech?
- What's the deployment/release process?
- Are there time constraints?

### Constraint Identification
Hard constraints (must meet):
- Performance requirements (latency, throughput)
- Security requirements (auth, encryption)
- Compatibility requirements (browsers, APIs, versions)
- Resource constraints (memory, CPU, cost)

Soft constraints (prefer to meet):
- Code style consistency
- Testing coverage
- Documentation standards

### Unknown Tracking

Mark unknowns explicitly:
```
UNKNOWN: [What we don't know]
IMPACT: [How it affects design]
RESEARCH: [How to resolve]
```

---

## Phase 3: Gather Input - Deep Dive

### Codebase Exploration
1. Find similar features - how were they implemented?
2. Identify reusable utilities, patterns, abstractions
3. Note anti-patterns to avoid

### External Research
1. How do similar systems solve this?
2. What are the known pitfalls?
3. Are there established best practices?

### Critical Analysis
For each solution studied:
- What problem does it actually solve?
- What are its limitations?
- What context was it designed for?
- Would those tradeoffs work for us?

---

## Phase 4: Analyze Tradeoffs - Deep Dive

### Solution Generation
Force yourself to generate alternatives:
1. The obvious solution
2. The simple solution (what's the minimum?)
3. The robust solution (what if we over-engineered?)
4. The unconventional solution (what would we do if [constraint] didn't exist?)

### Tradeoff Dimensions

| Dimension | Questions |
|-----------|-----------|
| Complexity | How hard to understand? Maintain? |
| Performance | Speed? Resource usage? |
| Flexibility | How easy to change later? |
| Risk | What could go wrong? |
| Effort | How long to implement? |
| Dependencies | What does this require? |

### The Sacrifice Question
For your preferred solution, complete:
"By choosing this, we are giving up ___"

If you can't answer this, you don't understand the tradeoffs.

---

## Phase 5: Hammock Time - Deep Dive

### When to Suggest
- Stuck between two valid approaches
- Solution feels forced or overcomplicated
- Major architectural decision
- User seems frustrated or rushed

### The Science
Rich Hickey references research showing:
- Background mind processes during sleep
- Subconscious finds connections conscious mind misses
- "Aha moments" come after incubation periods

### Practical Prompts
> "This is a significant decision. Before committing, consider sleeping on it. Your background mind will continue processing."

> "We've loaded up a lot of context. This might be a good point to step away and let it percolate."

> "Major abstractions often need time. Days or weeks isn't unusual for foundational decisions."

---

## Phase 6: Capture & Implement - Deep Dive

### Solution Documentation
Write as if explaining to someone who wasn't in the discussion:
- What did we decide?
- Why this over alternatives?
- What are we watching out for?

### Implementation Planning
Break into concrete steps:
1. Each step should be independently verifiable
2. Identify dependencies between steps
3. Note where we might discover we were wrong

### Iteration Mindset
Include in the document:
```
## Assumptions That Might Be Wrong
- [Assumption 1]: If wrong, we would [action]
- [Assumption 2]: If wrong, we would [action]
```

This normalizes course correction as part of the process.
