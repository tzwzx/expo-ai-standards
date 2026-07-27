---
root: false
targets:
  - "*"
description: パッケージ管理ルール
globs:
  - package.json
---

# パッケージ管理

新しいパッケージを追加する際は、常に `-E`（または `--save-exact`）を使ってバージョンを固定する。
