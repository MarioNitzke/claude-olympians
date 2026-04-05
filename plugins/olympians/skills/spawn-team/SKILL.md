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
- **Task** — what needs to be done (OPTIONAL — if not provided, just spawn the roles without a specific task)
- **Roles** — which roles to spawn (by name)
- **Phases** — if provided (typically from plan-team). If NOT provided, infer them automatically from the roles' declared phase preferences and contract-first ordering.
- **Team Lead Instructions** — if provided (from plan-team's Step 5b)

The only thing required is at least one role name. Task is optional — if the user just says `/olympians:spawn-team backend-developer frontend-developer`, spawn them without a task description.

If `$ARGUMENTS` is completely empty (no roles, nothing), say:

> Tell me what to spawn. Example:
> `/olympians:spawn-team backend-developer frontend-developer`
> `/olympians:spawn-team fix booking API with backend-developer and qa-reviewer`

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

## Step 5: Launch Immediately

No preview. No confirmation. No "Ready to launch?". Just do it.

Spawn a real **Agent Team** (separate Claude Code CLI instances, NOT subagents). Use the `TeamCreate` tool to create the team. The current session becomes the **team lead** that coordinates but does not implement.

1. Use `TeamCreate` to create an agent team with a descriptive team name
2. Pass the full spawn prompt as natural language — it must be **self-contained** (teammates receive ONLY this context plus any CLAUDE.md in the repo)
3. Claude Code will spawn each teammate as a separate CLI instance

The spawn prompt must include:
- Mission statement — what the task is and what "done" looks like
- Phases — numbered, with entry/exit conditions and assigned roles
- Teammate specifications — for each role: model, tools, isolation, system prompt, file ownership
- Execution rules — phase gating, contract sharing, file ownership enforcement
- Delegate mode — team lead coordinates only, does not implement

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
