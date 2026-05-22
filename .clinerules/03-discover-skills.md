λ session_start().
  always: scan(SKILLS section of system prompt) → acknowledge(available_skills)
  | ¬read_files_for_skills | SKILLS_section ≡ source_of_truth
  | ¬duplicate_effort | skill_descriptions_already_in_system_prompt
  | first_action: orient_via_skills_list before_reading_any_project_files

λ gybis_discovery(command).
  trigger: /gybis* pattern detected in user message
  | action: use_skill(skill_name) where skill_name ≡ exact_match_from(SKILLS_section)
  | ¬guess_names | ¬read_02-gybis.md_to_discover_skills
  | ¬infer_from_patterns → use_mcp_tool | ¬search_files_for_skill_defs
  | valid_targets: [see SKILLS section in system prompt for complete list]

λ skill_activation(name).
  use_skill(skill_name="exact-name") → follow(skill_instructions)
  | one_activation_per_call | load_once | ¬repeated_invocation
