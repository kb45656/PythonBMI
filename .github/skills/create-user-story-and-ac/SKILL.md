---
name: create-user-story-and-ac
description: 'Create or complete User Story and Acceptance Criteria for GitHub Project work items in the Ready column. Use when the user asks to review Ready items and fill missing requirements from each item title.'
argument-hint: Review Ready items in GitHub Project [project URL or owner/project number]
---

# Create User Story and Acceptance Criteria

Review the GitHub Project specified by the user and complete missing requirements for every work item in its `Ready` status.

## Workflow

1. Identify the project owner and project number from the supplied URL.
2. Read the project with GitHub GraphQL or the GitHub CLI. Retrieve:
   - Project item IDs
   - The `Status` field and its `Ready` option
   - Each Ready item's content title, body, URL, and repository
3. Inspect each Ready item's existing body:
   - Treat a User Story as present when a non-empty `## User Story` section exists.
   - Treat Acceptance Criteria as present when a non-empty `## Acceptance Criteria` section exists.
   - Do not overwrite an existing non-empty section.
4. For each missing section, draft content from the item's title and the item's existing context.
5. Update the underlying GitHub issue, draft issue, or pull request body. Preserve all existing body content and append or insert only the missing sections.
6. Use this exact structure:

```markdown
## User Story
身為<角色>，我希望<需求>，以便<使用者價值>。

## Acceptance Criteria
- [ ] <可驗證的條件一>
- [ ] <可驗證的條件二>
- [ ] <可驗證的條件三>
```

## Writing rules

- Write in Traditional Chinese unless the item's title or user request clearly requires another language.
- Derive the role, need, value, and criteria from the title; do not invent unrelated scope.
- Make every Acceptance Criteria item independently verifiable.
- Include normal flow, validation/error handling, and result/display behavior when applicable.
- Acceptance Criteria must always use unchecked Markdown checklist syntax: `- [ ]`.
- Keep criteria concise, concrete, and testable.
- If the title is too vague to produce safe, specific requirements, use neutral wording tied to the title rather than guessing implementation details.
- If the body already contains one section, add only the other missing section.
- If the body already contains both sections, make no update.

## GitHub CLI guidance

For a user-owned project, query project items with GraphQL:

```powershell
$query = @'
query($login:String!, $number:Int!) {
  user(login:$login) {
    projectV2(number:$number) {
      id
      fields(first:100) {
        nodes {
          __typename
          ... on ProjectV2SingleSelectField { id name options { id name } }
        }
      }
      items(first:100) {
        nodes {
          id
          fieldValues(first:100) {
            nodes {
              __typename
              ... on ProjectV2ItemFieldSingleSelectValue {
                name optionId
                field { ... on ProjectV2SingleSelectField { id name } }
              }
            }
          }
          content {
            __typename
            ... on Issue { id number title body url repository { nameWithOwner } }
            ... on DraftIssue { id title body }
            ... on PullRequest { id number title body url repository { nameWithOwner } }
          }
        }
      }
    }
  }
}
'@
$variables = '{"login":"OWNER","number":PROJECT_NUMBER}'
$payload = @{ query = $query; variables = $variables | ConvertFrom-Json } |
  ConvertTo-Json -Depth 20
$payload | gh api graphql --input -
```

Update an issue body with:

```powershell
gh api repos/OWNER/REPOSITORY/issues/NUMBER -X PATCH -f body="$body"
```

For pull requests, use the pull request endpoint. Draft issues require the project/API mutation supported by the current GitHub CLI/API version; if it is unavailable, report the item and do not silently mark it complete.

## Completion report

Report:

- Which Ready items were inspected
- Which items were updated
- Which sections were added to each item
- Which items were skipped because both sections already existed
- Any item that could not be updated, with the reason

Verify every updated item by reading it back from GitHub and checking that the required headings and `- [ ]` checklist lines are present.
