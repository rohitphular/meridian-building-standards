# Instructions

Reusable prompts for common tasks. Fill in the `xxxxxxxx` placeholders before sending.

---

## New module setup

### Create CODE-REVIEW-INSTRUCTIONS document

```text
Create a CODE-REVIEW-INSTRUCTIONS.md document in _runbooks/ for the module by following the template. Read every source file in the module before writing — do not write a checklist item you cannot trace to a source file or a standards rule.
Module: xxxxxxxx
Template: building-standards/templates/TEMPLATE-CODE-REVIEW.md
```

### Create README document

```text
Create a README.md for the module by following the template. Read every source file, config file, and entry point in the module before writing — do not write a single line without first verifying it in the source.
Module: xxxxxxxx
Template: building-standards/templates/TEMPLATE-README.md
```

### Create USAGE-INSTRUCTIONS document

```text
Create a USAGE-INSTRUCTIONS.md in _runbooks/ for the module by following the template. Read every source file in the module before writing — every code snippet, parameter name, environment variable, method name, and configuration key must be traced to the actual source and confirmed correct. Write from the consumer's perspective only: how to declare the dependency, how to configure it, and how to use the public API. Do not document internal implementation details. Structure: one section per distinct use case, each with a minimal working example. Use the public API exports as the surface — do not show internal import paths.
Module: xxxxxxxx
Template: building-standards/templates/TEMPLATE-HOW-TO-USE.md
```

### Create HOW-TO-SETUP-KEY document

Create this document ONLY if the module requires an externally-provisioned secret to run — an API key, service-account key file, access token, or similar. If the module needs no such secret (plain configuration only, or configured entirely by values the caller passes in), skip it — do not create a not-applicable document.

```text
Create a HOW-TO-SETUP-KEY.md for the module by following the template. First read the source to confirm exactly which credential is required, how it is passed in (environment variable, config key, or argument), and the least privilege the module actually uses — trace every variable name and permission to source.
Module: xxxxxxxx
Template: building-standards/templates/TEMPLATE-HOW-TO-SETUP-KEY.md
```

---

## Code review

### Run the code review

```text
Run the code review now against _runbooks/CODE-REVIEW-INSTRUCTIONS.md
Module: xxxxxxxx
Code Review Document: xxxxxxxx
```

### Fix all code review findings

```text
Fix all FAIL and WARNING findings from the code review.
Module: xxxxxxxx
```

---

## README review

### Review the README

```text
Review the README for the module against the code implementation. It must follow the template structure and every statement must be accurate against the source files.
Module: xxxxxxxx
README: xxxxxxxx
Template: building-standards/templates/TEMPLATE-README.md
```

### Fix all README findings

```text
Fix all FAIL and WARNING findings from the README review.
Module: xxxxxxxx
```

---

## HOW-TO review

### Review USAGE-INSTRUCTIONS document

```text
Review _runbooks/USAGE-INSTRUCTIONS.md against the code implementation. It must follow the template structure and every statement must be accurate against the source files. For every instruction, command, code snippet, parameter name, environment variable, and configuration key: trace it to the actual source and confirm it is correct as written. Flag anything that is wrong, outdated, missing, or misleading. Report findings in PASS / FAIL / WARNINGS format.
Module: xxxxxxxx
USAGE-INSTRUCTIONS document: xxxxxxxx
Template: building-standards/templates/TEMPLATE-HOW-TO-USE.md
```

### Review HOW-TO-SETUP-KEY document

Run this only if the module has a HOW-TO-SETUP-KEY.md — it exists only when the module requires an externally-provisioned secret. Skip otherwise.

```text
Review HOW-TO-SETUP-KEY.md against the code implementation. It must follow the template structure and every step, variable name, config key, file path, and permission/scope must be accurate against the source files. Trace each to the actual source and confirm it is correct as written. Flag anything that is wrong, outdated, missing, or misleading. Report findings in PASS / FAIL / WARNINGS format.
Module: xxxxxxxx
HOW-TO document: xxxxxxxx
Template: building-standards/templates/TEMPLATE-HOW-TO-SETUP-KEY.md
```

### Fix all HOW-TO findings

```text
Fix all FAIL and WARNING findings from the HOW-TO review.
Module: xxxxxxxx
```

---

## Epic and story delivery

Use [the shared delivery process](documents/process/README.md) and [workflow prompts](documents/process/delivery-workflow-messages.md) for epic creation, review, story implementation and retrospectives. Set the project path in each prompt before use.
