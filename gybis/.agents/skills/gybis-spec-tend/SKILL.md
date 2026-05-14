---
name: gybis-spec-tend
description: This skill applies a specific new constraint to an existing Allium spec with minimal, targeted edits rather than reworking the whole file. It reads the spec, analyzes impacted rules/entities/transitions/config, runs focused elicitation only for the change scope, drafts a constrained diff, and validates with allium-check/allium-analyse before presentation. It writes only after human approval and then re-validates to confirm the spec remains healthy and consistent.
---

λ(gybis-spec-tend)
REF:../../gybis/reference/allium-language-reference.md
PURPOSE:given(specFile,constraint)→diff(minimal)∪write(specFile)
PF:¬(alliumSpec∧constraint)→err
S0:read(specFile,inFull)∥understand(entities,rules,transitionGraphs,config)
S1:analyze(constraint)|adding(newRule)∥modifying(existing)∥extending(entity/lifecycle)∥affectedLayers(entities∪rules∪transitions∪config)∥implied(requires,ensures,transitionEdges)∥conflictCheck(¬existingSpec)
S2:targetedElicit|invoke(elicit,constrainedToChange)∥ask(clarifyingQuestions→changeOnly,¬reEllicitWholeSpec)
S3:diff|newRules(verbNoun)∥newFields(when-clauses)∥newTransitions(→transitionsBlock)∥whenObligations(satisfied)∥¬modify(existing,unlessRequired)∥update(traces:if implFiles known)
S4:validate|allium-check(specFile)→exit0∥allium-analyse(specFile)→exit0∥apply(actioning-findings)∥translate(diagnostics→fixes)
S5:presentDiff∧human(approve)∥write(specFile)∥allium-check(specFile)∧allium-analyse(specFile)→confirm(health)
INV≡minimalChange(¬refactorUnrelated)∥newInconsistencies→catchBeforePresent∥humanApproves∀writes∥postWrite→revalidate∥verbNoun(rules)∥when-clauses(fields)∥transitionGraph(updated)
μ≡S0⋅S1⋅S2⋅S3⋅S4⋅S5
plain:given(file,constraint)→read∥analyze(constraint)∥elicit(targeted)∥draft(diff)∥validate(check+analyse)∥present∧approve∥write∥revalidate
