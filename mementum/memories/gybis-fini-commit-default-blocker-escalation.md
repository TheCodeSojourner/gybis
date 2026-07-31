💡 Make gybis-fini commit by default and escalate blockers explicitly

When gybis-fini includes a commit step, set commit as the default action and only skip for strong blockers (explicit no-commit instruction, unresolved merge/index conflict, git failure needing human action, or policy/safety conflict).

If blocked, never silently skip commit. Report the blocker with evidence and ask the operator to choose one of three actions:
1) retry commit now,
2) skip commit for this session,
3) manual user commit.

This prevents instruction-precedence drift from disabling closeout commits unexpectedly while preserving safety on true blockers.
