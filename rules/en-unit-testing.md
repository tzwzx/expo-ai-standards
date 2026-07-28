---
root: false
targets:
  - "*"
description: Shared unit-testing conventions (jest-expo)
---

# Unit testing (`bun test:unit`, coverage via `bun test:unit:coverage`)

## When to write tests

- Always add a unit test or component test when implementing a new feature.
- When fixing a bug, add a test that reproduces it if one does not already exist, to prevent regressions.
- To disable test code, comment it out rather than deleting it.

## How to write tests

- Runner is Jest + the `jest-expo` preset. react-native / Expo SDK mocks are provided by jest-expo, so do not mock them yourself.
- Tests live under `src/__tests__/`, mirroring `src/`. Test names and comments are in English (matching the rest of this repo).
- Global mocks go in `jest.setup.ts`. Before adding a mock, follow the order: (1) rewrite the test to pass against the real module, (2) a file-local `jest.mock`, (3) only then a global mock. Jest keeps a separate module registry per test file, so a file-local mock does not leak to other files.
- `jest.mock()` factories are hoisted, so they can only reference variables whose names start with `mock` (case-insensitive). If the mocked module is statically imported, keep the factory self-contained and grab references afterwards with `jest.mocked(importedFn)`.
- Do NOT place a `babel.config.*` / `.babelrc*` at the repo root. There is no `metro.config.*`, so `enableBabelRCLookup` defaults to true and Metro would read it during real builds (`expo run:ios` / `eas build`). Jest-only babel options are confined to `jest.config.cjs`'s `transform` with `babelrc: false` / `configFile: false` (`unstable_transformProfile: "hermes-stable"` and `babel-plugin-dynamic-import-node` are required; once the shared preset `@tzwzx/expo-jest-preset` is adopted, the preset owns this).
- `@testing-library/react-native`'s `render` / `renderHook` / `rerender` / `unmount` and `fireEvent.*` all return Promises. Always `await`.
- The real `FlatList` / `SectionList` are virtualized (default `initialNumToRender=10`). Write tests assuming long lists do not fully render initially.
- The real `Pressable` does not expose `onPress` / `onPressIn` / `disabled` on the host node. Check disabled state via `.props.accessibilityState.disabled`.
- For time-dependent tests, initialize with `jest.useFakeTimers()` + `jest.setSystemTime(BASE)` and advance only with `jest.advanceTimersByTime(N)`. `waitFor` under fake timers advances the mocked clock, so scope fake timers to the tests that need them and always add `afterEach(() => jest.useRealTimers())`. To freeze only `Date`, use `jest.useFakeTimers({ doNotFake: ["setTimeout", "clearTimeout", "setInterval", "clearInterval"] })`.
- Leave no `console.error` in test output. Suppress jest-environment-specific false positives only per-message via `IGNORED_CONSOLE_ERRORS` in `jest.setup.ts`; never silence `console.error` wholesale.
- Coverage thresholds live in `jest.config.cjs`'s `coverageThreshold`. Jest uses 0-100 percentages (bun used 0-1 fractions) — do not confuse them.
- Repo-specific mock setups and pitfalls belong in each repository's `.rulesync/rules/general.md`.
