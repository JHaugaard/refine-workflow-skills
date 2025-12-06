# Planning Mindset Refinement Tracker

> **Purpose**: Track our refinement of the Planning Mindset concept as we prepare to integrate it into workflow skills and eventually extract a standalone skill.

---

## Working Approach

**Single-track, sequential refinement:**
1. Refine core Planning Mindset definition (here, in `.docs/`)
2. Weave appropriate elements into each workflow skill
3. When crystallized, extract standalone skill → migrate to `skill-foundry`

---

## Refinement Areas

### 1. Discovery Protocol ✅

**Refined structure**: 3 questions max, opportunity-focused (not pain-points)

| # | Question Type | Intent | Always/Situational |
|---|---------------|--------|-------------------|
| 1 | **Scope** | What's in, what's out? | Always |
| 2 | **Opportunity** | What becomes possible for you by building this? (journey-focused) | Always |
| 3 | **Curiosity Prompt** | What else should I know that I haven't asked? | Always |

**Key decisions**:

- Dropped: pain points, constraints, success criteria as benchmarks
- Added: organic mid-stream questions are expected and welcomed
- Flow principle: questions are conversation starters, not checklists

**Status**: 🟢 Complete

---

### 2. Framing Exercise ✅

**Refined approach**: Default framing blend, no options menu

**John's Framing Blend**:

1. **Solution-Centric** (primary): What gets built — the tangible thing
2. **Outcome-Centric** (secondary): What it enables — without rigid success metrics
3. **Exploration-Centric** (added): What you'll learn/discover along the way

**Key decisions**:

- Dropped: Problem-Centric (doesn't fit opportunity-focused mindset)
- No "pick from 2-3 framings" — use the blend, deviate only if signaled
- Reflects John's "eager learner" identity

**Status**: 🟢 Complete

---

### 3. Approval Gates ✅

**Refined approach**: Minimal friction, two gates, traffic-light signals

**Gates**:

| Gate | When | Prompt |
|------|------|--------|
| **Understanding** | After Discovery, before drafting | "Does this capture it?" |
| **Handoff** | Before moving to next phase | "Ready to proceed?" |

**Signal Language**:

| Signal | Meaning | Examples |
|--------|---------|----------|
| 🟢 Green | Continue | "Good" / "Continue" / "Yes" / "👍" |
| 🟡 Yellow | Adjust | "Yes, but..." / "Tweak X" / "Almost" |
| 🔴 Red | Stop | "Wait" / "Back up" / "Let's rethink" |

**Key decisions**:

- Silence never means proceed — always wait for signal
- Dynamic "Light" mode: gates stay light, but can slow down for learning moments
- Learning pauses decrease as experience grows

**Status**: 🟢 Complete

---

### 4. Planning Mindset vs Sequential Thinking ✅

**Refined approach**: Sequential Thinking off by default, human-invoked only

| Aspect | Decision |
|--------|----------|
| Default state | Not invoked |
| Trigger | Human-in-the-loop only |
| Frequency | Rare |
| Invocation | User explicitly requests it |

**Key decisions**:

- Planning Mindset is conversational, not tool-orchestration
- Sequential Thinking remains available but as a footnote, not integrated component
- Keeps the skill design clean and focused

**Status**: 🟢 Complete

---

## Integration Map

### Summary

| Workflow Skill | Elements to Integrate | Weight | Status |
|----------------|----------------------|--------|--------|
| project-brief-writer | Discovery Protocol, Framing Exercise, Approval Gates, Rationale Capture | Heavy | 🟢 Complete |
| tech-stack-advisor | Approval Gates (refine), Rationale Capture | Medium | 🔴 Pending |
| deployment-advisor | Approval Gates (refine), Rationale Capture | Medium | 🔴 Pending |
| project-spinup | Approval Gates only | Light | 🔴 Pending |
| test-orchestrator | Approval Gates only | Light | 🔴 Pending |
| ci-cd-implement | Approval Gates only | Light | 🔴 Pending |
| deploy-guide | None (execution phase) | Minimal | ⚪ N/A |

---

### project-brief-writer (Heavy Integration)

**Current state**: Template-driven, batch questions triggered by gaps, procedural checkpoints

**Changes needed**:

1. **Replace Phase 2 (template presentation)** with Discovery Protocol
   - Lead with: Scope → Opportunity → Curiosity Prompt
   - Flow into organic mid-stream questions
   - Remove template as entry point (keep as internal structure guide)

2. **Add Framing Exercise** after Discovery
   - Present project through Solution + Outcome + Exploration blend
   - Single reflection back, not options menu
   - Understanding Gate: "Does this capture it?"

3. **Simplify Phase 4 (clarifying questions)**
   - Remove batch triggers — organic flow instead
   - Remove pain-point/constraint/success-criteria language
   - Keep questions that naturally emerge from conversation

4. **Strengthen Approval Gates**
   - Use Green/Yellow/Red signal language
   - Understanding Gate after framing
   - Handoff Gate before tech-stack-advisor

5. **Add Rationale Capture** to handoff document
   - Record key decisions and why
   - Note alternatives considered

---

### tech-stack-advisor (Medium Integration)

**Current state**: Has checkpoints, Enterprise vs Hacker analysis, recommendation-focused

**Changes needed**:

1. **Refine checkpoint language** to match Approval Gates
   - Use "Does this capture it?" / "Ready to proceed?"
   - Support Green/Yellow/Red signals

2. **Add Rationale Capture section** to tech-stack-decision.md
   - Why primary recommendation fits
   - What alternatives were considered and why ruled out

3. **No Discovery Protocol** — context comes from upstream handoff

---

### deployment-advisor (Medium Integration)

**Current state**: Has checkpoints, decision framework, option-presentation focused

**Changes needed**:

1. **Refine checkpoint language** to match Approval Gates
2. **Add Rationale Capture section** to deployment-strategy.md
3. **No Discovery Protocol** — context comes from upstream

---

### project-spinup (Light Integration)

**Current state**: Execution phase, decisions already made upstream

**Changes needed**:

1. **Single Approval Gate** before generating
   - "Ready to generate your project foundation?"
   - Wait for explicit signal

2. **No other changes** — this is execution, not deliberation

---

### test-orchestrator, ci-cd-implement, deploy-guide

**Minimal or no changes** — these are execution phases where Planning Mindset doesn't apply. Approval Gates only where phase transitions occur.

---

## Future Considerations

- **User Preference Registry**: A user-level preferences file (`~/.claude/preferences.yaml`) that skills could read to eliminate repetitive "which do you prefer?" moments. Noted for later exploration in skill-foundry.

---

## Session Log

### Session 1 — 2024-12-06

- Reviewed planning-mindset.md from previous session
- Decided: single-track approach, work in refine-workflow-skills
- Identified four refinement areas
- Created this tracking document
- Completed all four refinement areas:
  - Discovery Protocol: 3 questions, opportunity-focused
  - Framing Exercise: Solution + Outcome + Exploration blend
  - Approval Gates: Minimal friction, two gates, traffic-light signals
  - Sequential Thinking: Off by default, human-invoked only
- **Next**: Create Integration Map for workflow skills

---

*Document created: 2024-12-06*
*Status: Core refinement complete, moving to integration*
