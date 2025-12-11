# Platform Expansion Roadmap

Adding mobile (iOS/iPadOS) and desktop (macOS) platform support to the workflow skills.

**Status:** Planning phase — scope decisions confirmed
**Created:** 2025-12-11

---

## Scope Decisions (Confirmed)

| Decision | Choice | Impact |
|----------|--------|--------|
| Platform scope | **Apple ecosystem only** | No Android, no Play Store |
| Distribution model | **Personal device only** | No App Store, no TestFlight |
| Deployment method | **Xcode → device** | Direct install via USB/Wi-Fi |

These decisions **dramatically simplify** the downstream skill changes. App Store complexity (code signing, provisioning profiles, review process, Fastlane, CI/CD for releases) is now out of scope.

---

## What's In Scope

- **Native iOS/iPadOS** (Swift/SwiftUI) — personal device deployment
- **Native macOS** (SwiftUI/AppKit) — direct run or local build
- **Cross-platform mobile** (React Native, Capacitor) — Apple targets only
- **Cross-platform desktop** (Electron, Tauri)
- **Hybrid approaches** (PWA, web + native companion)

## What's Out of Scope

- Android / Play Store
- App Store / Mac App Store submission
- TestFlight distribution
- Production code signing complexity
- Fastlane / CI/CD for app releases

---

## Completed

| Item | Status | Notes |
|------|--------|-------|
| Scope decisions | ✅ Confirmed | Apple-only, personal device, no App Store |
| Cross-platform decisions | ✅ Confirmed | PWA + Tauri (Capacitor, React Native excluded) |
| **Phase 1: Foundation** | ✅ Complete | See below |

### Phase 1 Deliverables

- **PLATFORM-CONSIDERATIONS.md** — Revised for simplified scope (Apple-only, personal device, PWA + Tauri)
- **tech-stack-advisor SKILL.md** — Added platform-signals detection in Phase 1
- **JSON schema** — Added optional `platform` field to tech-stack-decision.json

---

## Revised Skill Impact Assessment

With App Store out of scope, the effort picture changes significantly:

| Skill | Original Effort | Revised Effort | Notes |
|-------|-----------------|----------------|-------|
| tech-stack-advisor | Small | **Small** | Add platform detection, reference doc |
| project-brief-writer | Small | **Small** | Minor schema update |
| workflow-status | Small | **Small** | Recognize platform field |
| test-orchestrator | Medium | **Medium** | XCTest/XCUITest, simulator guidance |
| deployment-advisor | Large | **Small-Medium** | Just "Xcode → device" path |
| project-spinup | Large | **Medium** | Xcode project generation, SwiftUI templates |
| deploy-guide | Very Large | **Small** | Just local device deployment |
| ci-cd-implement | Very Large | **Small** | Optional local build scripts only |

**Total effort reduced from "Very Large" to "Medium"**

---

## Recommended Phased Approach

Given the simplified scope:

### Phase 1: Foundation — COMPLETE

See "Phase 1 Deliverables" above.

### Phase 2: Native Apple Support

- project-spinup: Xcode project generation, SwiftUI templates
- deployment-advisor: "Xcode → device" deployment path
- deploy-guide: Local device deployment walkthrough
- test-orchestrator: XCTest basics, simulator testing

### Phase 3: Cross-Platform Options (Confirmed)

- **PWA** — web app with install + offline capability
- **Tauri** — desktop app with web UI

*Capacitor and React Native excluded — PWA + Tauri provide solid base for someone starting from zero.*

---

## Dependencies

| Dependency | Required For | Notes |
|------------|--------------|-------|
| macOS | All native iOS/macOS development | Already have |
| Xcode | iOS/macOS projects | Free from App Store |
| Apple Developer Account | Apps that don't expire after 7 days | $99/year, optional |

---

## Next Up

**Phase 2: Native Apple Support** — when ready

- project-spinup: Xcode project generation, SwiftUI templates
- deployment-advisor: "Xcode → device" deployment path
- deploy-guide: Local device deployment walkthrough
- test-orchestrator: XCTest basics, simulator testing

---

## Other Pending Items

Noted during this planning session (not platform-specific):

| Item | Skill | Notes |
|------|-------|-------|
| Remove all cost/price considerations | deployment-advisor | Don't mention or compare costs |

---

## Glossary

| Term | Definition |
|------|------------|
| PWA | Progressive Web App — web app that can be "installed" and work offline |
| SwiftUI | Apple's declarative UI framework for iOS, macOS, watchOS, tvOS |
| Capacitor | Wraps web apps in native container for device deployment |
| Tauri | Lightweight desktop app framework (Rust + system webview) |
| XCTest | Apple's testing framework for Swift/Objective-C |

---

## Notes

- Personal device deployment works with free Apple ID (7-day app expiration) or paid Developer account (no expiration)
- This roadmap reflects the simplified Apple-only, personal-use scope
