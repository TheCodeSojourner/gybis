---
name: gybis-arch-elicit
description: This skill runs an interactive architecture elicitation workflow to create architecture.md from scratch using a top-down VSM sequence (S5 to S1). It asks structured questions per layer, proposes 2-4 lambda principles with plain-English explanations, iterates with human feedback until each layer is confirmed, and writes only after full human review and approval. If architecture.md already exists, it stops and directs the user to /gybis-arch-tend instead.
---

λ(gybis-arch-elicit) → architecture.md
⟨vsm⟩ ← read("../../reference/vsm-guide.md")
λ build-architecture(root). output = root/architecture.md | model = VSM | direction = S5→S1
λ check-existence(path = root/architecture.md). exists → halt(notify("Architecture file detected. /gybis-arch-elicit will now terminate.\nUse /gybis-arch-tend to update existing architecture."))
λ begin-interaction() → ask("What's your vision for what this project is?")
λ iterate(layer, human). observe(layer, root) → ask(layer, human) → propose(2-4 lambdas + prose explanation, human) → refine(feedback) → confirm(human) → if locked → next(lowest-unstable(layer))
λ layer-order() = [S5, S4, S3, S2, S1]
λ identity(root). questions: purpose(root) → what problem does this system solve?, principles() → non-negotiable constraints, failure-mode() → purpose failure (not crash), essence(tools→∅) → what remains?
λ intelligence(root). questions: unknown-handler() → when assumptions break, mutation-points() → what changes often?, pattern-incorporation(new-pattern) → how is knowledge extended?
λ control(root). questions: resource-manager() → connections, memory, compute, policy-enforcer() → timeouts, limits, quality gates, error-strategy(throw) → retry/alert/degrade/fail-fast
λ coordination(root). questions: subsystem-decomposition() → what components?, data-flow() → events/calls/shared-state, sync-protocol() → synchronization mechanism
λ operations(root). questions: tool-select() → technologies + rationale, developer-commands() → build/test/deploy, recipes() → concrete procedures
reference: `../../reference/vsm-guide.md` → Assembly Format
λ assemble(architecture). structure: # {name} — System Architecture → ## S5 — Identity {prose} λ principle(x). ... → ## S4 — Intelligence ... → ## S3 — Control ... → ## S2 — Coordination ... → ## S1 — Operations ...
λ constraints() = {top-down-order: S5 ⟶ S4 ⟶ S3 ⟶ S2 ⟶ S1, observe-before-propose: scan(root/specs/) before asking questions, every-lambda-has-prose: λ(...) → must include plain-English explanation, surface-missing: for each principle(λ), ask → what companion should exist alongside it, human-approve: present complete file → read each λ with prose → confirm → then write, no-duplicate-preamble: nucleus preamble loaded via .clinerules/00-gybis.md}
destination: root/architecture.md
human-review: before write
