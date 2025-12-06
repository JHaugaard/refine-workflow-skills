# Planning Mindset: A Pre-Codebase Planning Philosophy

> **Purpose**: Define a deliberate, exploratory approach for early-stage workflow phases where formal Plan Mode (designed for codebase exploration) doesn't apply, but the *spirit* of careful planning is essential.

---

## Context: Why Not Formal Plan Mode?

Claude Code's **Plan Mode** is powerful but designed for a specific context:
- Exploring an *existing* codebase
- Using tools like `Grep`, `Glob`, `Read` to understand structure
- Designing implementation approaches based on what exists

**The problem**: Early workflow phases (especially project-brief-writer) operate *before* a codebase exists. There's nothing to explore with code tools. Yet this phase arguably needs *more* deliberation than later phases—decisions made here cascade through everything that follows.

**The solution**: Adopt the *philosophy* of Plan Mode without its literal tool invocation. We call this the **Planning Mindset**.

---

## Core Principles of Planning Mindset

### 1. Exploration Before Commitment

**In Plan Mode**: "Let me read through the codebase before proposing changes."

**In Planning Mindset**: "Let me understand your vision fully before structuring it."

- Ask clarifying questions before drafting anything
- Surface assumptions explicitly
- Identify what's *not* said but implied
- Probe for constraints, preferences, and non-negotiables

### 2. Multiple Valid Paths

**In Plan Mode**: "There are several architectural approaches—here are the trade-offs."

**In Planning Mindset**: "There are several ways to frame this problem—here's what each emphasizes."

- Present 2-3 framings of the project concept
- Articulate what each framing prioritizes and de-emphasizes
- Let the user choose or synthesize
- Document *why* a framing was selected

### 3. Explicit Approval Gates

**In Plan Mode**: Uses `ExitPlanMode` to signal readiness for user review.

**In Planning Mindset**: Build in natural pause points for confirmation.

- "Before I draft the brief, does this understanding feel right?"
- "Here's the draft brief—shall I proceed to tech-stack-advisor?"
- Never auto-advance to next workflow phase without explicit approval

### 4. Rationale Documentation

**In Plan Mode**: The plan document captures reasoning.

**In Planning Mindset**: The handoff document captures reasoning.

- Record not just *what* was decided, but *why*
- Note alternatives considered and rejected
- Preserve context for future phases that may need to revisit decisions

### 5. Reversibility Awareness

**In Plan Mode**: Understanding which changes are easy to undo.

**In Planning Mindset**: Understanding which decisions are easy to revisit.

- Flag decisions that will be hard to change later
- Distinguish foundational choices from adjustable details
- Encourage user to scrutinize high-impact, low-reversibility decisions

---

## Component Parts of Planning Mindset

### A. Discovery Protocol

A structured approach to understanding the user's input:

1. **Receive input** - The brainstorm, rough idea, or initial brief
2. **Identify gaps** - What's missing? What's ambiguous?
3. **Surface assumptions** - What am I inferring that wasn't stated?
4. **Ask targeted questions** - 3-5 questions maximum, prioritized by impact
5. **Synthesize understanding** - Reflect back: "Here's what I understand..."

### B. Framing Exercise

Present the project concept through different lenses:

| Framing Type | Focus | Example |
|--------------|-------|---------|
| **User-Centric** | Who benefits and how | "A tool that helps freelancers track invoices" |
| **Problem-Centric** | What pain is solved | "Eliminating the chaos of scattered invoice records" |
| **Solution-Centric** | What gets built | "A dashboard with invoice CRUD and payment status" |
| **Outcome-Centric** | What success looks like | "Freelancers get paid faster with less admin overhead" |

Not all framings suit all projects—select 2-3 that illuminate meaningful choices.

### C. Decision Weight Matrix

Categorize decisions by impact and reversibility:

```
                    Easy to Change    Hard to Change
                    ─────────────────────────────────
High Impact    │   Review carefully   │  CRITICAL - pause here
               │                      │
Low Impact     │   Proceed freely     │  Note for awareness
               │                      │
```

Flag CRITICAL decisions for explicit user confirmation.

### D. Approval Gate Structure

Built-in pause points with clear prompts:

1. **Understanding Gate**: "Before drafting, does this capture your intent?"
2. **Draft Gate**: "Here's the brief—does this represent the project accurately?"
3. **Handoff Gate**: "Ready to proceed to [next phase]?"

Each gate requires explicit user confirmation to proceed.

