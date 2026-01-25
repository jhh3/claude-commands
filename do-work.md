# Execute Work Task

$ARGUMENTS

---

## CRITICAL: SUB-AGENT REQUIREMENT

**You MUST use the Task tool to invoke sub-agents throughout this workflow.** This is NON-NEGOTIABLE. Do NOT attempt to do the work yourself when the instructions say to invoke an agent.

When you see "Invoke @agent-name", you MUST:
1. Use the `Task` tool with `subagent_type` set to that agent name
2. Pass the full prompt/context as specified in the instruction
3. Wait for the agent's response before proceeding

**Available agents (use these exact names for subagent_type):**
- `system-architect` - for analysis and design
- `code-architect` - for implementation
- `code-simplifier:code-simplifier` - for simplification reviews
- `arch-code-reviewer` - for code reviews

**Example Task tool usage:**
```
Task(
  subagent_type="system-architect",
  description="Analyze work request",
  prompt="<the full prompt from the step>"
)
```

**WHY THIS MATTERS:** Sub-agents provide specialized expertise and fresh context. Doing the work yourself instead of delegating defeats the purpose of this orchestrated workflow.

---

## SKIP CONDITIONS

**Skip Phase 1 (Analysis) if ANY of the following apply:**
- The user explicitly says to skip analysis or "just do it"
- Sufficient context was already discussed in the conversation
- The user provides specific files and approach in their request
- This is a continuation of previous work where analysis was already done

**Skip Phase 2B (Complex Design) if:**
- The user provides a detailed implementation plan
- A design document already exists
- The user explicitly requests to proceed directly to implementation

When skipping, proceed directly to the appropriate implementation step using the context already available.

---

## PHASE 1: TASK ANALYSIS

### Step 1.1: Understand the Request

Parse the task description and identify:

- **Objective**: What needs to be accomplished
- **Scope**: Which parts of the codebase are involved
- **Type**: Feature, bugfix, refactor, or chore
- **Constraints**: Any specific requirements or limitations mentioned

### Step 1.2: Codebase Discovery

**MANDATORY:** Use the Task tool to invoke @system-architect with:

```
## Task
Analyze this work request and identify the implementation approach.

## Request
<task description from arguments>

## Deliverable
Provide:
1. Files to examine/modify (with paths)
2. Related files that may need updates (tests, types, etc.)
3. Approach summary (2-3 sentences)
4. Any risks, edge cases, or concerns
5. Classification: EASY (1-3 files, clear path) or COMPLEX (4+ files, design decisions needed)
```

---

## PHASE 2A: EASY PATH (1-3 files, straightforward changes)

### Step 2A.1: Implementation

**MANDATORY:** Use the Task tool to invoke @code-architect with:

```
## Task
Implement the following change.

## Request
<task description from arguments>

## Architecture Analysis
<summary from Step 1.2>

## Files to Modify
<list from Step 1.2>

## Constraints
- Follow existing patterns in the codebase
- Maintain type safety
- Keep changes minimal and focused
- Write tests if the change involves logic
```

### Step 2A.2: Simplification Review

**MANDATORY:** Use the Task tool to invoke @code-simplifier:code-simplifier with:

```
## Task
Review this changeset for unnecessary complexity.

## Changeset
<git diff output>

## Review Criteria
- Are there any unnecessary abstractions?
- Can anything be simplified without losing functionality?
- Are there redundant patterns?

## Respond with
- APPROVED (no changes needed), or
- SUGGESTIONS: <numbered list of specific simplifications>
```

**If suggestions returned**: Apply relevant simplifications, then proceed.

→ Continue to PHASE 3

---

## PHASE 2B: COMPLEX PATH (4+ files, requires design)

### Step 2B.1: Design Document

**MANDATORY:** Use the Task tool to invoke @system-architect with:

