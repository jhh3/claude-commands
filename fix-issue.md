Fix a sentry issue

$ARGUMENTS

1. If we're not already on a branch jhh3/<issue-id>, create one off of main
   issue id is not a number it's the dash string you'll get after looking it up with sentry mcp
2. Use sentry mcp to gather details about the issue
3. Diagnose the issue (or respect any guidance provided by the user) and fix it
   IMPORTANT: NEVER use sentry seer, diagnose the issue yourself
4. If it makes sense, create a regression test for the issue (do this first when test driven development is possible)
5. Clean up code.

```
make total-format # from the repo root
```

6. Create a PR using the instructions in ~/.claude/commands/create-pr.md

7. After creating the PR, wait for Gemini Code Assist review:

```bash
timeout 600 ./scripts/wait-for-gemini-review.sh <PR_URL>
```

If the review is detected, address the feedback per `~/.claude/commands/address-pr-feedback.md`

8. Watch CI checks and fix any failures:

```bash
gh pr checks <PR_NUMBER> --watch
```

If any checks fail, fix them locally, push, and re-watch until all checks pass.

^ do not miss any steps! The PR is NOT complete until both Gemini feedback is addressed AND all CI checks pass.
