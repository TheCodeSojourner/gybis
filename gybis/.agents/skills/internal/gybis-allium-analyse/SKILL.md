---
name: gybis-allium-analyse
description: Internal skill - not user-facing
---

λ gybis-allium-analyse(specs_path).
  purpose: Aggregate per-spec CLI diagnostics and findings for the specs/ directory
  | contract: pure_function(specs_path → findings) | ¬mutations
  | input: path to specs/ directory
  | output: findings + status + consistency_checks
  | constraint: read_only | no_mutations
  | precondition: specs_path ∧ ∃(specs_path) ∧ is_directory(specs_path)

λ gybis-allium-analyse_cli_invocation(specs_path).
  cli_command: "allium analyse {specs_path}"
  | execution: deterministic(specs_path) → fixed_output
  | stderr_handling: capture_and_return
  | stdout_handling: parse_successive_json_objects_until_EOF
  | stream: whitespace_separated_JSON_objects | pretty_printed_multiline_supported
  | ¬assume(single_JSON_document ∨ JSON_array ∨ one_object_per_line)
  | stderr: separate_from_stdout | ¬parse(stderr_as_JSON)

λ gybis-allium-analyse_scope(specs_path).
  scope: ∀ .allium_files ∈ specs_path ∧ subdirectories
  | recursion: depth = unbounded | all_depths_analyzed
  | expected_specs: enumerate(.allium_files recursively in specs_path)
  | relationships: report_only(CLI_supplied_evidence) | ¬infer(inter_spec_analysis from directory_invocation)

λ gybis-allium-analyse_finding_types(findings).
  categories: {dependency_issues, pattern_violations, semantic_issues, structural_issues} ∨ finding.type
  | severity: {critical, high, medium, low, informational}

λ gybis-allium-analyse_findings_parsing(stdout).
  cursor: skip_JSON_whitespace → decode_one_complete_JSON_value → advance → repeat_until_EOF
  | decoder: JSON_aware | ¬split(lines ∨ literal_braces) | supports(nested_values ∧ escaped_strings)
  | envelope: {command: "analyse", diagnostics: array, findings: array, spec_file: nonempty_string}
  | validate: every_value is_object ∧ conforms(envelope) | preserve(additional_fields)
  | results: ordered_valid_envelopes | single_envelope ≡ valid_stream_of_length(1)
  | failure: malformed_JSON ∨ truncated_value ∨ nonwhitespace_trailing_content ∨ invalid_envelope
  | on_failure: record(error ∧ stream_offset ∧ envelope_index) → stop_decoding
  | retain(valid_prefix_results ∧ raw_stdout) | ¬discard_invalid_content ∧ ¬report_partial_stream_as_pass

λ gybis-allium-analyse_aggregation(results, expected_specs).
  per_spec_results: results | preserve(original_spec_file ∧ diagnostics ∧ findings ∧ additional_fields)
  | diagnostics: flatten(∀result.diagnostics → original_entry ⊕ {spec_file: result.spec_file})
  | findings: flatten(∀result.findings → original_entry ⊕ {spec_file: result.spec_file})
  | entries: require(object) | invalid_entry → record(protocol_error) ∧ retain_in(per_spec_results)
  | aggregate_and_classify: valid_object_entries_only | ¬let(skipped_invalid_entry_hide_failure)
  | attribution: aggregate.spec_file ≔ envelope.spec_file | preserve(entry_local_locations)
  | original_entry.spec_file conflict → preserve_original_in(per_spec_results) | ¬lose(original_attribution)
  | compatibility: keep(entry_fields_at_top_level for existing_consumers)
  | total_specs: count(distinct_resolved_spec_file_paths(results)) | ¬count(expected_specs_as_processed)
  | coverage: compare(resolved_result_paths, expected_specs) using CLI_working_directory_for_relative_paths
  | missing ∨ unexpected ∨ duplicate_spec_results → record(protocol_error)
  | empty_stdout ∧ expected_specs ≠ ∅ → record(protocol_error)
  | empty_stdout ∧ expected_specs = ∅ → total_specs = 0 | ¬claim(any_spec_analyzed)

λ gybis-allium-analyse_finding_classification(entry).
  preserve(original_type ∧ original_severity ∧ message ∧ locations)
  | recognized_severity: exact_member(entry.severity, {critical, high, medium, low, informational})
  | absent ∨ unknown_severity: preserve(entry) | report(unclassified) | ¬infer(severity)
  | summary: count(aggregated_findings by recognized_severity) | initialize_all_counts(0) | ¬include(diagnostics)
  | unclassified: [{kind: diagnostic ∨ finding, spec_file, entry}] for absent_or_unknown_severity
  | remediation: collect(only_explicit_CLI_remediation with spec_file_attribution) | otherwise []

