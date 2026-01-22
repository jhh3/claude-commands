Please create a PR for the current branch using the gh and any git commands you need.

Briefly summarize the the changes we are making in this PR. This should be a concise description of the changes / bulleted list of changes.

If you need context, diff the current branch against the main branch.

It is critical you commit all outstanding changes before creating the PR.
It is also critical that you run `make total-format` from the root of the repo to ensure formatting is correct.

IMPORTANT! Do not put "Generated with Claude Code" or "Co-Authoried-By: ..." or anything similar in the PR description.

## After Creating the PR

After the PR is created, check if the Gemini Code Assist review script exists:

```bash
test -f ./scripts/wait-for-gemini-review.sh && echo "exists"
```

If the script exists, run it with a 5 minute timeout to wait for the Gemini Code Assist review:

```bash
timeout 600 ./scripts/wait-for-gemini-review.sh <PR_URL>
```

Where `<PR_URL>` is the URL of the PR you just created (e.g., `https://github.com/Nullframe/studio/pull/1234`).

**Only if the Gemini review is successfully detected**, follow the instructions in `~/.claude/commands/address-pr-feedback.md` to fetch and address any PR feedback.

If the script doesn't exist or times out, do NOT proceed to address PR feedback - just report the PR URL to the user and stop.

## After Addressing PR Feedback

After addressing PR feedback and pushing changes, check the GitHub status checks to see if any CI jobs failed:

```bash
gh pr checks <PR_NUMBER> --watch
```

If any checks fail (tests, linting, formatting, typecheck, etc.):

1. Review the failed check output to understand what failed
2. Fix the issues locally
3. Run relevant local checks before pushing (e.g., `make total-format`, `npm run typecheck`, `npm test`, `cd backend && make check`)
4. Commit and push the fixes
5. Re-run `gh pr checks <PR_NUMBER> --watch` to verify all checks pass

Do not consider the PR ready until all status checks are passing.
