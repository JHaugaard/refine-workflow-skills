# Claude Code Memory Scaffolding Reference

Personal reference compiled December 2025 from Anthropic documentation and engineering posts.

---

## Overview

Claude Code uses two complementary approaches to maintain context and enable effective long-running work:

1. **File-based session memory** - CLAUDE.md files that persist preferences, conventions, and project context across sessions
2. **Long-running agent scaffolding** - Patterns for agents working across multiple context windows on complex tasks

These solve different problems: the first handles "what does Claude need to know about this project/user," while the second handles "how does an agent maintain coherent progress when context windows reset."

---

## Part 1: File-Based Memory (CLAUDE.md)

### Core Concept

Memory is stored in simple Markdown files named CLAUDE.md. All memory files are automatically loaded into Claude Code's context when launched. This is a deliberate design choice favoring transparency and user control over complex vector databases or RAG systems.

The content is directly injected into the model's prompt context for every session. Users curate these files manually, making memory a tangible, editable artifact that can be version-controlled alongside project code.

### The Four Memory Locations

Listed in order of precedence (highest to lowest):

**1. Enterprise Memory**

- Location: System-level path (macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`)
- Purpose: Organization-wide policies, security requirements, compliance rules
- Deployment: Via configuration management (MDM, Group Policy, Ansible, etc.)
- Scope: Applies to all users across all projects in the organization

**2. Project Memory**

- Location: `./CLAUDE.md` or `./.claude/CLAUDE.md` at project root
- Purpose: Team's shared context - architectural decisions, coding conventions, API patterns, tech stack
- Scope: Committed to version control, shared across all team members working on the project
- This is where project-specific rules live that should be consistent across the team

**3. User Memory**

- Location: `~/.claude/CLAUDE.md`
- Purpose: Personal preferences that travel across all projects
- Scope: Individual developer only, applies globally
- Examples: Preferred indentation, ESLint rules, commit message style, formatting preferences

**4. Project Local Memory**

- Location: `./CLAUDE.local.md`
- Purpose: Personal workspace notes for a specific project
- Scope: Individual developer, single project
- Key detail: Automatically added to .gitignore - ideal for sandbox URLs, personal debugging strategies, local environment specifics
- Not shared with team

### How Discovery Works

Claude Code reads memories recursively starting from the current working directory, traversing upward toward (but not including) the root directory. It reads any CLAUDE.md or CLAUDE.local.md files it finds along this path.

This means in a monorepo at `foo/bar/`, Claude loads memories from both `foo/CLAUDE.md` and `foo/bar/CLAUDE.md`, with more specific (deeper) files building on the foundation set by parent directories.

Claude also discovers CLAUDE.md files nested in subtrees under the current working directory, but these are only included when Claude reads files in those subtrees (not at launch).

### Import System

Memory files can reference other documents using `@path/to/import` syntax:

- Both relative and absolute paths work
- Imports can be recursive up to 5 levels deep
- Imports inside markdown code blocks are ignored (prevents accidental imports)
- Useful pattern: `@~/.claude/my-project-instructions.md` lets team members provide individual instructions not checked into the repo

### Best Practices

- Be specific: "Use 2-space indentation" beats "Format code properly"
- Use structure: Format individual memories as bullet points under descriptive markdown headings
- Keep it lean: Memory consumes context window space every session
- Use docs/ folder: For information needed only occasionally, store in docs/ and reference with `@docs/filename.md` rather than loading every session
- Review periodically: Update as projects evolve, remove outdated info
- Use `/memory` command to see what's loaded and edit files
- Use `#` at start of input to quickly add a memory during a session

---

## Part 2: Long-Running Agent Scaffolding

### The Problem

Complex projects cannot be completed within a single context window. When context resets between sessions, the agent has no memory of what came before. This creates two failure patterns:

1. **One-shotting**: Agent tries to do too much at once, runs out of context mid-implementation, leaves features half-done and undocumented. Next session has to guess what happened.

2. **Premature completion**: After some features are built, a later agent instance looks around, sees progress, and declares the job done.

The challenge: agents need a way to bridge the gap between sessions while leaving the environment in a clean, recoverable state.

### The Two-Agent Solution

Anthropic developed a two-part approach:

**Initializer Agent** (first session only)

Runs once at project start with a specialized prompt. Sets up:

- Feature list file: Comprehensive list of requirements expanding on the user's initial prompt, all initially marked as "failing"
- Progress file: `claude-progress.txt` to log what agents have done
- Init script: `init.sh` to run development server and basic tests
- Initial git commit showing what files were added

The feature list is critical - it prevents both one-shotting (by defining discrete units of work) and premature completion (by providing a clear checklist of what "done" looks like).

**Coding Agent** (all subsequent sessions)

Every session after initialization follows this pattern:

1. Get bearings: Run `pwd`, read progress file, read git logs, read feature list
2. Verify state: Run init.sh, do basic end-to-end test to catch any undocumented bugs
3. Work incrementally: Choose ONE feature to implement
4. Test properly: Verify end-to-end (not just unit tests), use browser automation for web apps
5. Leave clean: Commit with descriptive message, update progress file, mark feature status

### Key Insight: Human Engineering Practices

The solution draws directly from what effective software engineers do:

- Git commits with descriptive messages allow reverting bad changes
- Progress notes let the next person (or agent) understand what happened
- Feature lists provide clear scope and prevent scope creep
- End-to-end testing catches bugs that unit tests miss
- Clean state after each session means anyone can pick up the work

### Example: Session Start Sequence

A typical coding agent session begins:

1. Check current directory (`pwd`)
2. Read `claude-progress.txt` to see recent work
3. Read `feature_list.json` to see what's done and what's next
4. Check git log for recent commits
5. Run `init.sh` to start dev server
6. Run basic functionality test to verify nothing is broken
7. Only then: choose highest-priority incomplete feature and begin work

This sequence ensures the agent doesn't start implementing new features on top of a broken state.

### Why This Works

The scaffolding solves the core coordination problem:

- **State handoff**: Progress file + git history = clear record of what happened
- **Scope control**: Feature list prevents both over-ambition and premature stopping
- **Recovery**: Git enables reverting bad changes to known-good states
- **Verification**: End-to-end testing catches bugs before they compound
- **Efficiency**: Agent doesn't waste tokens figuring out how to run the app or what was done

### Failure Modes and Solutions Summary

| Problem | Initializer Does | Coding Agent Does |
|---------|------------------|-------------------|
| Declares victory too early | Creates feature list with all requirements | Reads feature list, works on one feature at a time |
| Leaves buggy/undocumented state | Sets up git repo and progress file | Reads progress + git logs at start, commits + updates at end |
| Marks features done prematurely | Creates structured feature list | Self-verifies with end-to-end testing before marking "passing" |
| Wastes time figuring out setup | Writes init.sh script | Reads and runs init.sh at session start |

### Open Questions

Anthropic notes these remain unresolved:

- Single general-purpose agent vs. multi-agent architecture (specialized testing agent, QA agent, cleanup agent)?
- Generalizing beyond web app development to scientific research, financial modeling, etc.

---

## Relationship Between the Two Approaches

These aren't competing solutions - they complement each other:

- **CLAUDE.md memory** handles persistent context about the project itself: what tech stack, what conventions, how to run things. This is relatively static.

- **Agent scaffolding** handles dynamic progress tracking: what's been done, what's next, what state is the code in. This changes every session.

A well-configured project uses both:
- CLAUDE.md tells the agent "here's how we do things here"
- Progress files tell the agent "here's where we are in the work"
- Feature lists tell the agent "here's what done looks like"
- Git history provides the audit trail and recovery mechanism

---

## Sources

- Anthropic Docs: Claude Code Memory (docs.claude.com)
- Anthropic Engineering: "Effective harnesses for long-running agents" (November 2025)
- Anthropic Engineering: "Building agents with the Claude Agent SDK"
