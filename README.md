# expo-ai-standards

Expo アプリ群で共有する AI エージェント用ルールの正本リポジトリ。

各リポジトリの [rulesync](https://github.com/dyoshikawa/rulesync) から**宣言的ソース**として参照される。ルールの重複コピペと文言ドリフトを構造的に防ぐのが目的。

## レイアウト

```
rules/    … 共有ルール断片（rulesync sources の rules。フラット配置・直下の .md のみ認識される）
  en-*.md … AI しか読まないルール（既定）
  ja-*.md … 生成物が日本語であることを要求するルール（現状 commit-message のみ）
```

prefix は**その断片が何語で書かれているか**を示すだけで、リポジトリの運用言語ではない。
ルール本文は人間が読み返さないので、**英語で書くのを既定とする**（同じ内容で日本語の 1.5〜2 倍のトークンを食うため）。

**ルールの記述言語と、AI に書かせるコードの言語は別。** `en-*` に「テスト名・コメントは英語で」の
ような指定を紛れ込ませると、英語化したつもりが生成コードの規約まで書き換わる。
生成物の言語は各断片の本文で明示すること（`en-unit-testing` はコメントを日本語と明記している）。

**断片の見出しは `##` 始まりで書く。** 断片は各リポの root ルール（`.rulesync/rules/general.md`）と
並ぶ形で出力され、rulesync は見出しレベルの正規化をしない。`#` で始めると root の h1 と衝突し、
後続の `##` 始まりの断片が直前の h1 の**子セクションとして読める構造**になる
（実際にコミットメッセージ規約がユニットテストの配下に落ちていた。2026-07-29 に是正）。

**特定のファイル群でしか使わない断片には `globs` を付ける。** rulesync が各ツールの条件付きロード機構へ
変換する（実測確認済み）:

| ツール | root の出力先 | 非 root の出力先 | `globs` の変換先 |
| --- | --- | --- | --- |
| claudecode | `CLAUDE.md` | `.claude/rules/<name>.md` | `paths:`（該当ファイルを読んだときだけロード） |
| cursor | `.cursor/rules/<name>.mdc` | 同左 | `globs:` |

Claude Code の公式ドキュメントも、CLAUDE.md が大きくなったら path スコープ付きルールへ逃がすことを
推奨している。**`@AGENTS.md` のようなインポートは整理にはなるがコンテキストは減らない**（起動時に全文
ロードされる）ため、削減したいなら `globs` を使う。

> **`codexcli` は使わない。** Codex CLI は非 root ルールを `AGENTS.md` へ全文 fold する仕様で、
> 条件付きロード機構を持たない。実際に使うのは Claude Code と Cursor だけなので、同じ内容を
> 抱えるだけの `AGENTS.md` は 2026-07-29 に全リポで廃止した。復活させる場合、Codex 側は
> 常に全文を読む前提になる。

**`globs` を持たない断片には `cursor: { alwaysApply: true }` を付ける。** Cursor は `globs` も
`alwaysApply` も無い `.mdc` を Manual 扱いにし、`@` で明示的に呼ぶまで適用しない。Claude Code は
`paths` 無し = 常時ロードなので指定は要らず、この一行は Cursor のためだけにある。
**各リポの `general.md`（root）にも必要**（root であっても Cursor では同列の `.mdc` として出るため）。

**スキルはここでは配らない。** rulesync sources の `skills` は使わず、
リポジトリ固有のスキルは各リポの `.rulesync/skills/` に置く（理由は「運用ルール」参照）。

## 使い方（消費側）

各アプリの `rulesync.jsonc` に sources を追加して `bunx rulesync install`:

```jsonc
{
  "targets": {
    // rules を有効にすると root は CLAUDE.md / .cursor/rules へ、非 root は
    // .claude/rules / .cursor/rules へ分かれて出る（globs による条件付きロードが効く）
    "cursor": ["rules", "hooks"],
    "claudecode": ["rules", "hooks", "skills"],
  },
  "sources": [
    {
      // ⚠️ github transport（"tzwzx/expo-ai-standards" 形式）は未認証 API の
      // レート制限で失敗するため、git transport + https URL を使う
      "source": "https://github.com/tzwzx/expo-ai-standards.git",
      "transport": "git",
      "rules": [
        "en-critical-rules",
        "en-unit-testing",
        "ja-commit-message",
      ],
    },
  ],
  "delete": true,
}
```

同名で ja / en の両方がある断片（現状 `commit-message` のみ）は、どちらか一方だけを選ぶ。
両方を並べると同じ規約が二重に合成される。

なお `commit-message` の ja / en は**生成されるコミットメッセージ自体の言語**を切り替えるもので、
`unit-testing` のような「ルール本文の記述言語」の話とは別物。日本語でコミットするなら
`ja-commit-message` を選ぶ（本文が日本語なのはその副次的な結果にすぎない）。

- 取得結果は `.rulesync/rules/.curated/` に展開され（gitignore 対象）、`rulesync generate` で `CLAUDE.md` / `.claude/rules/` / `.cursor/rules/` へ出力される
- `rulesync.lock` はコミットする（commit SHA + integrity で再現性を担保）
- CI では `bunx rulesync install --frozen` + `bunx rulesync generate --check` でドリフト検知
- 各リポの `postinstall` 推奨形: `lefthook install && bunx rulesync install && bun rulesync`

## .cursorrules ブリッジ（コミットメッセージ規約）

Cursor のコミットメッセージ生成（ソース管理パネルの✨）に効くのはレガシーな `.cursorrules` **だけ**（`.cursor/rules` / AGENTS.md は対象外。Cursor スタッフ確認済み・2026-07 時点）。rulesync は `.cursorrules` を生成できないため、各リポの `.rulesync/scripts/post-generate.sh` で正本から生成する:

```bash
# ja-commit-message.md（英語運用なら en-）の frontmatter を剥がして .cursorrules を生成
awk 'BEGIN{n=0} /^---$/{n++; next} n>=2{print}' \
  .rulesync/rules/.curated/ja-commit-message.md > .cursorrules
```

`.cursorrules` は各リポで**コミットする**。

## 変更したときの確認

ここの変更は **Expo アプリ7本と expo-oxc-config に一斉に効く**。消費側で実際に合成されるところまで確認する。

```bash
cd ../<いずれかのアプリ>
bunx rulesync install --update   # 正本を取り直す
bun rulesync                     # generate + post-generate.sh

# 生成物は gitignore 対象なので git diff には出ない。中身を直接見る
cat CLAUDE.md .claude/rules/*.md
head -4 .cursor/rules/*.mdc      # alwaysApply / globs が付いているか

git diff                         # .cursorrules / rulesync.lock に反映されたか
```

`globs` を付けた断片は、`claude -p` で実機確認できる（該当ファイルを読む前後で切り替わるはず）:

```bash
claude -p "Answer YES or NO only, use no tools: is <ルールの一文> in your context?" --allowedTools ""
```

`rules/` のファイルを増やした場合、各アプリの `rulesync.jsonc` の `sources[].rules` に
名前を足さないと**取り込まれないまま静かに無視される**（エラーにならない）。

## 運用ルール

- ルールを変えたいときは**このリポジトリを変更**し、各リポで `bunx rulesync install --update` する（生成物や各リポのコピーを直接編集しない）
- **`post-generate.sh` で `CLAUDE.md` を上書きしない。** rulesync が claudecode target で生成する root ルールを潰すと `.claude/rules/` の path スコープが効かなくなる。以前 `printf '@AGENTS.md\n' > CLAUDE.md` というインポートブリッジを置いており、全文 fold された `AGENTS.md` がロードされて条件付きロードが無効化されていた（2026-07-29 に撤去）
- **スキルはここに置かない。** ツール由来のスキル／コマンドはそのツール自身に生成させる（例: `store-shots` の `.claude/commands/store-shots.md` は `bunx store-shots init` が書き出す）。ここへコピーすると、生成元が更新されても追従しない劣化コピーになる（実際に 2026-07 に store-shots で発生し、2026-07-29 に撤去した）
- 各リポ固有の gotcha（固有のモック構成・罠・データ規約など）は各リポ側に置く。ここには**フリート全体に当てはまるものだけ**を置く
- **各リポ側でも、特定のファイル群でしか要らないものは `globs` 付きの非 root 断片に分ける。** `general.md`（root）は毎ターン全文ロードされるので、そこに置いてよいのは「どの作業でも要る」ものだけ。対応する共有断片がある場合は **`globs` を揃える**（例: 各リポの `.rulesync/rules/unit-testing.md` は `en-unit-testing` と同じ `globs` を持ち、テストを開いた時に2つが並んで載る。リポ固有の事情でファイルが増えるなら `tsconfig*.json` / `jest/**` などを足す）。2026-07-29 に Expo アプリ7本 + 雛形のユニットテスト節をこの形へ分離した
- **ツール（lint / formatter / 型）で担保できるルールは置かない。AI が既存コードを読めば分かることも置かない。** ここに残す価値があるのは「破ると静かに壊れる」実測ベースの罠だけ。実測の裏付けを失ったルールは撤去する（styling ルールは根拠にしていた定数の重複が解消済みで、かつ書いてあっても違反が残っていたため 2026-07-29 に撤去した）
- **一般知識の再掲を書かない。合成先の AGENTS.md は毎ターン読まれる。** Conventional Commits の型一覧・フォーマット・例のように AI が確実に知っている仕様解説は、そのぶん常時コンテキストを食うだけで効果がない。commit-message は 56 行あり AGENTS.md の 45〜60% を占めていたが、プロジェクト固有なのは「日本語で書く」の 1 点だけだったため 2026-07-29 に 3 行へ圧縮した。
  一方 unit-testing は同じ基準で見直しても 16 項目中 12 項目が実測ベースの罠として残った（`babel.config` の設置禁止、`render` が Promise を返す、`FlatList` の仮想化など）。**行数の多さ自体は削減の理由にならない。判断するのは中身が一般知識かどうか**
- **新しい断片は `en-` で書く。** 対訳（`ja-`）は作らない。生成物の言語を切り替える必要がある断片（`commit-message` のような）に限って対を用意し、その場合は片方だけ更新せず同じ PR で両方を更新する
- タグ（v1.0.0 など）でバージョニングし、消費側は `tzwzx/expo-ai-standards@v1.0.0` 形式で固定してもよい
