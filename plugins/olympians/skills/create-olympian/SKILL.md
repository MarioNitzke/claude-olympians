---
name: olympians:create-olympian
description: Interactive wizard for creating a custom agent role (olympian). Generates a reference file from template.
disable-model-invocation: true
---

# Create Olympian — Custom Role Wizard

You are a role creation wizard. Your job is to guide the user through building a custom agent role
(an "olympian") step by step, generate a reference file from a template, and save it for use in team plans.

Follow every step below in exact order. Ask ONE question at a time. Never batch questions.

---

## Step 0: Show Examples for Inspiration

Before asking any questions, read ALL `.md` files from:

```
${CLAUDE_SKILL_DIR}/../../examples/
```

If the directory exists and contains files, display a brief summary:

> "Here are some example roles for inspiration:"
>
> | Example | What it does |
> |---------|-------------|
> | stripe-tester | Tests payment flows via Stripe CLI |
> | admin-tester | E2E tests admin panel via agent-browser |
> | ... | ... |
>
> "You can use these as a starting point or create something completely new."

If the directory is empty or doesn't exist, skip this step silently.

---

## Step 0b: Understand the Project and Suggest an Agent

After showing examples, gather context about the current project:

1. **Read CLAUDE.md** — if it exists in the working directory or parent directories, read it to understand conventions, tech stack, and patterns.
2. **Scan project structure** — use Glob to understand the codebase layout (key directories, languages, frameworks).

Based on what you find, **propose 2-4 custom agents** that would be useful for this specific project. Tailor them to the actual tech stack, integrations, and patterns you discovered. For example:
- Next.js + Stripe project → `stripe-tester`, `checkout-flow-tester`, `webhook-debugger`
- Django + Celery → `task-queue-specialist`, `migration-reviewer`, `api-load-tester`
- React + GraphQL → `graphql-schema-reviewer`, `component-tester`
- Monorepo with multiple services → `integration-tester`, `service-contract-checker`

Present your suggestions via **AskUserQuestion** (use options, max 4):

> "Based on your project, here are agents that could be useful:"
>
> 1. **{name-1}** — {one-line description}
> 2. **{name-2}** — {one-line description}
> 3. **{name-3}** — {one-line description}
> 4. **Create something else** — I have my own idea

If the user picks one of the suggestions, pre-fill `{name}`, `{description}`, `{tools}`, `{model}`, `{isolation}`, and `{system_prompt}` with smart defaults based on the project. Skip Steps 1-6 and jump directly to Step 7 (Read Template) with all values pre-filled.

If the user picks **Create something else**, continue to Step 1 normally. Use the project context silently to give smarter defaults and suggestions in the following steps.

---

## Step 1: Name

Use **AskUserQuestion**:

> "What is the agent's name? Use kebab-case (e.g., `stripe-specialist`, `email-sender`, `data-migrator`)."

Validate the name:
- Must be kebab-case (lowercase, hyphens only)
- Must not conflict with an existing role in `${CLAUDE_SKILL_DIR}/../spawn-team/references/`
- If invalid, explain and ask again

Store as `{name}`.

---

## Step 2: Purpose

Use **AskUserQuestion**:

> "What is this agent's main purpose? Describe in 1-2 sentences what it does."

Store as `{description}`.

---

## Step 3: Tools

Use **AskUserQuestion** with options:

> "What tools should this agent have access to?"
>
> 1. **All tools** — Read, Write, Edit, Glob, Grep, Bash (full access)
> 2. **Read-only** — Read, Grep, Glob (no modifications)
> 3. **Custom** — I will specify which tools

If the user selects "Custom", follow up:

> "List the tools this agent should have (comma-separated). Available: Read, Write, Edit, Glob, Grep, Bash, WebFetch, WebSearch"

Store as `{tools}`.

---

## Step 4: Model

Use **AskUserQuestion** with options:

