---
root: false
targets:
  - "*"
description: Baseline rules for any code change
# globs を持たない断片は Cursor では Manual 扱い（@ で呼ぶまで適用されない）になるため、
# 常時適用を明示する。Claude Code 側は paths なし = 常時ロードなので指定不要。
cursor:
  alwaysApply: true
---

## Baseline rules for code changes

Keep every change scoped to what was asked. Unless explicitly instructed, leave design (appearance, layout, colors, fonts), styling (CSS, `style` props), UI arrangement, and existing behavior untouched.
