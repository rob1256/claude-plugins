---
name: bpp-jira-create-ticket
description: Write new Jira tickets
model: opus
---

# Skill: Write a Jira Ticket

## Description

A guided, step-by-step flow to write and create a Jira ticket directly in Jira. Supports four ticket
types, each with its own template: **User Story**, **Task**, **Spike**, and **Defect**.

## Trigger

When the user asks to "write a ticket", "create a Jira ticket", "new ticket", or similar.

## Tools Required

- `Atlassian:createJiraIssue`
- `Atlassian:getAccessibleAtlassianResources` (to resolve Cloud ID if not in CLAUDE.md)
- `Atlassian:getJiraProjectIssueTypesMetadata` (to fetch valid issue types for the project)
- `Atlassian:getJiraIssueTypeMetaWithFields` (to fetch required/available fields for the issue type)

## Pre-flight: Load Project Jira Config

**Before starting the guided flow**, read the project's `CLAUDE.md` file and look for a `## Jira`
section (or any Jira-related instructions anywhere in the file). Extract and remember:

- **Default project key** (e.g. `LEARN`)
- **Any other required fields** (e.g. labels, components, custom fields)
- **Any other instructions** (e.g. naming conventions, mandatory fields)

These values will be applied automatically in Step 7 without the user needing to provide them. If
the CLAUDE.md specifies required fields, they are non-negotiable — always include them.

## Step-by-step Flow

Walk through each step one at a time. Do NOT dump all questions at once. Wait for the user's
response before moving to the next step.

### Step 1 — Resolve Cloud ID & Project

1. Resolve the Jira Cloud ID via `getAccessibleAtlassianResources`.
2. If a default project was found in CLAUDE.md, suggest it. Let the user override.
3. Confirm the project key.

### Step 2 — Fetch Issue Types & Select Ticket Type

Once you have the Cloud ID and project key, fetch the actual issue types from Jira:

1. Call `getJiraProjectIssueTypesMetadata` with the `cloudId` and `projectIdOrKey`.
2. Present the available issue types to the user as a interactive numbered list, mapping them to
   friendly names:
   - **User Story** → look for a type named `Story` (or similar)
   - **Task** → look for `Task`
   - **Spike** → look for `Spike`
   - **Defect** → look for `Bug`
3. If any of the four expected types don't exist in the project, omit them from the list.
4. If the project has additional types not in the list above, include them too.
5. **Store the exact `id` and `name`** of the selected issue type — you'll need the `id` for field
   metadata lookup.

### Step 3 — Fetch Field Metadata

Call `getJiraIssueTypeMetaWithFields` with the `cloudId`, `projectIdOrKey`, and `issueTypeId` from
Step 2. This tells you which fields are required and which are available for this issue type.

1. Note any **required** fields that aren't covered by the template (summary, description, issue
   type, project).
2. Also identify any **pinned** fields — fields with `configuration.pinned: true` in the metadata
   response. These are fields the team considers important even if not strictly required.
3. For both required and pinned fields that have `allowedValues`, store those values — you'll
   present them to the user as selectable options in Step 5.
4. Ignore any "Template" type fields - you'll make the ticket contents from your own template.

### Step 4 — Title / Summary

Ask for a short, clear ticket title.

### Step 5 — Description & Considerations

Ask the user to describe what they need in their own words. This is a free-form prompt — encourage
them to include context, goals, constraints, edge cases, or anything else relevant. This will be
used to draft the template sections.

### Step 6 - Checking Description Against Type

If Step 2 is a Defect, Bug or similar issue type, and Step 5 did not cover the expected vs actual
result, prompt specifically for the "_expected result_", then prompt for the "_actual result_", to
ensure there is enough detail for the issue description.

If Step 2 is a Spike, Discovery or similar issue type, and Step 5 did not cover outcomes, have a
separate prompt specifically to cover.

If Step 3 revealed required or pinned fields not covered by the template (beyond summary,
description, issue type, and project), ask about those here too. For fields with `allowedValues`,
present the options as a numbered list so the user can pick easily rather than having to type exact
values.

Repeat Step 6 until confident to continue.

### Step 7 — Draft the Template

Using the user's description from Step 5 & Step 6, draft the full ticket description by filling in
the appropriate template below. Present the drafted description to the user for review.

### Step 8 — Review & Refine

Show the user the full drafted ticket (title + description). Ask if they want to change anything.
Iterate until they're happy.

### Step 9 — Create

Once approved, create the ticket in Jira using `Atlassian:createJiraIssue` with:

