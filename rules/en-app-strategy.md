---
root: false
targets:
  - "*"
description: Collaboration contract with the cross-app strategy repo (app-strategy)
# globs を持たない断片は Cursor では Manual 扱い（@ で呼ぶまで適用されない）になるため、
# 常時適用を明示する。Claude Code 側は paths なし = 常時ロードなので指定不要。
cursor:
  alwaysApply: true
---

## Working with the cross-app strategy repo (app-strategy)

Cross-app strategy, priorities, and measured results are managed in
`/Users/tazawa/Documents/Obsidian/Core/アプリ戦略/` (this app's `docs/` is the source of truth
for app-specific matters; app-strategy is the source of truth across the business).

### Read (reference)

- **Always read before proposing an initiative**: `portfolio/priorities.md` (check this app's
  current-phase focus; when proposing something outside that focus, say so explicitly)
- **Recommended reading**: `portfolio/strategy.md` (business direction), `portfolio/apps.md`
  (ledger of all apps), `revenue/model.md` (revenue assumptions), `playbooks/` (cross-app
  knowledge — read the relevant file especially before new-feature, monetization, ASO, or
  release work)

### Write (sync)

- **Update within the same work session as a release or status change**: the facts columns of
  `portfolio/apps.md` (current version / status / store URL / release date; also set the
  frontmatter `updated` to today)
- **Append cross-app learnings**: the learnings log at the end of the relevant `playbooks/`
  file (note the source app and the date)
- **When a new app ships**: update the cross-promo array (`other-apps.ts`) of every existing
  app the same day, and update `portfolio/apps.md`

### Do not write (propose only)

- `portfolio/priorities.md` / `portfolio/strategy.md` / `portfolio/roadmap.md` /
  `revenue/model.md` must not be edited directly. To change them, present a proposal to the
  owner (before / after / reason)
- `decisions/`, `revenue/snapshots/`, and `portfolio/metrics.md` are not writable (only
  sessions opened in app-strategy update them)
- Do not duplicate app-specific specs or ASO copy into app-strategy (the source of truth is
  this repo's `docs/`)
