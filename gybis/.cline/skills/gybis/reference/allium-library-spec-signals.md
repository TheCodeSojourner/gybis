# Recognising library spec opportunities

During elicitation, stay alert for descriptions that suggest a library spec rather than application-specific logic. Library specs are standalone specifications for generic integrations that could be reused across projects.

This applies equally to distillation. When examining existing code and finding OAuth flows or payment processing, the same questions apply.

## Signals that something might be a library spec

λ(signal, external_integration, "Google/Microsoft/GitHub login" ∨ "Stripe/PayPal payments" ∨ "SendGrid/Postmark email" ∨ "Google Calendar sync" ∨ "S3/GCS storage")
λ(signal, generic_patterns, OAuth_flows ∨ session_management ∨ token_refresh ∨ payment_processing ∨ subscriptions ∨ invoicing ∨ email_delivery ∨ bounce_handling ∨ unsubscribes ∨ file_upload ∨ virus_scanning ∨ thumbnail_generation ∨ webhook_receipt ∨ retry_logic ∨ signature_verification)
λ(signal, impl_agnostic, "work account SSO" ∨ "charge monthly" ∨ "get notified")

## Questions to ask

λ(question, standard_or_specific, "Is this specific to your system, or a standard integration?" → standard = library_spec_candidate)
λ(question, reusable, "Would another system integrating with [X] work the same way?" → yes = library_spec_candidate)
λ(question, customisation, "Specific customisations to [X], or standard?" → standard = library_spec ∨ heavy_customisation = library_spec_with_config)
λ(question, existing_spec, "Look for existing library spec for [X], or need custom?" → encourages_reuse)

## How to handle the decision

λ(option, existing_library_spec, "Standard OAuth flow — likely existing library spec. Reference that rather than specifying OAuth details here. Application spec responds to authentication events.")
λ(option, create_new_library_spec, "Greenhouse ATS integration sounds generic enough for its own library spec. Other hiring apps might integrate same way. Create separate greenhouse-ats.allium that this application references.")
λ(option, keep_inline, "Integration so specific to your system — doesn't make sense as standalone spec. Include directly.")

## Common library spec candidates

λ(candidate, Authentication, OAuth_providers(Google ∨ Microsoft ∨ GitHub) ∨ SAML ∨ magic_links)
λ(candidate, Payments, Stripe ∨ PayPal ∨ subscription_billing ∨ usage_based_billing)
λ(candidate, Communications, Email_delivery ∨ SMS ∨ push_notifications ∨ Slack ∨ Teams)
λ(candidate, Storage, S3_compatible ∨ file_scanning ∨ image_processing)
λ(candidate, Calendar, Google_Calendar ∨ Outlook ∨ iCal_feeds)
λ(candidate, CRM_ATS, Salesforce ∨ HubSpot ∨ Greenhouse ∨ Lever)
λ(candidate, Analytics, Segment ∨ Mixpanel ∨ event_tracking)
λ(candidate, Infrastructure, Webhook_handling ∨ rate_limiting ∨ audit_logging)

## The boundary question

When you identify a library spec candidate, the key question is: "Where does the library spec end and the application spec begin?"

λ(boundary, library_handles, integration_mechanics(OAuth_flow ∨ payment_processing) ∨ events_any_consumer_cares(login_succeeded ∨ payment_failed) ∨ config_varies_between_deployments)
λ(boundary, application_handles, events_response_in_your_system ∨ application_specific_entities(your_User ∨ your_Subscription) ∨ business_rules_unique_to_domain)

```
-- Library spec (oauth.allium) handles:
--   - Provider configuration
--   - Token exchange
--   - Session lifecycle
--   - Emits: AuthenticationSucceeded, SessionExpired, etc.

-- Application spec handles:
--   - Creating your User entity on first login
--   - What roles/permissions new users get
--   - Blocking suspended users from logging in
--   - Audit logging specific to your compliance needs
```

## Red flags you missed a library spec

During review, watch for:

λ(red_flag, protocol_descriptions, "First redirect to Google, then redirect back with code, then exchange for token..." → OAuth → use_library_spec)
λ(red_flag, vendor_details, "Stripe sends webhook with event type invoice.paid..." → Stripe_integration → use_library_spec)
λ(red_flag, repeated_patterns, similar_retry/timeout/error_handling_for_multiple_integrations → extract_common_pattern)