# HOW-TO-USE-THIS-LIB Template

<!--
INSTRUCTIONS FOR THE LLM WRITING THE HOW-TO-USE-THIS-LIB DOCUMENT
================================================================

WHAT YOU ARE CREATING:
  A consumer-facing guide for a reusable library/package: how to depend on it, configure it,
  and call its public API. Write for someone integrating the library into their own module —
  never document internal implementation details.

BEFORE WRITING ANYTHING:
  Read every source file in the module. Every command, dependency name, parameter, environment
  variable, configuration key, public symbol, and return type must be traced to the source and
  confirmed correct as written. Do not write a line you cannot verify.

RULES:
  1. Replace every {{PLACEHOLDER}} with the real value. Never leave a placeholder unfilled.
  2. Every [OPTIONAL] section: include it if it applies, delete it entirely if it does not.
  3. Document only the PUBLIC API — the symbols the package exports for consumers. Never show
     internal or private import paths, and never document internal helpers.
  4. One section per distinct use case. Each use case gets a minimal, runnable example.
  5. Every example must be self-contained and copy-pasteable, and every symbol in it traceable
     to source.
  6. Remove ALL comment blocks (this one included) from the final document.
  7. Run the Self-Check at the bottom before delivering. Then delete the Self-Check section.

STYLE:
  - Plain declarative sentences. Active voice. Present tense.
  - No marketing language. No motivation, history, or future plans.
  - Shorter is always better.
-->

---

# HOW-TO-USE-THIS-LIB — {{LIBRARY_NAME}}

<!-- One sentence: what the library is and the way(s) a consumer uses it (e.g. a CLI, an imported API, or both). -->

{{ONE_LINE_DESCRIPTION}}

---

## Add as a dependency

<!--
  Show how a consuming module declares this library with its dependency manager, using the
  project's standard mechanism (package manifest + source location). State whether it is a
  runtime dependency or a development/build-only dependency, and where it is declared.
  Read: the package manifest and the dependency-source conventions in the standards docs.
-->

{{DEPENDENCY_DECLARATION}}

<!-- State the minimum language/runtime version the library requires, traced to the manifest. -->

{{RUNTIME_REQUIREMENT}}

---

## Configure

<!--
  List every input the consumer must provide before the library works: environment variables,
  credential/key files, config-file keys, and/or constructor/initialisation arguments.
  For each: name it exactly as the source reads it, state whether it is required, and what
  happens if it is missing (traced to source). If the library reads NOTHING from the environment
  and is configured only by values the caller passes in, say so explicitly.
  If the library requires a credential or external key, link to HOW-TO-SETUP-KEY.md here.
-->

{{CONFIGURATION}}

---

## {{USE_CASE_1}}

<!--
  One section per distinct use case (one public operation, or one common task). Repeat for each.
  Each: one or two sentences of what it does, then a minimal working example using only the
  public API. State the return type/shape and the failure behaviour (what it raises or returns
  on error) — traced to source.
-->

{{USE_CASE_EXAMPLE}}

---

## Public API reference [OPTIONAL]

<!--
  A compact table of every public symbol the package exports for consumers. Include only if the
  use-case sections do not already cover the full public surface. Omit otherwise.
-->

| Symbol | Signature | Returns |
|--------|-----------|---------|
| `{{SYMBOL}}` | `{{SIGNATURE}}` | {{RETURNS}} |

---

## Common pitfalls [OPTIONAL]

<!--
  A table of real, traceable mistakes: what the consumer does wrong, what actually happens (the
  real error message or behaviour from the source), and the fix. Omit if none apply.
-->

| Mistake | What happens | Fix |
|---|---|---|
| {{MISTAKE}} | {{OUTCOME}} | {{FIX}} |

---

<!--
SELF-CHECK — complete before delivering. Then delete this entire block.
================================================================
[ ] Every {{PLACEHOLDER}} replaced with a real value.
[ ] Every command, dependency name, parameter, env var, config key, and public symbol traced to source.
[ ] Only the public API is shown — no internal import paths or private helpers.
[ ] One section per distinct use case, each with a minimal working example.
[ ] Every example is self-contained and copy-pasteable.
[ ] Runtime/version requirement stated and traced to the manifest.
[ ] HOW-TO-SETUP-KEY.md linked if the library requires a credential or external key.
[ ] No marketing language, motivation, history, or future plans.
[ ] All comment blocks removed from the final document.
-->
