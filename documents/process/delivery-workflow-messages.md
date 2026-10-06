# Delivery Workflow Messages

Set `HOME` to the consuming project’s absolute path in each prompt. Process instructions come from its `building-standards` submodule; completed specifications, reviews and samples stay in that project’s `delivery/workboard/`. These prompts use the market-analytics roles described in [README.md](README.md).

1. Writing Epic Details

```
@rp-trading-desk-lead

HOME = <absolute-path-to-project>
EPIC_NUMBER = 1

Write the epic requirement for T{EPIC_NUMBER} following the instructions and using the template as the exact structure — do not deviate from it:
- Instructions: {HOME}/building-standards/documents/process/epic-creation-instructions.md
- Template:     {HOME}/building-standards/documents/process/epic-creation-template.md

Save the completed epic to:
{HOME}/delivery/workboard/epic={EPIC_NUMBER}/epic-details.md

Complete the pre-submission checklist in the instructions before saving.
```

2. Reviewing Epic Details
```
@rp-principal-python-developer

HOME = <absolute-path-to-project>
EPIC_NUMBER = 1
OPERATION_ROUND = 1

Review the epic details available here:
{HOME}/delivery/workboard/epic={EPIC_NUMBER}/epic-details.md

Follow the review instructions and write your feedback using the template as the exact structure — do not deviate from it:
- Instructions: {HOME}/building-standards/documents/process/epic-review-instructions.md
- Template:     {HOME}/building-standards/documents/process/epic-review-template.md

Save your review to:
{HOME}/delivery/workboard/epic={EPIC_NUMBER}/epic-review/round={OPERATION_ROUND}/review-feedback.md
```

3. Responding to Epic Details Review
```
@rp-trading-desk-lead

HOME = <absolute-path-to-project>
EPIC_NUMBER = 1
OPERATION_ROUND = 1

Your epic details have been reviewed. Respond to the review feedback following the instructions and using the template as the exact structure — do not deviate from it:
- Review feedback:  {HOME}/delivery/workboard/epic={EPIC_NUMBER}/epic-review/round={OPERATION_ROUND}/review-feedback.md
- Instructions:     {HOME}/building-standards/documents/process/epic-review-response-instructions.md
- Template:         {HOME}/building-standards/documents/process/epic-review-response-template.md

Save your response to:
{HOME}/delivery/workboard/epic={EPIC_NUMBER}/epic-review/round={OPERATION_ROUND}/review-feedback-response.md
```

4. Loop — Epic Review

Repeat messages 2 and 3 incrementing OPERATION_ROUND by 1 each time until the
reviewer marks the epic as approved with no outstanding findings.

5. Writing Story Details
```
@rp-trading-desk-lead

HOME = <absolute-path-to-project>
EPIC_NUMBER = 1
STORY_NUMBER = 1

Write the story requirement for story {STORY_NUMBER} under epic {EPIC_NUMBER} following the instructions and using the template as the exact structure — do not deviate from it:
- Instructions: {HOME}/building-standards/documents/process/story-creation-instructions.md
- Template:     {HOME}/building-standards/documents/process/story-creation-template.md

Save the completed story to:
{HOME}/delivery/workboard/epic={EPIC_NUMBER}/story={STORY_NUMBER}/story-details.md

Complete the pre-submission checklist in the instructions before saving.
```

6. Reviewing Story Details
```
@rp-principal-python-developer

HOME = <absolute-path-to-project>
EPIC_NUMBER = 1
STORY_NUMBER = 1
OPERATION_ROUND = 1

Review the story details available here:
{HOME}/delivery/workboard/epic={EPIC_NUMBER}/story={STORY_NUMBER}/story-details.md

Follow the review instructions and write your feedback using the template as the exact structure — do not deviate from it:
- Instructions: {HOME}/building-standards/documents/process/story-review-instructions.md
- Template:     {HOME}/building-standards/documents/process/story-review-template.md

Save your review to:
{HOME}/delivery/workboard/epic={EPIC_NUMBER}/story={STORY_NUMBER}/review-requirement/round={OPERATION_ROUND}/review-feedback.md
```

