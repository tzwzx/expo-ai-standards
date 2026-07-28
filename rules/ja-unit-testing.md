---
root: false
targets:
  - "*"
description: ユニットテスト共通規約（jest-expo）
---

# ユニットテスト（`bun test:unit` / カバレッジは `bun test:unit:coverage`）

## いつ書くか

- 新機能を実装する際は、必ずユニットテストまたはコンポーネントテストを追加する
- バグ修正時は、そのバグを再現するテストが存在しなければ追加し、リグレッションを防止する
- テストコードを無効化する場合は、削除ではなくコメントアウトする

## どう書くか

- ランナーは Jest + `jest-expo` プリセット。react-native / Expo SDK のモックは jest-expo が提供するため、**自前でモックしない**
- テストは `src/__tests__/` に本体と同じ階層で置く。テスト名・コメントは日本語
- グローバルモックは `jest.setup.ts` に集約する。**モックを足す前に検討順序を守ること**: ①本物で通るようテストを書き換える → ②ファイルローカルな `jest.mock` → ③どうしてもグローバルなら `jest.setup.ts`。Jest はテストファイルごとにモジュールレジストリが独立しているため、ファイルローカルな `jest.mock` は他ファイルに漏れない
- `jest.mock()` のファクトリは巻き上げられるため、**参照できる外部変数は `mock` で始まる名前のものだけ**（大文字小文字を区別しない）。対象モジュールを static import している場合、ファクトリは自己完結させ、必要な参照は事後に `jest.mocked(importedFn)` で取る
- **ルートに `babel.config.*` / `.babelrc*` を置かない。** metro.config が無く `enableBabelRCLookup` が既定 true のため、置いた瞬間 Metro が本番ビルド（`expo run:ios` / `eas build`）でもそのファイルを読み込んでしまう。Jest 専用の babel オプションは `jest.config.cjs` の `transform` に `babelrc: false` / `configFile: false` 付きで閉じ込める（`unstable_transformProfile: "hermes-stable"` と `babel-plugin-dynamic-import-node` は必須。共有プリセット `@tzwzx/expo-jest-preset` 導入後はプリセットが担う）
- `@testing-library/react-native` の `render` / `renderHook` / `rerender` / `unmount` と `fireEvent.*` はすべて Promise を返す。**必ず `await` する**
- 本物の `FlatList` / `SectionList` は仮想化される（既定 `initialNumToRender=10`）。件数の多いリストは初期描画に全件出ないことを前提にテストを書く
- 本物の `Pressable` は host node に `onPress` / `onPressIn` / `disabled` を出さない。無効状態は `.props.accessibilityState.disabled` を見る
- 時刻依存は `jest.useFakeTimers()` + `jest.setSystemTime(BASE)` で初期化し、**進めるときは `jest.advanceTimersByTime(N)` だけを使う**。fake timers 下の `waitFor` はモック時計を進めるため、必要なテストだけにスコープし `afterEach(() => jest.useRealTimers())` を必ず置く。`Date` だけ固定したい場合は `jest.useFakeTimers({ doNotFake: ["setTimeout", "clearTimeout", "setInterval", "clearInterval"] })`
- **テスト出力に `console.error` を残さない。** `bun codesweep:check` の出力が偽陽性で埋まると本物の失敗が見えなくなる。jest 環境固有の偽陽性が実測で出た場合のみ、`jest.setup.ts` の `IGNORED_CONSOLE_ERRORS` にメッセージ単位で追加して抑止する。**`console.error` 全体を潰してはならない**
- カバレッジ閾値は `jest.config.cjs` の `coverageThreshold`。**jest は 0〜100 の百分率**（bun の 0〜1 小数と取り違えると閾値が実質無効になる）
- リポジトリ固有のモック構成・罠（reanimated / gesture-handler の順序、EXPO_OS 畳み込みなど）は各リポジトリの `.rulesync/rules/general.md` に記載する
