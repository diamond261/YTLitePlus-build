# AGENTS.md
Guidance for coding agents working in `YTLitePlus-build`.

## 1) Repository Snapshot
- Project type: Theos/Logos iOS tweak build for YouTube.
- Main languages: Objective-C/Logos (`.xm`, `.h`) plus small Python utilities.
- Primary outputs: rootless `.ipa` packages under `packages/`.
- Build orchestration: `Makefile`, `build.sh`, GitHub Actions.

## 2) High-Signal Paths
- `Makefile`: canonical build flags, targets, and subproject wiring.
- `build.sh`: interactive local helper for package builds.
- `YTLitePlus.xm`: main hooks and runtime feature toggles.
- `YTLitePlus.h`: shared imports, macros, interfaces, enums.
- `Source/*.xm`: feature-specific hooks.
- `.github/workflows/buildapp.yml`: CI build and release pipeline.

## 3) Cursor/Copilot Rules Status
- `.cursor/rules/`: not present.
- `.cursorrules`: not present.
- `.github/copilot-instructions.md`: not present.
- This file is the active in-repo agent instruction source.

## 4) Environment Prerequisites
- Prefer macOS for local build parity with CI.
- Install Theos and set `THEOS`.
- Install `theos-jailed` for sideload/rootless packaging.
- Required tools used in CI: `ldid`, `dpkg`, `make`, `jq`, `wget`, `unzip`.
- iOS SDK expected at `$(THEOS)/sdks/iPhoneOS<version>.sdk/`.
- Provide a decrypted `YouTube.ipa` (or `YouTube.app`) for local packaging.

## 5) Build Commands (Authoritative)

### Standard local package build
```bash
make package THEOS_PACKAGE_SCHEME=rootless IPA="/abs/path/YouTube.ipa" FINALPACKAGE=1
```

### Sideload/CI-style package build
```bash
make package SIDELOAD=1 THEOS_PACKAGE_SCHEME=rootless FINALPACKAGE=1 YTLITE_VERSION=<version>
```

### Interactive helper build
```bash
./build.sh
```

### Cleanup
```bash
make clean
make internal-clean
```

### Build notes
- `build.sh` prompts for IPA/app path; if blank, it auto-detects one `.ipa`/`.app` in repo root.
- `Makefile` may auto-download/extract YTLite assets via `before-all`.
- If changing bundle/app/version values, keep workflow logic in `.github/workflows/buildapp.yml` aligned.

## 6) Lint/Test Commands (Current State)
- There is no dedicated lint command configured.
- There is no native unit/integration test suite configured.
- There is no `make test` target in this repository.

### Quality gate to run after code changes
```bash
make package THEOS_PACKAGE_SCHEME=rootless IPA="/abs/path/YouTube.ipa" FINALPACKAGE=1
```

### Single-test guidance (important)
- There is currently no first-class command to run a single automated test.
- For “single test” validation today, run one focused manual scenario on-device after a successful build.
- Example scenario: toggle one feature key, exercise one matching UI path, verify behavior and no crash.
- In PR notes, state exactly which single scenario was validated.

### If tests are added later
- Add explicit test commands to this file.
- Include both full-suite and single-test syntax (for example `-only-testing` or test name filters).

## 7) Workflow Expectations for Agents
- Keep edits minimal and scoped; avoid broad refactors unless requested.
- Treat YouTube version compatibility as fragile; call out assumptions.
- Build after meaningful hook changes when environment allows.
- Do not modify unrelated files.
- Preserve release/build pipeline behavior unless task requires changing it.

## 8) Code Style Guidelines

### Imports and includes
- Use `#import` for Objective-C headers.
- Keep include order stable:
  1) Apple/system frameworks (`Foundation`, `UIKit`, etc.)
  2) C/system headers (`string.h`, `dlfcn.h`, etc.)
  3) Project headers (`YTLitePlus.h`, `Tweaks/...`, `Source/...`)
- Prefer central shared declarations in `YTLitePlus.h` for reused symbols.
- Avoid duplicate imports in touched files.

### Formatting and layout
- Use 4-space indentation; no tabs in new lines.
- Keep existing brace style:
  - Method/hook braces on same line.
  - Short guards may remain one-line.
- Preserve spacing around operators and ternaries.
- Keep hooks compact; split methods only when readability clearly improves.
- Prefer one logical responsibility per `%hook` block where practical.

### Logos/hooking conventions
- Use `%group` for optional feature sets controlled at runtime.
- Preserve `%orig` behavior unless intentionally replacing it.
- Fail safe in broad hooks; minimize side effects.
- Add compatibility comments when behavior is version-sensitive.
- Avoid indirect recursion in hooks; add guard logic when needed.

### Types and APIs
- Prefer Cocoa types (`BOOL`, `NSInteger`, `NSUInteger`, `CGFloat`, `NSString *`).
- Prefer typed collections where feasible (`NSArray<NSString *> *`).
- Use `static`/`static inline` helpers for local utilities.
- Use `NS_ENUM` for enum definitions.
- Use named constants for repeated strings/keys.

### Naming conventions
- Follow existing style:
  - Methods/functions: `camelCase`
  - Types/interfaces: `PascalCase`
  - Constants: `kPrefixName` or compile-time macro style
- Keep `NSUserDefaults` keys backward compatible.
- New defaults keys should be descriptive and stable.

### Error handling and safety
- Guard nil values before dereference.
- Prefer early return for invalid state.
- In uncertain hooks, prefer `%orig` over blocking base app behavior.
- For scripts/network steps, check exit codes and emit actionable errors.
- Avoid unsafe assumptions about ivars/selectors without checks.

### Performance and threading
- Use `dispatch_once` for shared initialization/singletons.
- Avoid heavy work in hot paths (`layoutSubviews`, frequently-called callbacks).
- Keep hook logic lightweight and cache when appropriate.
- Keep UIKit mutations on main thread.

### Comments and docs
- Comment only non-obvious or version-sensitive behavior.
- Keep comments short and focused on why.
- Remove stale TODOs and outdated compatibility notes.

## 9) Validation Checklist
- Build succeeds with standard `make package` command.
- No new warnings/errors introduced by touched files (as reported).
- At least one targeted manual validation scenario executed.
- New settings keys/defaults wired consistently.
- No accidental formatting churn in untouched code.

## 10) Commit/PR Hygiene
- Keep commits focused and descriptive.
- Mention user-visible changes and compatibility implications.
- State limitations if full local verification was not possible.
- Do not claim lint/tests passed when no such suite exists.
