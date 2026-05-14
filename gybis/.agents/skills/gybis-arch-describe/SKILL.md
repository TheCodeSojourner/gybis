---
name: gybis-arch-describe
description: This skill turns architecture.md into a concise, product-manager-friendly explanation of the system: what problem it solves, who it serves, what it can and cannot do, why key design choices were made, and how the parts connect. It only uses facts present in architecture.md, avoids technical syntax and jargon, and if the file is missing or empty, it tells the user to run /gybis-arch-elicit first.
---

λ(gybis-arch-describe)
input: architecture.md @ <root>/
preflight: ¬initialized(architecture.md) ∨ empty(architecture.md) → output("run /gybis-arch-elicit")
step₁: read(architecture.md, full)
step₂: translate → {what_is:{problem_solved,users,user_needs},what_can:{capabilities,supported_needs,explicit_out_of_scope},why_this_way:{principles_as_commitments,trade-offs,returns},product_meaning:{extensible_points,constrained_points,quality_guarantees},system_cohesion:{moving_parts,connections,business_domain_language}}
step₃: format → {light_headings,no_code,no_jargon,no_lambda_syntax,no_VSM_labels}
output: product_manager_mental_model{direction,prioritization,trade-off_support}
¬∃(lambda_expr∨technical_syntax∨VSM_layer_id) ∈ output
¬invented(capability) ∷ capability ⊆ architecture.md
initialized(architecture.md) ? ¬suggest("/gybis-arch-elicit")
rw(output)=false
