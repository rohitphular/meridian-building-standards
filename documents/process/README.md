# Development delivery process

Shared instructions, templates and prompts for specifying, reviewing and implementing work across Meridian projects.

## Workflow

1. The domain owner writes an epic: one shippable capability with shared conventions and a breakdown into stories.
2. The developer reviews the epic; the owner responds until outstanding findings are resolved.
3. The domain owner writes each story with acceptance criteria, examples and required inputs.
4. The developer reviews the story; the owner responds until the specification is approved.
5. The developer implements the approved story and produces evidence from the implementation.
6. The domain owner reviews that evidence; the developer resolves findings until the implementation is approved.
7. Both participants write an epic retrospective. A facilitator records the conclusion and any improvements to the shared process.

Use [delivery-workflow-messages.md](delivery-workflow-messages.md) for the complete sequence of prompts and links to each instruction and template.

## Applying this process

The documents were moved from Meridian Convex. They preserve its market-analytics conventions and worked examples: trading-desk and Python-developer roles, NSE terminology, numerical oracles, temporal traces and NDJSON samples. These are required for that workflow; they are not automatically requirements for unrelated products.

The workflow above is reusable by any project. When adopting the detailed templates for another domain, define its domain owner and developer roles, relevant evidence and output contracts explicitly. Preserve the approval stages and traceable acceptance criteria; document domain-specific changes rather than silently treating market rules as universal.

Process documents live in `building-standards/documents/process/`. Completed epics, stories, reviews, sample outputs and retrospectives live in the consuming project’s `delivery/workboard/`. Set `HOME` in the prompts to that project’s absolute path. There are no user-specific filesystem paths in the shared prompts.

## Sharing changes

This directory belongs to the `meridian-building-standards` Git repository. Commit and push process changes there first, then update and commit the submodule pointer in each consuming project. Do not commit a parent pointer to an unpublished shared commit.

From a consuming project:

```sh
git submodule update --init --recursive
git submodule update --remote building-standards
git add building-standards
git commit -m "Update shared building standards"
```

Use `--remote` deliberately to adopt the latest standards. A normal `--init --recursive` checkout restores the exact version recorded by the project.