- `cloudId`: resolved in Step 1
- `projectKey`: from Step 1
- `issueTypeName`: the exact `name` from the Jira API (Step 2) — NOT the friendly label
- `summary`: from Step 4
- `description`: the final markdown description
- `additional_fields`: include any fields required by CLAUDE.md (e.g. team), any required fields,
  and any pinned fields the user provided values for in Step 5 & Step 6. Use the field metadata from
  Step 3 to determine the correct field key and value format. Common examples:
  - **Team:** `{ "customfield_XXXXX": "<< TEAM ID/UUID >>" }` — look up the correct custom field key
    for "Team" from the field metadata returned in Step 3
  - **Labels:** `{ "labels": ["label1", "label2"] }`
  - **Components:** `{ "components": [{ "name": "component-name" }] }`
- `parent`: if provided by the user

Confirm creation and share the ticket key/link with the user.

---

## Templates

Use the template that matches the ticket type selected in Step 2. Fill in each section using the
user's input from Step 5 & Step 6. If the user hasn't provided enough info for a section, either ask
them or skip the section (including header) entirely.

---

### Story Template

```
> As a [persona]
>
> I would like to [complete an action]
>
> So that [consequences of completing that action]

## Context

Provide a clear and detailed description of the task or story. Include any relevant background information,
objectives, or context necessary to understand the work that needs to be done.

## Designs

Link or attach relevant design files or mockups.

- Which components will we be using?
- Do we need to build any new component?

## Acceptance Criteria

- Criterion 1
- Criterion 2
- Criterion 3

## Technical Notes

- Permissions - any specific requirements?
- Technical note 1
- Technical note 2

## Accessibility

Specify any accessibility requirements or considerations.

## Testing

Specify if any of the following tests is required and outline any specific considerations.

- Unit testing:
- Playwright testing:
- Integration testing:

## Extra Material

Provide links or attach any additional materials, documents, or resources that might be needed.
```

- **Acceptance Criteria** for this issue type should be written in a GIVEN.. WHEN.. THEN.. format.

### Task Template

```
## Context

Provide a clear and detailed description of the task or story. Include any relevant background information,
objectives, or context necessary to understand the work that needs to be done.

## Designs

Link or attach relevant design files or mockups.

- Which components will we be using?
- Do we need to build any new component?

## Acceptance Criteria

- Criterion 1
- Criterion 2
- Criterion 3

## Technical Notes

- Permissions - any specific requirements?
- Technical note 1
- Technical note 2

## Accessibility

Specify any accessibility requirements or considerations.

## Testing

Specify if any of the following tests is required and outline any specific considerations.

- Unit testing:
- Playwright testing:
- Integration testing:

## Extra Material

Provide links or attach any additional materials, documents, or resources that might be needed.
```

---

### Defect Template

```
## Context

Provide a clear description of the bug and any relevant context or background information.

## Environment

- Browser: [Name and version]
- Device (e.g. OS): [Make, model, version]
- User or Role: [If applicable, no passwords]
- Environment: [Dev, Test, Demo, Prod]

## Steps to Reproduce

1. First
2. Second
3. Third

## Actual Result

Describe what actually happens when the steps above are followed.

## Expected Result

Describe what should happen when the steps above are followed.

## Technical Notes

Any technical insights or initial findings about the cause of the bug.

## Testing

Specify if any of the following is required to confirm the bug fix and any specific considerations.

- Unit testing:
- Playwright testing:
- Integration testing:

## Attachments / Screenshots

Attach any relevant screenshots, logs, or other files that help illustrate the bug.

Link to a PI ticket (if applicable).
```

---

### Spike Template

```
## Hypothesis

What is the context that we are investigating?
Why are we looking into this?

## Steps to Investigate

Outline the steps of the spike, including other relevant ideas & references.

## Technical Notes

- Note 1
- Note 2
- Note 3

## Expected Outcome

- Confluence page with the investigation outcome?
- A follow up ticket to complete the work?
- A showcase PR with the work completed?

## Duration

How long we want to spend on the investigation before providing feedback.

## Open Questions

Any open questions to consider during the spike.

## Extra Material

Provide links or attach any additional materials, documents, or resources that might be needed.
```

### Other Templates

If using a Jira issue type that does not fit - consider the templates above & create something
appropriate.

---

## Important Notes

- **Stick to the templates exactly** — do not add or remove sections.
- Story and Task use the **same template**.
- Always present the full draft for review before creating.
- Use markdown formatting in the description field.
- Be concise but thorough when drafting — use the user's own language where possible.
- **Always use the issue type `name` from the Jira API** — never hardcode or guess type names.
- **Always apply CLAUDE.md Jira instructions** — if it says to assign a team, add labels, etc.,
  include them in `additional_fields`. These are non-negotiable.
- **Always consider pinned fields** — pinned fields (`configuration.pinned: true`) are fields the
  team has marked as important. Treat them like soft-required: present them to the user with their
  allowed values and include any values the user provides.
- **Use field metadata to get correct field keys** — don't guess custom field IDs. The metadata from
  `getJiraIssueTypeMetaWithFields` tells you the exact key (e.g. `customfield_10001`) and expected
  value format.