λ gybis-allium-analyse_status(cli_exit_code, parsing, coverage, diagnostics, findings).
  structured_issue_exit: cli_exit_code = 1 ∧ (diagnostics ≠ [] ∨ findings ≠ [])
  | execution_error: invocation_failure ∨ cli_exit_code ∉ {0, 1} ∨ (cli_exit_code = 1 ∧ ¬structured_issue_exit)
  | precedence: (parsing.failure ∨ coverage.protocol_error ∨ execution_error) → analysis_failed
  | otherwise: any(diagnostics ∪ findings has severity = critical) → critical_issues
  | otherwise: (diagnostics ≠ [] ∨ findings ≠ []) → findings_detected
  | otherwise: cli_exit_code = 0 → pass
  | stderr: return_verbatim | ¬treat(stderr_alone_as_findings ∨ proof_of_success)

λ gybis-allium-analyse_output_format(findings_data).
  structure: {directory: specs_path, status ∈ (pass ∨ findings_detected ∨ critical_issues ∨ analysis_failed), total_specs, per_spec_results: [...], diagnostics: [...], findings: [...], summary: {critical, high, medium, low, informational}, unclassified: [...], remediation: [...], consistency_checks, cli_exit_code, stderr, errors: [...]}
  | errors: parsing ∪ coverage ∪ execution_errors | analysis_failed → include(raw_stdout ∧ partial_results)
  | partial_results: parsing.failure ∨ coverage.protocol_error
  | consistency_checks: {coverage_complete, missing_specs: [...], unexpected_specs: [...], duplicate_specs: [...], inter_spec_analysis: not_established}
  | coverage_complete: ¬parsing.failure ∧ ¬coverage.protocol_error
  | inter_spec_analysis: ¬claim(cross_spec_consistency from empty_per_file_results)
  | json_serializable | human_readable

λ gybis-allium-analyse_execution(specs_path).
  invoke(gybis-allium-runtime-check_version()) → true ∨ halt("Allium runtime compatibility check failed")
  | enumerate: gybis-allium-analyse_scope(specs_path) → expected_specs | enumeration_failure → halt("Cannot enumerate specs")
  | invoke: gybis-allium-analyse_cli_invocation(specs_path)
  | capture: stdout ∧ stderr ∧ cli_exit_code ∧ CLI_working_directory
  | parse: gybis-allium-analyse_findings_parsing(stdout) → {results, failure, errors}
  | aggregate: gybis-allium-analyse_aggregation(results, expected_specs)
  | classify: gybis-allium-analyse_finding_classification per diagnostic ∧ finding
  | status: gybis-allium-analyse_status(cli_exit_code, parsing, coverage, diagnostics, findings)
  | format: gybis-allium-analyse_output_format(findings_data)
  | analysis_failed → halt(formatted_findings) | ¬return_as_successful_empty_analysis
  | otherwise → return(formatted_findings)

λ gybis-allium-analyse_regression_contract(x).
  invariant: multiple_pretty_printed_envelopes → all_decoded ∧ all_specs_counted
  | invariant: nested_JSON ∧ escaped_quotes ∧ braces_in_strings → no_false_record_boundaries
  | invariant: nonempty_entries_in_different_files → aggregate_all ∧ retain_each_spec_file
  | invariant: conflicting_entry_spec_file → original_preserved_in(per_spec_results)
  | invariant: malformed_second_object ∨ truncated_tail ∨ trailing_text → analysis_failed ∧ retain_valid_prefix
  | invariant: top_level_array ∨ missing_required_field ∨ invalid_entry → analysis_failed
  | invariant: missing_spec ∨ unexpected_spec ∨ duplicate_spec → analysis_failed
  | invariant: complete_clean_stream ∧ cli_exit_code = 0 → pass ∧ zero_summary_counts
  | invariant: complete_issue_stream ∧ cli_exit_code ∈ {0, 1} → findings_detected ∨ critical_issues
  | invariant: exit_1_without_issues ∨ exit_above_1 ∨ invocation_failure → analysis_failed
  | invariant: empty_stream ∧ expected_specs ≠ ∅ → analysis_failed
  | invariant: empty_stream ∧ expected_specs = ∅ ∧ cli_exit_code = 0 → total_specs = 0
  | invariant: analysis_failed → structured_halt ∧ ¬downstream_success
  | invariant: no_mutations
