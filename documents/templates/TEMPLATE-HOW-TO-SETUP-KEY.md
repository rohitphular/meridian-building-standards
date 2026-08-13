# HOW-TO-SETUP-KEY Template

<!--
INSTRUCTIONS FOR THE LLM WRITING THE HOW-TO-SETUP-KEY DOCUMENT
=============================================================

WHEN TO CREATE THIS DOCUMENT:
  Create it ONLY if the module requires an externally-provisioned secret to run — an API key,
  service-account key file, OAuth client, access token, TLS certificate, or similar access grant
  a person must obtain from a provider or an administrator. If the module needs no such secret
  (it takes only plain configuration, or is configured entirely by values the caller passes in),
  DO NOT create this document.

WHAT YOU ARE CREATING:
  A step-by-step guide for obtaining the credential, storing it safely, and wiring it into the
  module. Write for an operator setting the module up for the first time.

BEFORE WRITING ANYTHING:
  Read the source to confirm exactly what the module needs: which credential, in what form (file,
  string, token), how it is passed in (environment variable, config key, or argument), and which
  scopes/permissions are actually used. Every variable name and permission level must be traced
  to source.

RULES:
  1. Replace every {{PLACEHOLDER}} with the real value. Never leave a placeholder unfilled.
  2. Provider UIs change — describe steps by their stable navigation and outcome, not brittle
     exact labels. Keep them accurate to the current provider.
  3. Instruct the least privilege the module actually uses — never a broader grant.
  4. Never instruct storing the secret in the repository or in source control.
  5. Remove ALL comment blocks (this one included) from the final document.
  6. Run the Self-Check at the bottom before delivering. Then delete the Self-Check section.

STYLE:
  - Numbered steps in the order the operator performs them.
  - Plain declarative sentences. Active voice.
-->

---

# HOW-TO-SETUP-KEY — {{MODULE_NAME}}

<!-- One sentence: what credential this sets up and what it grants access to. -->

{{ONE_LINE_DESCRIPTION}}

---

## What you need

<!-- Prerequisites the operator must already have (an account, admin access, the target resource). -->

- {{PREREQUISITE}}

---

## Step 1 — {{OBTAIN_STEP_TITLE}}

<!--
  Numbered steps to provision the credential at the provider. Repeat as "## Step 2 — ..." etc.
  State the exact permission/scope to grant — the least privilege the module uses, traced to source.
-->

{{OBTAIN_STEPS}}

---

## Store and wire in the credential

<!--
  State a safe storage location OUTSIDE any source-controlled directory. Name the exact
  environment variable / config key / file path the module reads, traced to source. If the module
  reads the secret only from a value the caller passes in, say so and show the recommended pattern:
  read it from the environment in the caller's configuration and fail loudly (with the variable
  name) if it is absent.
-->

{{STORE_AND_WIRE}}

---

## Security rules

- Never commit the credential or its file path to source control.
- Never log the credential, its path, or any field derived from it.
- Store it in an environment variable or the runtime's secrets manager — never hardcoded.
- Grant the least privilege the module actually uses.
- If the credential is exposed, revoke or rotate it at the provider immediately, then issue a new one.
- {{ADDITIONAL_RULE}} [OPTIONAL]

---

<!--
SELF-CHECK — complete before delivering. Then delete this entire block.
================================================================
[ ] This document is warranted — the module genuinely requires an externally-provisioned secret.
[ ] Every {{PLACEHOLDER}} replaced with a real value.
[ ] Every variable name / config key / file path the module reads is traced to source.
[ ] The permission/scope instructed is the least privilege the module actually uses.
[ ] No step instructs storing the secret in source control.
[ ] No step instructs logging the secret or its path.
[ ] All comment blocks removed from the final document.
-->