7. Responding to Story Details Review
```
@rp-trading-desk-lead

HOME = <absolute-path-to-project>
EPIC_NUMBER = 1
STORY_NUMBER = 1
OPERATION_ROUND = 1

Your story details have been reviewed. Respond to the review feedback following the instructions and using the template as the exact structure — do not deviate from it:
- Review feedback:  {HOME}/delivery/workboard/epic={EPIC_NUMBER}/story={STORY_NUMBER}/review-requirement/round={OPERATION_ROUND}/review-feedback.md
- Instructions:     {HOME}/building-standards/documents/process/story-review-response-instructions.md
- Template:         {HOME}/building-standards/documents/process/story-review-response-template.md

Save your response to:
{HOME}/delivery/workboard/epic={EPIC_NUMBER}/story={STORY_NUMBER}/review-requirement/round={OPERATION_ROUND}/review-feedback-response.md
```

8. Loop — Story Requirement Review

Repeat messages 6 and 7 incrementing OPERATION_ROUND by 1 each time until the
reviewer marks the story as approved with no outstanding findings.

9. Implementation
```
@rp-principal-python-developer

HOME = <absolute-path-to-project>
EPIC_NUMBER = 1
STORY_NUMBER = 1
OPERATION_ROUND = 1

Implement story {STORY_NUMBER} under epic {EPIC_NUMBER} following the instructions below:
- Epic details:   {HOME}/delivery/workboard/epic={EPIC_NUMBER}/epic-details.md
- Story details:  {HOME}/delivery/workboard/epic={EPIC_NUMBER}/story={STORY_NUMBER}/story-details.md
- Instructions:   {HOME}/building-standards/documents/process/story-implementation-instructions.md

Save your sample output to:
{HOME}/delivery/workboard/epic={EPIC_NUMBER}/story={STORY_NUMBER}/review-implementation/round={OPERATION_ROUND}/sample/
```

10. Review Implementation
```
@rp-trading-desk-lead

HOME = <absolute-path-to-project>
EPIC_NUMBER = 1
STORY_NUMBER = 1
OPERATION_ROUND = 1

Review the implementation sample for story {STORY_NUMBER} under epic {EPIC_NUMBER} following the instructions and using the template as the exact structure — do not deviate from it:
- Story details:  {HOME}/delivery/workboard/epic={EPIC_NUMBER}/story={STORY_NUMBER}/story-details.md
- Sample:         {HOME}/delivery/workboard/epic={EPIC_NUMBER}/story={STORY_NUMBER}/review-implementation/round={OPERATION_ROUND}/sample/
- Instructions:   {HOME}/building-standards/documents/process/story-implementation-review-instructions.md
- Template:       {HOME}/building-standards/documents/process/story-implementation-review-template.md

Save your review to:
{HOME}/delivery/workboard/epic={EPIC_NUMBER}/story={STORY_NUMBER}/review-implementation/round={OPERATION_ROUND}/review-feedback.md
```

11. Responding to Implementation Review
```
@rp-principal-python-developer

HOME = <absolute-path-to-project>
EPIC_NUMBER = 1
STORY_NUMBER = 1
OPERATION_ROUND = 1
NEXT_ROUND = 2   (= OPERATION_ROUND + 1; increment both when OPERATION_ROUND increments)

Your implementation has been reviewed. Respond to the review feedback following the instructions and using the template as the exact structure — do not deviate from it:
- Story details:    {HOME}/delivery/workboard/epic={EPIC_NUMBER}/story={STORY_NUMBER}/story-details.md
- Review feedback:  {HOME}/delivery/workboard/epic={EPIC_NUMBER}/story={STORY_NUMBER}/review-implementation/round={OPERATION_ROUND}/review-feedback.md
- Instructions:     {HOME}/building-standards/documents/process/story-implementation-review-response-instructions.md
- Template:         {HOME}/building-standards/documents/process/story-implementation-review-response-template.md

Save your response to:
{HOME}/delivery/workboard/epic={EPIC_NUMBER}/story={STORY_NUMBER}/review-implementation/round={OPERATION_ROUND}/review-feedback-response.md

Save your updated sample to:
{HOME}/delivery/workboard/epic={EPIC_NUMBER}/story={STORY_NUMBER}/review-implementation/round={NEXT_ROUND}/sample/
```

