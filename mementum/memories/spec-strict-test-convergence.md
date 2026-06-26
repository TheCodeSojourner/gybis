💡 spec-propagate and spec-weed should converge on passing tests, not only structural consistency

When skills mutate implementation/tests, structural/spec checks are necessary but insufficient for completion.

For `/gybis-spec-propagate` and `/gybis-spec-weed`, completion criteria should include strict test execution (`test_suite_passes = true`) with an explicit test-running state and loop-back on failures.

This preserves SDD authority while preventing false convergence where artifacts look aligned but behavior still fails under the project test runner.

Pattern: resolve test command from architecture S1 first, fall back to repository conventions, and halt explicitly when no command can be resolved.