Read and address the PR feedback.

## Workflow Summary:

1. Fetch PR details to get owner/repo info
2. Fetch ALL three types of feedback (issue comments, review comments, AND pull request reviews) using `--paginate`
3. **List out EVERY piece of feedback** in your response so the user can verify
4. **Create individual TodoWrite items** for each distinct piece of feedback (one todo per feedback item)
5. Address each todo systematically
6. Run tests and format code before committing

## Steps to properly fetch PR comments:

1. First get the PR details including owner:
   ```bash
   gh pr view --json number,headRepositoryOwner,headRepository
   ```

2. Then fetch **ALL** comments using the OWNER's repository (not nearform):

   **Issue comments** (general PR-level comments):
   ```bash
   gh api --paginate /repos/{owner}/studio/issues/{number}/comments --jq '.[] | {user: .user.login, body: .body, created: .created_at}'
   ```

   **Review comments** (line-specific code review comments):
   ```bash
   gh api --paginate /repos/{owner}/studio/pulls/{number}/comments --jq '.[] | {user: .user.login, body: .body, path: .path, line: .line, created: .created_at}'
   ```

   **Pull request reviews** (overall review feedback with state like "Changes requested"):
   ```bash
   gh api --paginate /repos/{owner}/studio/pulls/{number}/reviews --jq '.[] | {user: .user.login, state: .state, body: .body, created: .created_at}'
   ```

   **CRITICAL**: You MUST fetch ALL THREE types of feedback:
   - Issue comments (conversation on the PR)
   - Review comments (inline code comments)
   - Reviews (overall approval/changes requested with optional body)

   **CRITICAL**: Always use `--paginate` flag to ensure all pages of results are fetched, not just the first page.

3. For this repository specifically, use `Nullframe` as the owner, not `nearform`.

4. **List out EVERY piece of feedback** - read through all comments carefully and identify each distinct issue or suggestion.

## When addressing feedback:

1. **CRITICAL**: Use TodoWrite to create a separate todo item for EACH DISTINCT piece of feedback
   - Read through ALL comments (both issue comments and review comments)
   - Create one todo per feedback item - do NOT group multiple items together
   - Be specific in the todo description (e.g., "Fix type safety in validateConfig function" not just "Fix types")
   - Example: If there are 7 pieces of feedback, create 7 separate todos

2. Prioritize critical functionality issues first (e.g., incorrect behavior)
3. Then address code quality issues (type safety, duplication, etc.)
4. Run relevant tests after making changes
5. Commit with a clear message describing what feedback was addressed

## Common feedback patterns to watch for:

- **Type safety**: Remove unnecessary `any` casts, use proper types
- **Code duplication**: Extract shared utilities to dedicated files
- **Modern APIs**: Use `structuredClone()` instead of `JSON.parse(JSON.stringify())`
- **Reuse existing helpers**: Check if there are existing utility functions before writing new ones
- **Dynamic generation**: Generate options from arrays/constants rather than hardcoding

Respond with a summary of the changes you made to address the feedback.

Before committing, remember to run `make total-format` to ensure the code is properly formatted.
