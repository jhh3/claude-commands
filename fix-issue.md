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

^ do not miss any steps! it's critical that you don't forget to run the script to wait for a review.
