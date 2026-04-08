---
name: Generate AGENTS.md file
overview: "Create a comprehensive AGENTS.md at the project root that provides AI coding agents with essential context about the react-native-video-trim library: architecture, conventions, file structure, build commands, and contribution patterns."
todos:
  - id: write-agents-md
    content: Write AGENTS.md at project root with all sections covering architecture, conventions, build commands, CI, and contribution patterns
    status: completed
isProject: false
---

# Generate AGENTS.md for react-native-video-trim

## Context

The project already has a `.github/copilot-instructions.md` that covers architecture overview, development patterns, and platform specifics. The new `AGENTS.md` at the project root will serve a similar but broader purpose -- it is the standard file that AI coding agents (Cursor, Copilot, etc.) read for project-level guidance. It should be authoritative, concise, and cover everything an agent needs to contribute correctly.

## Content Structure

The `AGENTS.md` will include the following sections, derived from the full codebase analysis:

### 1. Project Overview
- React Native library for video/audio trimming, dual-architecture (Old + New/Fabric)
- Built with `react-native-builder-bob`, Yarn 4 workspaces, TypeScript
- Platforms: iOS (Swift + Obj-C++), Android (Kotlin + Java)
- Core dependency: FFmpegKit for media processing

### 2. Repository Layout
- `src/` -- TypeScript source (entry point, TurboModule spec, old-arch bridge)
- `ios/` -- Swift/Obj-C++ native implementation (flat directory)
- `android/src/main/` -- shared Kotlin/Java base classes + UI + utils
- `android/src/oldarch/` and `android/src/newarch/` -- architecture-specific modules
- `example/` -- Yarn workspace example app (RN 0.79.2, React 19)
- Key config files: `package.json`, `tsconfig.json`, `eslint.config.mjs`, `lefthook.yml`, `VideoTrim.podspec`, `android/build.gradle`

### 3. Architecture & Data Flow
- Runtime architecture detection via `global.nativeFabricUIManager`
- TypeScript spec in `src/NativeVideoTrim.ts` drives codegen (`VideoTrimSpec`)
- Factory functions (`createBaseOptions`, `createEditorConfig`, `createTrimOptions`) with defaults and `processColor` for colors
- Event system: Old Arch uses single `"VideoTrim"` event with `name` field; New Arch uses per-event codegen emitters

### 4. Adding Features Checklist
- Step-by-step: spec -> base implementation -> arch-specific wiring -> JS export

### 5. Code Style & Conventions
- Prettier config from `package.json` (single quotes, 2-space tabs, trailing commas es5)
- ESLint 9 flat config with `@react-native` + Prettier integration
- Conventional commits enforced by commitlint via Lefthook
- TypeScript strict mode with `verbatimModuleSyntax`

### 6. Build & Development Commands
- Key scripts: `yarn prepare`, `yarn lint`, `yarn typecheck`, `yarn test`, `yarn example start/android/ios`
- Turbo for native builds: `yarn turbo run build:android`, `yarn turbo run build:ios`
- Architecture testing env vars

### 7. CI Pipeline
- GitHub Actions: lint, typecheck, test, build library, build Android, build iOS
- Caching strategy: Yarn, Gradle, CocoaPods, Turborepo

### 8. Platform-Specific Notes
- iOS: VideoTrim.podspec, FFmpegKit package env var, Swift/Obj-C++ bridge pattern
- Android: Gradle source sets, composition pattern, FileProvider, FFmpeg-mobile dependency

### 9. Testing
- Jest with `react-native` preset (currently placeholder test)
- Manual testing via `example/` app

### 10. Release Process
- `yarn release` via release-it with conventional changelog

## Key Files to Reference

- [package.json](package.json) -- dependencies, scripts, prettier/commitlint/bob/codegen config
- [src/NativeVideoTrim.ts](src/NativeVideoTrim.ts) -- TurboModule spec (source of truth for API surface)
- [src/index.tsx](src/index.tsx) -- public JS API with factory functions
- [ios/VideoTrim.mm](ios/VideoTrim.mm) -- dual-arch native bridge (iOS)
- [ios/VideoTrim.swift](ios/VideoTrim.swift) -- core iOS implementation
- [android/src/main/java/com/videotrim/BaseVideoTrimModule.kt](android/src/main/java/com/videotrim/BaseVideoTrimModule.kt) -- core Android implementation
- [.github/workflows/ci.yml](.github/workflows/ci.yml) -- CI pipeline
- [CONTRIBUTING.md](CONTRIBUTING.md) -- contributor guide