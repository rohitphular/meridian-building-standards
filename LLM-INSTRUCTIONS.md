# Instructions

Reusable prompts for common tasks. Fill in the `xxxxxxxx` placeholders before sending.

---

## New module setup

### Create CODE-REVIEW document

```text
Create a CODE-REVIEW.md document for the module by following the template. Read every source file in the module before writing — do not write a checklist item you cannot trace to a source file or a standards rule.
Module: xxxxxxxx
Template: building-standards/TEMPLATE-CODE-REVIEW.md
```

### Create README document

```text
Create a README.md for the module by following the template. Read every source file, config file, and entry point in the module before writing — do not write a single line without first verifying it in the source.
Module: xxxxxxxx
Template: building-standards/TEMPLATE-README.md
```

### Create HOW-TO-USE-THIS-LIB document

```text
Create a HOW-TO-USE-THIS-LIB.md for the module. Read every source file in the module before writing — every code snippet, parameter name, environment variable, method name, and configuration key must be traced to the actual source and confirmed correct. Write from the consumer's perspective only: how to declare the dependency, how to configure it, and how to use the public API. Do not document internal implementation details. Structure: one section per distinct use case, each with a minimal working example. Use the public API exports from __init__.py as the surface — do not show internal import paths.
Module: xxxxxxxx
```

---

## Code review

### Run the code review

```text
Run the code review now against CODE-REVIEW.md
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
Template: building-standards/TEMPLATE-README.md
```

### Fix all README findings

```text
Fix all FAIL and WARNING findings from the README review.
Module: xxxxxxxx
```

---

## HOW-TO review

### Verify HOW-TO documents against implementation

```text
Read every HOW-TO-*.md document in the module and verify it is 100% accurate against the code. For every instruction, step, command, code snippet, parameter name, environment variable, and configuration example: trace it to the actual source and confirm it is correct as written. Flag anything that is wrong, outdated, missing, or misleading. Report findings in PASS / FAIL / WARNINGS format.
Module: xxxxxxxx
HOW-TO documents: xxxxxxxx
```

### Fix all HOW-TO findings

```text
Fix all FAIL and WARNING findings from the HOW-TO review.
Module: xxxxxxxx
```
