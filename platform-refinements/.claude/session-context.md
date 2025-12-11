# Session Context: Platform Refinements

## Current Focus

**Platform Expansion - PLANNING PHASE**

Evaluating addition of mobile (iOS/iPadOS) and desktop (macOS) platform support to the workflow skills.

**Status:** Phase 1 complete. Phase 2 details in roadmap for when ready.

---

## Completed This Session (2025-12-11)

### 1. Problem Identification

User observed that tech-stack-advisor doesn't consider:
- Native iOS/iPadOS (Swift/SwiftUI)
- Native macOS (SwiftUI/AppKit)
- Cross-platform mobile/desktop options

### 2. Scope Decisions

| Decision | Choice |
|----------|--------|
| Platform scope | Apple ecosystem only (no Android) |
| Distribution | Personal device only (no App Store) |
| Cross-platform | PWA + Tauri (Capacitor, React Native excluded) |

### 3. Phase 1 Complete

| Deliverable | Status |
|-------------|--------|
| PLATFORM-CONSIDERATIONS.md revised | ✅ |
| tech-stack-advisor platform detection | ✅ |
| JSON schema platform field | ✅ |

### 4. Design Principles Applied

- Platform is part of *output* (recommendation), not *input* (user choice upfront)
- Detect signals from brief, don't force platform selection
- Claude Code is the capable implementation partner — don't limit options based on user skill ceiling

---

## Phased Approach Options

### Option A: Full Native Support
All at once — iOS, macOS, cross-platform across all skills.
- Large effort, complete solution

### Option B: Cross-Platform First (Recommended)
**Phase 1:** PWA, Capacitor, Electron, Tauri (leverages web skills)
**Phase 2:** Native SwiftUI for iOS/macOS
**Phase 3:** React Native, Flutter (if desired)
- Incremental value, manageable scope

### Option C: Recommendation Only
tech-stack-advisor can recommend, but downstream skills say "not yet supported."
- Minimal effort, incomplete workflow

---

## Scope Decisions (Confirmed)

| Decision | Choice |
|----------|--------|
| Platform scope | Apple ecosystem only (no Android) |
| Distribution model | Personal device only (no App Store) |
| Deployment method | Xcode → device (USB/Wi-Fi) |

## Open Questions (Remaining)

1. **Phase preference?** Start with Phase 1 foundation, or jump to Phase 2?

## Cross-Platform Decisions (Confirmed)

| Include | Exclude |
|---------|---------|
| PWA | Capacitor |
| Tauri | React Native |

*PWA + Tauri provide solid base for someone starting from zero.*

---

## Key Documents

| Document | Location | Purpose |
|----------|----------|---------|
| Platform Considerations | `~/.claude/skills/tech-stack-advisor/PLATFORM-CONSIDERATIONS.md` | Reference for tech-stack-advisor |
| Expansion Roadmap | `../docs/platform-expansion-roadmap.md` | Full impact assessment and options |
| Parent Session Context | `../.claude/session-context.md` | Broader workflow skills context |

---

## Dependencies (If Proceeding)

| Dependency | Required For | Notes |
|------------|--------------|-------|
| macOS | Native iOS/macOS development | Cannot build iOS on Windows/Linux |
| Xcode | iOS/macOS projects | Free from App Store |
| Apple Developer Account | App Store distribution | $99/year |
| Fastlane (optional) | CI/CD automation | Open source |

---

## Glossary

| Term | Definition |
|------|------------|
| PWA | Progressive Web App — web app that can be "installed" and work offline |
| SwiftUI | Apple's declarative UI framework for iOS, macOS, watchOS, tvOS |
| Capacitor | Wraps web apps in native container for App Store distribution |
| Tauri | Lightweight alternative to Electron (Rust + system webview) |
| Fastlane | Automation tool for iOS/Android build, test, deploy |
| TestFlight | Apple's beta testing distribution platform |

---

## Next Steps

1. User reviews roadmap and decides on approach
2. Scope Android in/out
3. Prioritize skills for implementation
4. Create detailed implementation plan for chosen phase

---

## Notes

- Session: 2025-12-11
- This sub-project focuses specifically on platform expansion
- Parent project (`refine-workflow-skills`) tracks broader workflow skill refinements
