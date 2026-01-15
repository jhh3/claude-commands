# Complete a Linear Issue

$ARGUMENTS

---

## PHASE 1: INTAKE

### Step 1.1: Fetch Issue Context

```
mcp__linear__get_issue(id: "<issue-id>")
mcp__linear__list_comments(issueId: "<issue-id>")
```

Extract and record to `@issue-context.md`:

- **ID**: <issue-id>
- **Title**: <title>
- **Description**: <description>
- **Acceptance Criteria**: (if any)
- **Comments/Discussion**: (summarize relevant context)
- **Suggested Branch**: <branch-name>

### Step 1.2: Branch Setup

If not already on the suggested branch:

```bash
git checkout -b <suggested-branch-name>
```

---

## PHASE 2: CLASSIFICATION

Evaluate the issue against these criteria:

| Factor                  | EASY                         | COMPLEX                          |
| ----------------------- | ---------------------------- | -------------------------------- |
| Files touched           | 1-3 files                    | 4+ files                         |
| Systems affected        | Single module                | Cross-module/cross-service       |
| New abstractions needed | None                         | New patterns/interfaces required |
| Test coverage           | Existing patterns sufficient | New test strategies needed       |
| Estimated diff          | <100 lines                   | 100+ lines                       |
| Ambiguity               | Clear path forward           | Requires design decisions        |

**Classification Rule**: If ANY factor is COMPLEX → classify as COMPLEX

Update `@issue-context.md` with classification and 1-sentence justification.

---

## PHASE 3A: EASY PATH

### Step 3A.1: Architecture Analysis

Invoke @system-architect with:

```
## Task
Analyze this change request and identify the cleanest implementation approach.

## Context
Read: @issue-context.md

## Deliverable
Provide:
1. Files to modify (with paths)
2. Approach summary (2-3 sentences)
3. Any risks or edge cases to handle
```

### Step 3A.2: Implementation

Based on the files identified, route to appropriate agent(s):

**For backend/logic changes** → Invoke @code-architect:

```
## Task
Implement the following change.

## Context
Read: @issue-context.md

## Architecture Decision
<summary from 3A.1>

## Files to Modify
<list from 3A.1>

## Constraints
- Follow existing patterns in the codebase
- Maintain type safety
```

**For frontend/UI changes** → Invoke @ui-ux-designer:

```
## Task
Implement the following UI change.

## Context
Read: @issue-context.md

## Architecture Decision
<summary from 3A.1>

## Files to Modify
<list from 3A.1>

## Constraints
- Match existing component patterns
- Ensure responsive behavior
```

### Step 3A.3: Simplification Review

Invoke @code-simplifier:code-simplifier with:

```
## Task
Review this changeset for unnecessary complexity. Suggest simplifications only - do not edit files.

## Context
Read: @issue-context.md

## Changeset
<git diff output or list of changes made>

## Respond with
- APPROVED (no changes needed), or
- SUGGESTIONS: <numbered list of specific simplifications>
```

**If suggestions returned**: Apply relevant simplifications, then proceed.

### Step 3A.4: Validation

Run the project's lint, format, typecheck, and test commands as defined in CLAUDE.md or package.json. Fix any failures before proceeding.

### Step 3A.5: Create PR

Follow instructions in `~/.claude/commands/create-pr.md`

→ Continue to PHASE 4

---

## PHASE 3B: COMPLEX PATH

### Step 3B.1: Problem Analysis

Create directory: `docs/linears/<issue-id>/`

Invoke @system-architect with:

```
## Task
Analyze this issue and document the problem space.

## Context
Read: @issue-context.md

## Research Instructions
1. Identify all code paths affected
2. Map current data flow / state management
3. Note any technical debt or constraints
4. Use web search for external dependencies (e.g., langgraph docs via MCP)

## Deliverable
Write `docs/linears/<issue-id>/01-problem-statement.md` containing:
1. **Problem Summary** (2-3 sentences)
2. **Current Behavior** (how it works today)
3. **Desired Behavior** (what we need)
4. **Constraints** (technical limitations, dependencies)
5. **Open Questions** (if any)
```

### Step 3B.2: Solution Design

Invoke @system-architect with:

```
## Task
Design a solution for the documented problem.

## Context
- Read: @issue-context.md
- Read: `docs/linears/<issue-id>/01-problem-statement.md`

## Design Principles (MANDATORY)
- We ship fast. No incremental migrations.
- Full cutovers only. No legacy support paths.
- Prefer deletion over deprecation.
- Simpler is better, even if more work upfront.

## Deliverable
Write `docs/linears/<issue-id>/02-design.md` containing:

1. **Solution Overview** (1 paragraph)

2. **Implementation Phases**
   For each phase:
   - Phase name
   - Files to create/modify/delete
   - Key changes (bullet points)
   - Tests to write (for logic-heavy sections)
   - Definition of done

3. **Data Migration** (if applicable)
   - Migration strategy
   - Rollback plan (even if "delete and re-run")

4. **Risks & Mitigations**

5. **Checklist**
   - [ ] Phase 1: <name>
   - [ ] Phase 2: <name>
   - [ ] ...
```

