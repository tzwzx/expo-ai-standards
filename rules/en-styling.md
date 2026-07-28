---
root: false
targets:
  - "*"
description: Styling rules
globs:
  - "src/**/*.ts"
  - "src/**/*.tsx"
---

# Styling

- **Every value that affects appearance (colors, spacing, sizes, corner radii, typography, etc.) belongs in the theme as a named constant.** It lives in `src/theme.ts` or under `src/theme/` (which one depends on the repository). Never inline magic numbers or raw hex / rgba values in a component file.
- **Never create a file-local color constant outside the theme.** That is how one value ends up scattered across files — `WHITE_ALPHA_60` was once duplicated across two of them.
- Before adding a new constant, check whether an existing one already covers the same value.
- Values that differ between light and dark belong in each theme object. Values that are fixed regardless of theme (text sitting on a colored surface, the scrim behind a sheet) belong in the fixed-color object.
- **When extracting an existing color into a constant, verify the value did not change.** Notation differences (`0.5` vs `0.50`, `#000` vs `#000000`) may be collapsed into a single constant.
