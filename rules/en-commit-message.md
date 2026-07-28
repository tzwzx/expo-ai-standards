---
root: false
targets:
  - "*"
description: Commit message convention (English, Conventional Commits)
# globs を持たない断片は Cursor では Manual 扱い（@ で呼ぶまで適用されない）になるため、
# 常時適用を明示する。Claude Code 側は paths なし = 常時ロードなので指定不要。
cursor:
  alwaysApply: true
---

## Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) and write **in English**. Mark breaking changes with `!` or a `BREAKING CHANGE:` footer. If the subject is not enough, leave a blank line and explain *why* in the body.