### Step 3B.3: Design Review (Loop max 3x)

**Iteration counter**: Start at 1

Invoke BOTH agents with the same context:

```
## Task
Review this design document. Provide feedback only - do not edit.

## Context
- Read: @issue-context.md
- Read: `docs/linears/<issue-id>/02-design.md`

## Review Criteria
- Is the solution the simplest path to the goal?
- Are there unnecessary abstractions?
- Does it avoid incremental migration? (REQUIRED)
- Are the phases correctly scoped?
- Are edge cases handled?

## Design Principles (for context)
- We ship fast. No incremental migrations.
- Full cutovers only. No legacy support.

## Respond with ONE of:
- APPROVED: <1-sentence confirmation>
- FEEDBACK: <numbered list of specific concerns>
```

**Agents to invoke**:

1. @code-simplifier:code-simplifier
2. @arch-code-reviewer

**After both respond**:

- If BOTH approved → proceed to Step 3B.4
- If ANY feedback:
  - Invoke @system-architect to address feedback and update `02-design.md`
  - Increment counter
  - If counter ≤ 3 → repeat Step 3B.3
  - If counter > 3 → log unresolved feedback and proceed

### Step 3B.4: Phased Implementation

For EACH phase in `02-design.md`:

#### 3B.4.a: Implement Phase

Route based on change type:

**Backend/Logic** → @code-architect:

```
## Task
Implement Phase <N>: <phase-name>

## Context
- Read: @issue-context.md
- Read: `docs/linears/<issue-id>/02-design.md`, Phase <N> section

## Scope
- Files: <list from design>
- Tests: <list from design>

## Constraints
- Only implement this phase
- Follow the design exactly
- Write tests specified in design
```

**Frontend/UI** → @ui-ux-designer:

```
## Task
Implement Phase <N>: <phase-name> (UI components)

## Context
- Read: @issue-context.md
- Read: `docs/linears/<issue-id>/02-design.md`, Phase <N> section

## Scope
- Files: <list from design>

## Constraints
- Only implement this phase
- Follow the design exactly
```

#### 3B.4.b: Commit Phase

```bash
git add -A
git commit -m "feat(<issue-id>): Phase <N> - <phase-name>"
git push
```

#### 3B.4.c: Phase Review (Loop max 3x)

**Iteration counter**: Start at 1

Invoke BOTH:

```
## Task
Review this changeset. Provide feedback only - do not edit.

## Context
- Read: @issue-context.md
- Changeset: <git diff for this phase>
- Design reference: `docs/linears/<issue-id>/02-design.md`, Phase <N>

## Respond with ONE of:
- APPROVED
- FEEDBACK: <numbered list>
```

**Agents**: @code-simplifier:code-simplifier, @arch-code-reviewer

**After both respond**:

- If BOTH approved → proceed to 3B.4.e
- If ANY feedback:
  - Invoke @code-architect to address
  - Commit and push
  - Increment counter
  - If counter ≤ 3 → repeat 3B.4.c
  - If counter > 3 → log unresolved and proceed

#### 3B.4.d: Update Design Doc

Mark phase complete in `02-design.md` checklist:

```markdown
- [x] Phase <N>: <name>
```

#### 3B.4.e: Next Phase

Repeat 3B.4 for next phase until all complete.

### Step 3B.5: Final Validation

Run the project's lint, format, typecheck, and test commands as defined in CLAUDE.md or package.json. Fix any failures before proceeding.

### Step 3B.6: Create PR

Follow `~/.claude/commands/create-pr.md`

→ Continue to PHASE 4

---

## PHASE 4: CLOSE OUT

### Step 4.1: Linear Comment

Post comment to the Linear issue:

```
mcp__linear__create_comment(
  issueId: "<issue-id>",
  body: "## Implementation Complete

**PR**: <link>

**Summary**: <2-3 sentences on what was done>

**Notable decisions**:
- <any non-obvious choices, or 'None'>

**Testing**: <brief note on test coverage>"
)
```

Keep it concise. Max 100 words.

---

## ORCHESTRATION RULES

1. **Context is mandatory**: Never invoke a sub-agent without providing `@issue-context.md` reference.

2. **Document references**: Always point agents to read specific files rather than pasting content inline.

3. **One job per invocation**: Each agent call should have a single, clear deliverable.

4. **Explicit handoffs**: When one agent's output feeds another, specify exactly what to pass.

5. **Fail forward**: If a step fails after retries, document the failure and continue rather than blocking indefinitely.

6. **Commit often**: For COMPLEX path, commit after each phase - not at the end.

---

## ERROR HANDLING

If any step fails 3 consecutive times:

1. Document the failure in `docs/linears/<issue-id>/failures.md`
2. Post a Linear comment describing the blocker
3. Stop execution and notify user