### E. Rationale Capture Template

For handoff documents, include:

```markdown
## Decisions Made

### [Decision Topic]
- **Chosen**: [What was decided]
- **Why**: [Reasoning]
- **Alternatives Considered**: [What else was on the table]
- **Reversibility**: [Easy / Moderate / Difficult to change later]
```

---

## Integration with Project Brief Writer

The project-brief-writer skill should embody Planning Mindset through:

### Phase 1: Discovery
- Receive the brainstorm/rough idea
- Apply Discovery Protocol
- Ask clarifying questions (limit to 3-5 high-value questions)
- Reflect understanding back to user

### Phase 2: Framing
- Apply Framing Exercise
- Present 2-3 ways to conceptualize the project
- User selects or synthesizes preferred framing
- **Approval Gate**: Confirm framing before drafting

### Phase 3: Drafting
- Create the structured brief
- Include Rationale Capture for key decisions
- Highlight any CRITICAL decisions from Decision Weight Matrix
- **Approval Gate**: Review draft brief

### Phase 4: Handoff Preparation
- Finalize brief with any revisions
- Generate handoff document for tech-stack-advisor
- Include context needed for next phase
- **Approval Gate**: Confirm ready to proceed

---

## Relationship to Later Workflow Phases

| Phase | Planning Approach |
|-------|-------------------|
| **project-brief-writer** | Full Planning Mindset (no codebase yet) |
| **tech-stack-advisor** | Planning Mindset + light research (web search for options) |
| **deployment-advisor** | Planning Mindset + light research |
| **project-spinup** | Formal Plan Mode IF adapting existing code; otherwise Planning Mindset |
| **test-orchestrator** | Formal Plan Mode (codebase exists) |
| **ci-cd-implement** | Formal Plan Mode (codebase exists) |
| **deploy-guide** | Guided execution (decisions already made) |

---

## Key Takeaways

1. **Planning Mindset is the philosophy of Plan Mode, adapted for pre-codebase phases**
2. **Core elements**: Exploration, multiple paths, approval gates, rationale capture, reversibility awareness
3. **It's not a tool invocation**—it's a behavioral pattern woven into skill design
4. **Most valuable at project-brief-writer** where foundational decisions are made
5. **Transitions to formal Plan Mode** once a codebase exists to explore

---

## Future Considerations

- Should Planning Mindset have a visual/UX indicator? (e.g., a note that "deliberate exploration mode" is active)
- How to balance thoroughness with user patience? (Some users want speed)
- Could there be a "light" vs "full" Planning Mindset for different project scales?

---

## Tool Option: Sequential Thinking MCP Server

The **sequentialthinking** MCP server (available in Docker MCP Toolkit) could serve as an implementation mechanism for Planning Mindset in complex scenarios.

### What It Provides

- **Numbered thought chain** - Each reasoning step is a discrete, trackable unit
- **Explicit revision tracking** - Marks when previous thoughts are being reconsidered
- **Branching support** - Explore alternative paths without losing track
- **Dynamic scope** - Adjust expected number of thoughts as understanding evolves
- **Hypothesis-verification loops** - Propose → test → refine cycles

### When to Consider Using It

**Good fit** (complex, nuanced problems):
- Implementing semantic search functionality
- Designing complex data pipelines
- Multi-system integration architecture
- Problems where early assumptions often prove wrong
- Situations requiring explicit reasoning audit trails

**Overkill** (straightforward projects):
- Standard CRUD applications
- Well-understood patterns (to-do list, blog, portfolio)
- Projects where the path from idea to implementation is clear

### Integration with Planning Mindset

| Planning Mindset Element | Sequential Thinking Maps To |
|--------------------------|----------------------------|
| Exploration before commitment | Early exploratory thoughts |
| Multiple valid paths | `branch_from_thought`, `branch_id` |
| Rationale documentation | The thought chain itself |
| Reversibility awareness | `is_revision`, `revises_thought` |
| Approval gates | `next_thought_needed: false` pauses |

### Open Questions

- Token cost vs. insight value trade-off
- How to decide threshold for "complex enough" to warrant use
- Integration with other tools (does thinking trigger actions?)

### Next Step

Experiment with Sequential Thinking on a genuinely complex problem to evaluate whether the structured thought chain adds actionable value or just overhead.

---

*Document created: 2024-12-05*
*Status: Foundational concept for workflow skill design*
*Next step: Integrate into project-brief-writer skill specification*
