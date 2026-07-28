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

**断片の見出しは `##` 始まりで書く。** rulesync は各リポの root ルール（`.rulesync/rules/general.md`）に
非 root ルールを**連結するだけ**で、見出しレベルの正規化もセクション区切りの挿入もしない。
`#` で始めると AGENTS.md に h1 が複数生まれ、さらに後続の `##` 始まりの断片が直前の h1 の
**子セクションとして読める構造**になる（実際にコミットメッセージ規約がユニットテストの配下に
落ちていた。2026-07-29 に是正）。

**特定のファイル群でしか使わない断片には `globs` を付ける。** rulesync が各ツールの条件付きロード機構へ
変換する（実測確認済み）:

| ツール | root の出力先 | 非 root の出力先 | `globs` の変換先 |
| --- | --- | --- | --- |
| claudecode | `CLAUDE.md` | `.claude/rules/<name>.md` | `paths:`（該当ファイルを読んだときだけロード） |
| cursor | `.cursor/rules/<name>.mdc` | 同左 | `globs:` |
| codexcli | `AGENTS.md` | **root へ fold** | 効かない（常に全文） |

Claude Code の公式ドキュメントも、CLAUDE.md が大きくなったら path スコープ付きルールへ逃がすことを
推奨している。**`@AGENTS.md` のようなインポートは整理にはなるがコンテキストは減らない**（起動時に全文
ロードされる）ため、削減したいなら `globs` を使う。ただし Codex CLI には効かないので、
Codex 側は常に全文を読む前提で書く。

**スキルはここでは配らない。** rulesync sources の `skills` は使わず、
リポジトリ固有のスキルは各リポの `.rulesync/skills/` に置く（理由は「運用ルール」参照）。

## 使い方（消費側）

各アプリの `rulesync.jsonc` に sources を追加して `bunx rulesync install`:

```jsonc
{
  "targets": {
    "cursor": ["hooks"],
    "claudecode": ["hooks", "skills"],
    "codexcli": ["rules", "hooks"],
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

- 取得結果は `.rulesync/rules/.curated/` に展開され（gitignore 対象）、`rulesync generate` で AGENTS.md 等に合成される
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

ここの変更は **Expo アプリ7本すべてに一斉に効く**。消費側で実際に合成されるところまで確認する。

```bash
cd ../<いずれかのアプリ>
bunx rulesync install --update   # 正本を取り直す
bun rulesync                     # generate + post-generate.sh
git diff                         # AGENTS.md / .cursorrules に意図どおり反映されたか
```

`rules/` のファイルを増やした場合、各アプリの `rulesync.jsonc` の `sources[].rules` に
名前を足さないと**取り込まれないまま静かに無視される**（エラーにならない）。

## 運用ルール

- ルールを変えたいときは**このリポジトリを変更**し、各リポで `bunx rulesync install --update` する（生成物や各リポのコピーを直接編集しない）
- **スキルはここに置かない。** ツール由来のスキル／コマンドはそのツール自身に生成させる（例: `store-shots` の `.claude/commands/store-shots.md` は `bunx store-shots init` が書き出す）。ここへコピーすると、生成元が更新されても追従しない劣化コピーになる（実際に 2026-07 に store-shots で発生し、2026-07-29 に撤去した）
- 各リポ固有の gotcha（固有のモック構成・罠・データ規約など）は各リポの `.rulesync/rules/general.md` に置く。ここには**フリート全体に当てはまるものだけ**を置く
- **ツール（lint / formatter / 型）で担保できるルールは置かない。AI が既存コードを読めば分かることも置かない。** ここに残す価値があるのは「破ると静かに壊れる」実測ベースの罠だけ。実測の裏付けを失ったルールは撤去する（styling ルールは根拠にしていた定数の重複が解消済みで、かつ書いてあっても違反が残っていたため 2026-07-29 に撤去した）
- **一般知識の再掲を書かない。合成先の AGENTS.md は毎ターン読まれる。** Conventional Commits の型一覧・フォーマット・例のように AI が確実に知っている仕様解説は、そのぶん常時コンテキストを食うだけで効果がない。commit-message は 56 行あり AGENTS.md の 45〜60% を占めていたが、プロジェクト固有なのは「日本語で書く」の 1 点だけだったため 2026-07-29 に 3 行へ圧縮した。
  一方 unit-testing は同じ基準で見直しても 16 項目中 12 項目が実測ベースの罠として残った（`babel.config` の設置禁止、`render` が Promise を返す、`FlatList` の仮想化など）。**行数の多さ自体は削減の理由にならない。判断するのは中身が一般知識かどうか**
- **新しい断片は `en-` で書く。** 対訳（`ja-`）は作らない。生成物の言語を切り替える必要がある断片（`commit-message` のような）に限って対を用意し、その場合は片方だけ更新せず同じ PR で両方を更新する
- タグ（v1.0.0 など）でバージョニングし、消費側は `tzwzx/expo-ai-standards@v1.0.0` 形式で固定してもよい
