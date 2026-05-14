---
name: gybis-memory-describe
description: This skill reads one or more Allium spec files (file, domain, or all specs) and translates them into clear, non-technical prose focused on user-visible behavior. It explains what exists, what users can do, governing rules, and guarantees, then outputs a coherent narrative with light headings and no code, jargon, or implementation detail. It stays read-only, surfaces gaps as questions for the human, and reports clearly when specs are missing or empty.
---

λ(gybis-spec-describe)
REF:../../gybis/reference/allium-language-reference.md
PURPOSE:given(file|domain)→prose(non-technical)∪readOnly
PF:¬alliumFiles→say(emptyOrMissing)
S0:files←parse(input)|file→read(single)∥domain→read(all,<root>/specs/{domain}/*.allium)∥noArg→read(all,<root>/specs/**/*.allium)∥ambiguous→ask(human,clarify)
S1:prose←Σ{
  whatExists:concepts(purpose,plainLang)∧¬entityNames,
  userCanDo:actions(outcomes)∧triggers∧userExperience,
  rules:businessRules=constraints∧prevents∧requires∧edgeCases,
  guarantees:invariants=userFacingPromises∧terminalStates=lifecycleEndpoints,
  domainMode:connections(userJourney,otherSpecs)
}
S2:output(flowingProse,lightHeadings,¬code,¬techSyntax,¬jargon,¬implDetails)
INV≡∀userVisibleBehavior→output∥¬omit∥multiFile→coherentNarrative∥gaps→questions(human)∥readOnly∥empty→saySo
μ≡S0⋅S1⋅S2
plain:given(scope)→resolve(files)∥read(inFull)∥translate(plainLang)∥output(prose,lightHeadings)∥surface(gaps)
