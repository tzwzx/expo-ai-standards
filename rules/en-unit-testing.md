---
root: false
targets:
  - "*"
description: Shared unit-testing conventions (jest-expo)
---

## Unit testing (`bun test:unit`, coverage via `bun test:unit:coverage`)

- Runner is Jest + the `jest-expo` preset. react-native / Expo SDK mocks are provided by jest-expo, so do not mock them yourself.
- To disable test code, comment it out rather than deleting it.
- Test names and comments are written in Japanese — this rule file is in English for the agent's benefit, which does not change the language of the code.
- Global mocks go in `jest.setup.ts`. Before adding a mock, follow the order: (1) rewrite the test to pass against the real module, (2) a file-local `jest.mock`, (3) only then a global mock. Jest keeps a separate module registry per test file, so a file-local mock does not leak to other files.
- Do NOT place a `babel.config.*` / `.babelrc*` at the repo root. There is no `metro.config.*`, so `enableBabelRCLookup` defaults to true and Metro would read it during real builds (`expo run:ios` / `eas build`). Jest-only babel options are confined to `jest.config.cjs`'s `transform` with `babelrc: false` / `configFile: false` (`unstable_transformProfile: "hermes-stable"` and `babel-plugin-dynamic-import-node` are required; once the shared preset `@tzwzx/expo-jest-preset` is adopted, the preset owns this).
- `@testing-library/react-native`'s `render` / `renderHook` / `rerender` / `unmount` and `fireEvent.*` all return Promises. Always `await`.
- The real `FlatList` / `SectionList` are virtualized (default `initialNumToRender=10`). Write tests assuming long lists do not fully render initially.
- The real `Pressable` does not expose `onPress` / `onPressIn` / `disabled` on the host node. Check disabled state via `.props.accessibilityState.disabled`.
- For time-dependent tests, initialize with `jest.useFakeTimers()` + `jest.setSystemTime(BASE)` and advance only with `jest.advanceTimersByTime(N)`. `waitFor` under fake timers advances the mocked clock, so scope fake timers to the tests that need them and always add `afterEach(() => jest.useRealTimers())`. To freeze only `Date`, use `jest.useFakeTimers({ doNotFake: ["setTimeout", "clearTimeout", "setInterval", "clearInterval"] })`.
- Leave no `console.error` in test output. Once `bun codesweep:check`'s output fills up with false positives, real failures become invisible. Suppress a message only after observing that it is a jest-environment-specific false positive, and only per-message via `IGNORED_CONSOLE_ERRORS` in `jest.setup.ts`; never silence `console.error` wholesale.
- Coverage thresholds live in `jest.config.cjs`'s `coverageThreshold`. Jest uses 0-100 percentages, not bun's 0-1 fractions — confusing the two silently disables the threshold.
- Repo-specific mock setups and pitfalls (reanimated / gesture-handler ordering, EXPO_OS constant folding, etc.) belong in each repository's `.rulesync/rules/general.md`.