12. Loop — Implementation Review and Stories

Inner loop: Repeat messages 10 and 11 incrementing OPERATION_ROUND by 1 each time
until the reviewer marks the story implementation as approved with no outstanding findings.

Outer loop: Repeat messages 5 through 12 incrementing STORY_NUMBER by 1 each time
until all stories in the epic are complete. Reset OPERATION_ROUND to 1 for each new story.

13.1 Retrospective — Trading Desk Lead
```
@rp-trading-desk-lead

HOME = <absolute-path-to-project>
EPIC_NUMBER = 1

Epic {EPIC_NUMBER} is complete. Write your retrospective following the instructions and using the template as the exact structure — do not deviate from it:
- Epic details:  {HOME}/delivery/workboard/epic={EPIC_NUMBER}/epic-details.md
- Instructions:  {HOME}/building-standards/documents/process/retrospective-creation-instructions.md
- Template:      {HOME}/building-standards/documents/process/retrospective-creation-template.md

Save your retrospective to:
{HOME}/delivery/workboard/epic={EPIC_NUMBER}/retrospective/trader-comments.md
```

13.2 Retrospective — Principal Python Developer
```
@rp-principal-python-developer

HOME = <absolute-path-to-project>
EPIC_NUMBER = 1

Epic {EPIC_NUMBER} is complete. Write your retrospective following the instructions and using the template as the exact structure — do not deviate from it:
- Epic details:  {HOME}/delivery/workboard/epic={EPIC_NUMBER}/epic-details.md
- Instructions:  {HOME}/building-standards/documents/process/retrospective-creation-instructions.md
- Template:      {HOME}/building-standards/documents/process/retrospective-creation-template.md

Save your retrospective to:
{HOME}/delivery/workboard/epic={EPIC_NUMBER}/retrospective/developer-comments.md
```

13.3 Retrospective — Conclusion
```
Act as a senior scrum master.

HOME = <absolute-path-to-project>
EPIC_NUMBER = 1

Both parties have completed their retrospectives for epic {EPIC_NUMBER}. Write the retrospective conclusion following the instructions and using the template as the exact structure — do not deviate from it:
- Epic details:            {HOME}/delivery/workboard/epic={EPIC_NUMBER}/epic-details.md
- Trader retrospective:    {HOME}/delivery/workboard/epic={EPIC_NUMBER}/retrospective/trader-comments.md
- Developer retrospective: {HOME}/delivery/workboard/epic={EPIC_NUMBER}/retrospective/developer-comments.md
- Instructions:            {HOME}/building-standards/documents/process/retrospective-conclusion-instructions.md
- Template:                {HOME}/building-standards/documents/process/retrospective-conclusion-template.md

Save the conclusion to:
{HOME}/delivery/workboard/epic={EPIC_NUMBER}/retrospective/retrospective-conclusion.md

After writing the conclusion, update any instruction or template files in
{HOME}/building-standards/documents/process/ that need to change based on the gaps identified.
Record every file changed in the conclusion under "Process Files Updated."
```

14.1 Learning — Trading Desk Lead
```
@rp-trading-desk-lead

HOME = <absolute-path-to-project>
EPIC_NUMBER = 1

Read the retrospective conclusion for epic {EPIC_NUMBER} and save the important learnings to your memory following the instructions below:
- Retrospective conclusion: {HOME}/delivery/workboard/epic={EPIC_NUMBER}/retrospective/retrospective-conclusion.md
- Instructions:             {HOME}/building-standards/documents/process/retrospective-learning-instruction.md
```

14.2 Learning — Principal Python Developer
```
@rp-principal-python-developer

HOME = <absolute-path-to-project>
EPIC_NUMBER = 1

Read the retrospective conclusion for epic {EPIC_NUMBER} and save the important learnings to your memory following the instructions below:
- Retrospective conclusion: {HOME}/delivery/workboard/epic={EPIC_NUMBER}/retrospective/retrospective-conclusion.md
- Instructions:             {HOME}/building-standards/documents/process/retrospective-learning-instruction.md
```