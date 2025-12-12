# Session Context: Platform Refinements

## Current Focus

**Platform Expansion - IMPLEMENTATION PHASE**

Adding mobile (iOS/iPadOS) and desktop (macOS) platform support to workflow skills.

**Status:** Phase 1-3 complete. project-spinup now supports native platforms. Remaining: deploy-guide assessment.

---

## Completed Work

### Session 2025-12-11: tech-stack-advisor

| Deliverable | Status |
|-------------|--------|
| PLATFORM-CONSIDERATIONS.md revised | ✅ |
| tech-stack-advisor platform detection | ✅ |
| JSON schema platform field | ✅ |

### Session 2025-12-12: deployment-advisor

| Deliverable | Status |
|-------------|--------|
| Phase 0.5 native-platform-handoff added | ✅ |
| Minimal handoff schema for native platforms | ✅ |
| HOSTING-TEMPLATES.md updated | ✅ |
| Short-circuit logic for ios/macos/tauri | ✅ |

### Session 2025-12-12: project-spinup

| Deliverable | Status |
|-------------|--------|
| Phase 1.5 native-platform-scaffolding added | ✅ |
| iOS/iPadOS scaffolding (SwiftUI + backend) | ✅ |
| macOS scaffolding (SwiftUI + backend) | ✅ |
| Tauri scaffolding (web frontend + Rust) | ✅ |
| JSON schema native_platform field | ✅ |
| EXAMPLES.md native platform examples | ✅ |
| QUICK_REFERENCE.md native scenarios | ✅ |
| Backend integration (homelab Supabase/PocketBase) | ✅ |

### Session 2025-12-12: Remaining skills assessment

| Skill | Changes Made | Notes |
|-------|--------------|-------|
| test-orchestrator | Minor additions | Added XCTest (iOS/macOS) and Tauri testing guidance to framework-recommendations |
| deploy-guide | Minor addition | Added native-platform-check to recognize and gracefully handle native projects |
| ci-cd-implement | Note added | Added native-platform-check noting CI/CD is future enhancement, with pointers to Xcode Cloud and tauri-action |

### Design Principles Applied

- Platform is part of *output* (recommendation), not *input* (user choice upfront)
- Detect signals from brief, don't force platform selection
- Claude Code is the capable implementation partner — don't limit options based on user skill ceiling
- Native platforms don't need hosting strategy — short-circuit to project-spinup

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

1. ~~**project-spinup** — needs platform-aware scaffolding (Xcode projects, Tauri setup)~~ ✅ COMPLETE
2. ~~**deploy-guide** — assess if device installation walkthrough needed~~ ✅ COMPLETE (added native-platform-check)
3. ~~**test-orchestrator** — assess if changes needed~~ ✅ COMPLETE (added XCTest + Tauri testing)
4. ~~**ci-cd-implement** — assess if changes needed~~ ✅ COMPLETE (noted as future enhancement)
5. **project-brief-writer** — No changes needed (platform emerges from requirements)

### Assessment: Remaining Skills (FINAL)

| Skill | Native Platform Impact | Action Taken |
|-------|----------------------|--------------|
| deploy-guide | Low | Added native-platform-check — gracefully exits for native projects |
| project-brief-writer | None | No changes needed — platform emerges from requirements |
| test-orchestrator | Low | Added XCTest (iOS/macOS) and Tauri testing to framework-recommendations |
| ci-cd-implement | Medium | Added native-platform-check noting future enhancement with pointers to Xcode Cloud/tauri-action |

---

## Notes

- Sessions: 2025-12-11, 2025-12-12
- This sub-project focuses specifically on platform expansion
- Parent project (`refine-workflow-skills`) tracks broader workflow skill refinements

---

## Summary: What's Now Possible

With these updates, the workflow skills can now:

1. **Detect native platform needs** from project brief signals (tech-stack-advisor)
2. **Short-circuit hosting decisions** for native platforms (deployment-advisor)
3. **Scaffold native projects** with:
   - SwiftUI project structure (iOS/macOS)
   - Tauri project structure (desktop)
   - Pre-configured backend integration (Supabase/PocketBase)
   - Platform-tailored claude.md with conventions and patterns
   - Guided Setup or Quick Start modes

**End-to-end flow example:**
```
User: "I want to build a habit tracker for my iPhone"
↓
tech-stack-advisor: Recommends SwiftUI + Supabase, platform: ios
↓
deployment-advisor: Detects native platform, creates minimal handoff
↓
project-spinup: Generates SwiftUI scaffold + SupabaseService.swift → homelab
↓
User: Creates Xcode project, copies files, builds to device
```
