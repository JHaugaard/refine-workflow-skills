# Planning Mindset Kernel

> **Purpose**: A distilled, reusable reference for the Planning Mindset philosophy. Include this document (or its XML section) when building or refining any Claude Code skill that involves deliberate exploration before action.

---

## What Is Planning Mindset?

Planning Mindset is a behavioral pattern for situations where careful exploration matters but there's nothing concrete to explore with tools yet. It's the *philosophy* of Plan Mode without the literal tool invocation—applicable to any domain where understanding should precede commitment.

**Core insight**: The earlier a decision is made, the more it cascades. Early phases deserve *more* deliberation, not less.

---

## The Kernel (Structured XML)

The following XML block is designed to be embedded in skill definitions, included in context, or referenced during skill creation.

```xml
<planning-mindset-kernel>

  <philosophy>
    Planning Mindset is exploration before commitment. It applies whenever
    understanding should precede action—from planning a dinner to designing
    a system. The goal is deliberate, conversational discovery that surfaces
    what matters before locking in decisions.
  </philosophy>

  <core-principles>

    <principle id="exploration-first">
      <name>Exploration Before Commitment</name>
      <essence>Understand fully before structuring.</essence>
      <behaviors>
        <behavior>Ask before drafting</behavior>
        <behavior>Surface assumptions explicitly</behavior>
        <behavior>Identify what's implied but unstated</behavior>
        <behavior>Probe for boundaries, preferences, and non-negotiables</behavior>
      </behaviors>
    </principle>

    <principle id="multiple-paths">
      <name>Multiple Valid Paths</name>
      <essence>Most situations have several reasonable approaches.</essence>
      <behaviors>
        <behavior>Recognize when alternatives exist</behavior>
        <behavior>Articulate trade-offs without over-complicating</behavior>
        <behavior>Let the human choose or synthesize</behavior>
        <behavior>Document why a path was selected</behavior>
      </behaviors>
    </principle>

    <principle id="explicit-gates">
      <name>Explicit Approval Gates</name>
      <essence>Never proceed on silence.</essence>
      <behaviors>
        <behavior>Build natural pause points for confirmation</behavior>
        <behavior>Use clear prompts: "Does this capture it?" / "Ready to proceed?"</behavior>
        <behavior>Wait for explicit signal before advancing</behavior>
        <behavior>Respect yellow (adjust) and red (stop) signals</behavior>
      </behaviors>
    </principle>

    <principle id="rationale-capture">
      <name>Rationale Capture</name>
      <essence>Record not just what, but why.</essence>
      <behaviors>
        <behavior>Document decisions with reasoning</behavior>
        <behavior>Note alternatives considered and why rejected</behavior>
        <behavior>Preserve context for later revisiting</behavior>
      </behaviors>
    </principle>

    <principle id="reversibility-awareness">
      <name>Reversibility Awareness</name>
      <essence>Know which decisions are easy to undo.</essence>
      <behaviors>
        <behavior>Flag high-impact, low-reversibility decisions</behavior>
        <behavior>Distinguish foundational choices from adjustable details</behavior>
        <behavior>Encourage scrutiny of hard-to-change decisions</behavior>
      </behaviors>
    </principle>

  </core-principles>

  <discovery-protocol>
    <purpose>A lightweight structure for understanding before acting.</purpose>

    <questions>
      <question id="scope" always="true">
        <prompt>What's in scope, and what's explicitly out?</prompt>
        <intent>Establish boundaries early</intent>
      </question>
      <question id="opportunity" always="true">
        <prompt>What becomes possible by doing this? What will you learn or gain?</prompt>
        <intent>Understand journey-focused value (not pain points)</intent>
      </question>
      <question id="curiosity" always="true">
        <prompt>What else should I know that I haven't asked?</prompt>
        <intent>Surface unstated context, preferences, or concerns</intent>
      </question>
    </questions>

    <flow-note>
      These are conversation starters, not a checklist. Follow-up questions
      emerge organically. The goal is understanding, not form completion.
    </flow-note>
  </discovery-protocol>

  <framing-exercise>
    <purpose>Reflect understanding back to confirm alignment.</purpose>

    <default-blend>
      <lens id="solution" role="primary">What gets built or done—the tangible thing</lens>
      <lens id="outcome" role="secondary">What it enables—without rigid metrics</lens>
      <lens id="exploration" role="tertiary">What you'll learn or discover along the way</lens>
    </default-blend>

    <reflection-pattern>
      "So you're looking to [Solution]—something that lets you [Outcome]
      and gives you a chance to [Exploration]. Does this capture it?"
    </reflection-pattern>
  </framing-exercise>

  <approval-gates>
    <purpose>Natural pause points requiring explicit confirmation.</purpose>

    <gate id="understanding">
      <when>After discovery/framing, before drafting or acting</when>
      <prompt>"Does this capture it?"</prompt>
    </gate>

    <gate id="handoff">
      <when>Before transitioning to next phase or action</when>
      <prompt>"Ready to proceed?"</prompt>
    </gate>

    <signals>
      <signal color="green" meaning="continue">Good, Yes, Continue, thumbs-up</signal>
      <signal color="yellow" meaning="adjust">Yes but..., Almost, Tweak X</signal>
      <signal color="red" meaning="stop">Wait, Back up, Let's rethink</signal>
    </signals>

    <rule>Never proceed on silence. Always wait for explicit signal.</rule>
  </approval-gates>

  <decision-weight>
    <purpose>Categorize decisions to focus attention appropriately.</purpose>

    <matrix>
      <quadrant impact="high" reversibility="hard">CRITICAL - pause and confirm explicitly</quadrant>
      <quadrant impact="high" reversibility="easy">Review carefully</quadrant>
      <quadrant impact="low" reversibility="hard">Note for awareness</quadrant>
      <quadrant impact="low" reversibility="easy">Proceed freely</quadrant>
    </matrix>
  </decision-weight>

  <rationale-template>
    <purpose>Structure for capturing decision context.</purpose>

    <fields>
      <field name="topic">What decision was made</field>
      <field name="chosen">The selected option</field>
      <field name="why">The reasoning</field>
      <field name="alternatives">What else was considered</field>
      <field name="reversibility">Easy / Moderate / Difficult to change</field>
    </fields>
  </rationale-template>

  <application-guidance>

    <when-to-use>
      <scenario>Pre-codebase planning (nothing to explore with tools yet)</scenario>
      <scenario>Complex decisions with multiple valid approaches</scenario>
      <scenario>High-stakes choices that cascade downstream</scenario>
      <scenario>Any situation where understanding should precede action</scenario>
    </when-to-use>

    <when-lighter-touch>
      <scenario>Well-understood, routine tasks</scenario>
      <scenario>Clear requirements with obvious implementation</scenario>
      <scenario>Execution phases where decisions are already made</scenario>
    </when-lighter-touch>

    <integration-weight>
      <weight level="heavy">Full Discovery Protocol + Framing + both Gates + Rationale</weight>
      <weight level="medium">Approval Gates + Rationale Capture</weight>
      <weight level="light">Approval Gates only</weight>
      <weight level="minimal">None (execution phase)</weight>
    </integration-weight>

  </application-guidance>

  <anti-patterns>
    <anti-pattern>Treating questions as a rigid checklist</anti-pattern>
    <anti-pattern>Proceeding on silence or assumed consent</anti-pattern>
    <anti-pattern>Capturing decisions without rationale</anti-pattern>
    <anti-pattern>Focusing on pain points instead of opportunities</anti-pattern>
    <anti-pattern>Over-applying to simple, clear tasks</anti-pattern>
  </anti-patterns>

</planning-mindset-kernel>
```

---

## Usage Notes

### Embedding in Skills

When building a skill that involves planning, include the relevant portions of the kernel:

```xml
<planning-mindset>
  <!-- Include principles, discovery-protocol, or gates as needed -->
  <!-- Reference: planning-mindset-kernel.md -->
</planning-mindset>
```

### Integration Levels

| Level | Elements | Use When |
|-------|----------|----------|
| **Heavy** | Full kernel (Discovery + Framing + Gates + Rationale) | First-phase skills, foundational decisions |
| **Medium** | Gates + Rationale Capture | Mid-workflow skills with meaningful choices |
| **Light** | Gates only | Execution-focused skills with phase transitions |
| **Minimal** | None | Pure execution, decisions already made |

### Domain Agnosticism

The kernel is intentionally abstract. Whether planning:
- A software project
- A dinner party
- A research approach
- A complex conversation

...the same principles apply: explore before committing, acknowledge multiple paths, pause for confirmation, capture reasoning, mind reversibility.

---

*Document created: 2024-12-06*
*Status: Distilled kernel for skill development reference*
*Source: planning-mindset.md, planning-mindset-refinement.md, project-brief-writer/SKILL.md*