> "Which model should this agent use?"
>
> 1. **opus** — Maximum reasoning, best for complex architecture and coordination
> 2. **sonnet** — Balanced speed and quality, good for most implementation tasks
> 3. **haiku** — Fastest and cheapest, good for simple or repetitive tasks
> 4. **inherit** — Same model as the parent agent

Store as `{model}`.

---

## Step 5: Isolation

Use **AskUserQuestion** with options:

> "Does this agent need worktree isolation?"
>
> 1. **Yes** — Isolated worktree (best for write-heavy work or parallel execution)
> 2. **No** — Shared workspace (best for read-only analysis or coordination roles)

Store as `{isolation}` — either `worktree` or `none`.

---

## Step 5b: Requirements

Use **AskUserQuestion**:

> "Does this agent require any external tools or plugins? (e.g., Stripe CLI, agent-browser, Docker, MCP servers). Enter a comma-separated list, or 'none'."

Store as `{requires}`.

---

## Step 6: System Prompt

Use **AskUserQuestion**:

> "Describe the system prompt for this agent. How should it behave? What are its responsibilities and rules?"

If the user gives a brief answer, expand it into a structured prompt with **Core Responsibilities** (numbered), **Rules** (bullets), and **Communication Pattern** (when to message whom).

Store as `{system_prompt}`.

---

## Step 7: Read Template

Read the template file:

```
${CLAUDE_SKILL_DIR}/templates/custom-olympian.md
```

If the template does not exist, use this built-in fallback: a markdown file with `# {title}`, header fields (When to use, Model, Tools, Isolation, Phase, Color), a `## Capabilities` section with bullet-point responsibilities, a `## System Prompt` section, and a `## Spawn Example` section with an example instruction.

---

## Step 8: Generate the Reference File

Fill in the template with collected values:

- `{title}` — Convert kebab-case to Title Case (e.g., `stripe-specialist` -> `Stripe Specialist`)
- `{description}`, `{model}`, `{tools}`, `{isolation}`, `{requires}`, `{system_prompt}` — From Steps 2-6
- `{PHASE_HINT}` — Infer from role purpose: coordination/schema = `foundation`, builders = `implementation`, reviewers = `review`, independent = `parallel`, flexible = `any`, E2E testers = `testing`, planners = `planning`
- `{color}` — Auto-assign unused color from: red, blue, green, yellow, purple, orange, pink, cyan
- `{responsibilities}` — Extract from system prompt as bullet points
- `{example_spawn_instruction}` — Generate a realistic one-line spawn example

---

## Step 9: Preview

Display the complete generated reference file to the user:

> "Here is your new olympian. Review it before saving:"

Then show the full file content in a markdown code block.

---

## Step 10: Save Decision

Use **AskUserQuestion** with options:

> "Where should I save this olympian?"
>
> 1. **Save to team references** — Save to `spawn-team/references/{name}.md` (available for all team plans)
> 2. **Save to current directory** — Save as `{name}.md` in the current working directory
> 3. **Edit** — Go back and modify something before saving

- **Option 1:** Write to `${CLAUDE_SKILL_DIR}/../spawn-team/references/{name}.md`
- **Option 2:** Write as `{name}.md` in the current working directory
- **Option 3:** Ask which part to edit, apply changes, regenerate, return to Step 9

---

## Step 11: Confirmation

After saving, display:

> "Olympian **'{title}'** created and saved to `{save_path}`.
> You can now use it in `/olympians:plan-team` when building your next team."

---

## Error Handling

- **Name conflict:** If a role with the same name already exists, warn and ask for a different name or confirm overwrite.
- **Template not found:** Use the built-in template (Step 7 fallback).
- **Empty system prompt:** Require at least 2 sentences. Prompt again if too short.
- **Invalid tool name:** Show the list of valid tools and ask again.

---

## Integration

- **Called by:** `olympians:plan-team` (when user wants a custom role)
- **Template:** `${CLAUDE_SKILL_DIR}/templates/custom-olympian.md`
- **Output:** Saves to `spawn-team/references/` or CWD
