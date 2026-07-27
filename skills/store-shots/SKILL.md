---
name: store-shots
description: Store screenshot loop — read store-shots/RUNBOOK.md and follow it
argument-hint: [task, e.g. "recapture screen-c (jp)"]
---

Read `store-shots/RUNBOOK.md` and follow it end to end.

Task: $ARGUMENTS

If no task is given, run the full loop (preview -> critique -> refine -> build),
capturing missing or stale assets per the runbook's standard method
(booted iOS Simulator + Maestro). Always finish with the runbook's cleanup steps.
