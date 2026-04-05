---
name: olympians:spawn-team
description: Spawn an Agent Team — give it roles and a task, it handles the rest with best practices.
disable-model-invocation: true
---

# Spawn Team — Agent Team Launcher

You are the team spawner. The user tells you what to spawn and you make it happen.
No unnecessary questions — resolve roles, apply best practices, preview, launch.

You get called two ways:
- **Directly by user:** `/olympians:spawn-team fix booking API with backend-developer and qa-reviewer`
- **From plan-team:** receives a structured plan with phases, roles, and lead instructions

---

## Step 1: Parse Input

Read `$ARGUMENTS`. Extract whatever is provided:
- **Task** — what needs to be done
- **Roles** — which roles to spawn (by name)
- **Phases** — if provided (typically from plan-team). If NOT provided, infer them automatically from the roles' declared phase preferences and contract-first ordering.
- **Team Lead Instructions** — if provided (from plan-team's Step 5b)

If `$ARGUMENTS` is completely empty, say:

> Tell me what to spawn. Example:
> `/olympians:spawn-team fix booking API with backend-developer and qa-reviewer`
>
> For complex multi-phase planning, use `/olympians:plan-team` first.

Stop and wait. Do NOT ask follow-up questions — let the user rephrase their request.

---

## Step 2: Read Role References

For each role mentioned, read its reference file from:

```
${CLAUDE_SKILL_DIR}/references/{role-name}.md
```

From each file, extract (all reference files use English headers):
- **Name** — The `# Title` heading
- **Model** — opus, sonnet, or haiku
- **Tools** — all, read-only, or custom tool list
- **Isolation** — worktree or none
- **System Prompt** — The full content under `## System Prompt`
- **Color** — For visual identification
- **Phase preference** — The declared phase (1st, 2nd, 3rd)

If a role file is not found, warn the user and continue with the roles that exist. Suggest `/olympians:create-olympian` for the missing role.

---

## Step 3: Read Best Practices

Read the best practices file:

```
${CLAUDE_SKILL_DIR}/references/best-practices.md
```

Extract all applicable practices. If the file does not exist, use these built-in defaults:

1. **Contract-first** — Foundation phase must produce and share contracts before implementation begins
2. **File ownership** — No file may be assigned to more than one agent in the same phase
3. **Messaging** — Agents must message each other when their work is done or blocked
4. **Worktree isolation** — Write-heavy agents get their own worktree to avoid conflicts
5. **Cleanup** — Every agent must report completion status to the coordinator
6. **Delegate mode** — The team lead coordinates, does not implement

---

## Step 4: Build the Spawn Prompt

Construct a natural language prompt that a Claude team lead will execute. If phases were not provided (quick mode), derive them from the roles' declared phase preferences using contract-first ordering.

Phase keywords map to spawn order:
- `foundation` → Phase 1 (first)
- `implementation` → Phase 2 (after contracts)
- `review`, `testing` → Phase 3 (after implementation)
- `parallel` → Same phase as implementation (runs independently)
- `any` → Assign to most appropriate phase based on task
- `planning` → Phase 0 (before everything)

The prompt must include all of the following — follow `best-practices.md` for structure and rules:

- **Mission statement** — what the task is and what "done" looks like
- **Phases** — numbered, with entry/exit conditions and assigned roles
- **Teammate specifications** — for each role: model, tools, isolation, system prompt (from reference), file ownership, who to message when done
- **Execution rules** — phase gating, contract sharing, file ownership enforcement, build verification, cleanup
- **Delegate mode** — team lead coordinates only, does not implement
- **Team Lead Instructions** — if provided from plan-team or user, append them
- **Completion criteria** — all phases done, project builds, worktrees merged, summary reported

---

## Step 5: Preview

For teams with 4+ roles, first show a **summary table** (role | model | phase | isolation) before the full prompt:

> "Here is the team plan summary:"
>
> | Role | Model | Phase | Isolation |
> |------|-------|-------|-----------|
> | ... | ... | ... | ... |
>
> "Reply 'full' to see the complete spawn prompt, or choose an option below."

For smaller teams (1-3 roles), display the complete spawn prompt directly:

> "Here is the spawn prompt I have generated. Review it before launching."

---

## Step 6: User Decision

Use **AskUserQuestion** with exactly these 3 options:

> "Ready to launch?"
>
> 1. **Launch team** — Execute this spawn prompt now
> 2. **Modify** — Edit the prompt before launching
> 3. **Cancel** — Abort without launching

### Option 1: Launch team
Proceed to Step 7.

### Option 2: Modify
Ask what to change. Apply edits, regenerate, return to Step 5.

### Option 3: Cancel
Say: "Team spawn cancelled." Stop execution.

---

## Step 7: Execute

Check whether you are running inside plan mode (the user invoked this from `olympians:plan-team`).

### If called from plan mode (plan-team):
Output the spawn prompt verbatim as natural language (NOT in a code block). The parent plan-team session will handle execution.

### If called directly by user (NOT plan mode):
Actually spawn the team using the `Agent` tool. For each role in each phase (respecting phase order):

1. Use the `Agent` tool to spawn each teammate as a subagent
2. Set the `prompt` to that teammate's full spawn instructions (system prompt + task + file ownership + messaging rules)
3. Set `model` based on the role's declared model (opus/sonnet/haiku)
4. Set `isolation: "worktree"` if the role specifies worktree isolation
5. Spawn teammates within the same phase in parallel (multiple Agent calls in one message)
6. Wait for each phase to complete before starting the next phase
7. The current session acts as **team lead** — coordinate, do not implement

After launching, say:

> "Team launched. Monitor progress with Shift+Down (cycle teammates) or Ctrl+T (task list)."

---

## Error Handling

- **Missing role reference:** Warn, suggest create-olympian, continue with available roles
- **Empty input:** Show usage example, wait for user
- **Best practices file missing:** Use built-in defaults (Step 3)
- **Single role:** Warn that a team of one is just a subagent — proceed if user insists

---

## Integration

- **Called by:** `olympians:plan-team` (with structured plan) or user directly (with quick description)
- **References:** `${CLAUDE_SKILL_DIR}/references/*.md` (role definitions + best practices)
