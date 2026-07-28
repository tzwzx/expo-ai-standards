# expo-ai-standards

Expo アプリ群で共有する AI エージェント用ルールの正本リポジトリ。

各リポジトリの [rulesync](https://github.com/dyoshikawa/rulesync) から**宣言的ソース**として参照される。ルールの重複コピペと文言ドリフトを構造的に防ぐのが目的。

## レイアウト

```
rules/    … 共有ルール断片（rulesync sources の rules。フラット配置・直下の .md のみ認識される）
  ja-*.md … 日本語で運用するリポジトリ向け
  en-*.md … 英語で運用するリポジトリ向け
```

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
        "ja-critical-rules",
        "ja-unit-testing",
        "ja-styling",
        "ja-commit-message",
      ],
    },
  ],
  "delete": true,
}
```

英語で運用するリポジトリは `en-*` を選択する:

```jsonc
"rules": ["en-unit-testing", "en-commit-message"]
```

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
- ja / en は対の内容を保つ。片方だけ更新したら、もう片方も同じ PR で更新する
- タグ（v1.0.0 など）でバージョニングし、消費側は `tzwzx/expo-ai-standards@v1.0.0` 形式で固定してもよい