```
## Task
Design a solution for this work request.

## Request
<task description from arguments>

## Analysis
<output from Step 1.2>

## Design Principles
- Prefer simple solutions over clever ones
- Follow existing codebase patterns
- Minimize surface area of changes
- Consider testability

## Deliverable
Provide an implementation plan with:
1. Solution overview (1 paragraph)
2. Implementation phases (ordered list of discrete steps)
3. For each phase: files to modify, key changes, tests needed
4. Risks and mitigations
```

### Step 2B.2: Design Review

**MANDATORY:** Use the Task tool to invoke BOTH agents IN PARALLEL with the same context:

```
## Task
Review this design. Provide feedback only - do not edit.

## Design
<design from Step 2B.1>

## Review Criteria
- Is the solution the simplest path to the goal?
- Are there unnecessary abstractions?
- Are the phases correctly scoped?
- Are edge cases handled?

## Respond with ONE of:
- APPROVED: <1-sentence confirmation>
- FEEDBACK: <numbered list of specific concerns>
```

**Agents to invoke**:
1. @code-simplifier:code-simplifier
2. @arch-code-reviewer

**After both respond**:
- If BOTH approved → proceed to Step 2B.3
- If ANY feedback → address concerns and re-review (max 3 iterations)

### Step 2B.3: Phased Implementation

For EACH phase in the design:

1. **MANDATORY:** Use Task tool to invoke @code-architect to implement the phase
2. **MANDATORY:** Use Task tool to invoke @code-simplifier:code-simplifier to review
3. Apply any necessary simplifications
4. Commit the phase: `git add -A && git commit -m "<descriptive message>"`

→ Continue to PHASE 3

---

## PHASE 3: VALIDATION

### Step 3.1: Code Quality Checks

Run the project's standard validation commands. Check for:
- Linting errors
- Type errors
- Formatting issues
- Test failures

Common commands (adjust based on project):
```bash
# Check CLAUDE.md or package.json for project-specific commands
make lint       # or npm run lint
make typecheck  # or npm run typecheck
make test       # or npm run test
make format     # or npm run format
```

Fix any failures before proceeding.

### Step 3.2: Final Review

**MANDATORY:** Use the Task tool to invoke @arch-code-reviewer with:

```
## Task
Final review of all changes made.

## Context
<original task description>

## Changeset
<git diff from start of work>

## Review for
- Does this fully address the original request?
- Are there any bugs or issues?
- Is the code maintainable?
- Are there any security concerns?

## Respond with
- APPROVED, or
- ISSUES: <list of problems to address>
```

Address any issues raised before completing.

---

## PHASE 4: COMPLETION

### Step 4.1: Summary Report

Provide a summary to the user including:

1. **What was done**: Brief description of changes made
2. **Files modified**: List of files changed
3. **Testing**: Description of test coverage added/updated
4. **Notes**: Any important decisions or trade-offs made

### Step 4.2: Next Steps (if applicable)

If the user needs to take action (create PR, deploy, etc.), provide clear instructions.

---

## ORCHESTRATION RULES

1. **ALWAYS USE THE TASK TOOL**: Every step marked "MANDATORY" or "Invoke @agent" MUST use the Task tool. DO NOT skip this. DO NOT do the work yourself. The sub-agents exist for a reason.

2. **Always provide full context**: Sub-agents need the original task description and relevant analysis.

3. **One job per invocation**: Each agent call should have a single, clear deliverable.

4. **Document decisions**: When agents make non-obvious choices, capture the reasoning.

5. **Fail forward**: If a step fails after 3 retries, document the failure and notify the user rather than blocking.

6. **Keep it simple**: Default to the simplest solution that fully addresses the request.

**REMINDER:** If you find yourself implementing code, doing analysis, or reviewing changes WITHOUT having used the Task tool to invoke the appropriate sub-agent, you are doing it wrong. STOP and use the Task tool.

---

## ERROR HANDLING

If any step fails 3 consecutive times:

1. Document what was attempted and what failed
2. Provide the user with:
   - Current state of the work
   - What remains to be done
   - Specific blockers encountered
3. Ask for guidance on how to proceed
