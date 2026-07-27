---
root: false
targets:
  - "*"
description: スタイリングルール
globs:
  - "src/**/*.ts"
  - "src/**/*.tsx"
---

# Styling

- All visual values (colors, spacing, sizes, radii, typography, etc.) must be managed as named constants in `src/theme.ts`. Do not use magic numbers or hard-coded hex colors in component files.
- Before adding a new constant, check if an existing one already covers the value.
