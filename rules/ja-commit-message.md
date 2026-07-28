---
root: false
targets:
  - "*"
description: コミットメッセージ規約（日本語・Conventional Commits）
# globs を持たない断片は Cursor では Manual 扱い（@ で呼ぶまで適用されない）になるため、
# 常時適用を明示する。Claude Code 側は paths なし = 常時ロードなので指定不要。
cursor:
  alwaysApply: true
---

## コミットメッセージ

[Conventional Commits](https://www.conventionalcommits.org/ja/v1.0.0/) に準拠し、**日本語で**書く。破壊的変更は `!` かフッターの `BREAKING CHANGE:` で明示する。タイトルで足りなければ 1 行空けて本文に「なぜ」を書く。
