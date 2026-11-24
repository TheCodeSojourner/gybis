---
name: gybis-check
kind: domain
description: Use for `/gybis-check`.
disable-model-invocation: true
---

λ gybis-check(x).
  purpose: Execute requirements, vocabulary, architecture, and specification checks in layer order
  | input: optional project root
  | output: separate domain reports ∧ downstream_pending_report
  | interaction: interactive
  | gate: project_root ∃

λ gybis-check_mode(m).
  m ∈ {interactive}
  | default: interactive
  | rationale: domain checks keep their own repair policy and may require human approval

λ gybis-check_state_machine(state, action).
  state ∈ {INIT, REQUIREMENTS, VOCABULARY, ARCHITECTURE, SPECS, COMPLETE}
  | transition(INIT → REQUIREMENTS) only_if(startup = true)
  | transition(REQUIREMENTS → VOCABULARY) only_if(req_result.reported = true ∧ req_result.valid = true)
  | transition(REQUIREMENTS → COMPLETE) only_if(req_result.reported = true ∧ req_result.valid = false)
  | transition(VOCABULARY → ARCHITECTURE) only_if(vocab_result.reported = true)
  | transition(VOCABULARY → COMPLETE) only_if(vocab_result.valid = false)
  | transition(ARCHITECTURE → SPECS) only_if(arch_result.reported = true)
  | transition(ARCHITECTURE → COMPLETE) only_if(arch_result.valid = false)
  | transition(SPECS → COMPLETE) only_if(spec_result.reported = true)
  | upstream_failure → report(pending_downstream_stages) ∧ transition(domain_state → COMPLETE) ∧ ¬ invoke(downstream_checks)

λ gybis-check_tool_guard(state, tool, path).
  state ∈ {REQUIREMENTS, VOCABULARY, ARCHITECTURE, SPECS} → allow(read(path))
  | state ∈ {REQUIREMENTS, VOCABULARY, ARCHITECTURE, SPECS} ∧ delegated_check.write_authorized = true
    → allow(write(path)) only_if(path ⊆ delegated_check.approval_scope)

λ gybis-check_pre_tool_check(state, tool, path).
  tool_guard(state, tool, path) = true ∨ halt("Tool not permitted in state " ⊕ state)
  | otherwise → deny(write(path))

λ gybis-check_fixed_point(result).
  invoke(result.owning_check) → verification
  | verification.valid = true → return(pass = true)
  | verification.valid = false → collect(remaining_errors) ∧ return(pass = false)

λ gybis-check_core(x).
  invoke(/gybis-req-check) → req_result
  | req_result.valid = false
      ? (report(req_result) ∧ downstream_pending(requirements) ∧ halt("Upstream requirement check failed"))
      : (invoke(/gybis-vocab-check) → vocab_result
         | vocab_result.valid = false
            ? (report(vocab_result) ∧ downstream_pending(vocabulary) ∧ halt("Upstream vocabulary check failed"))
            : (invoke(/gybis-arch-check) → arch_result
               | arch_result.valid = false
                  ? (report(arch_result) ∧ downstream_pending(architecture) ∧ halt("Upstream architecture check failed"))
                  : (invoke(/gybis-spec-check) → spec_result
                     | return(domain_reports: {req_result, vocab_result, arch_result, spec_result}))))

λ gybis-check_boundaries().
  ¬ inspect(lower_layer) when the owning upstream check fails
  | ¬ change(nearby_domain)
  | preserve(each_domain_check_repair_policy)
  | check repairs only the domain owned by the invoked check

λ gybis-check_regression_contract(x).
  invariant: each result remains separate
  | invariant: upstream failure produces pending downstream report, not implicit pass
  | invariant: all modified artifacts are re-checked by their owning check

λ gybis-check_deliver(report).
  report(domain_results) → stdout
  | handoff: failed domain → domain refine/tend/weed skill; unresolved upstream → pending downstream stages
