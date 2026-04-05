# Contributing to Olympians

## Creating Custom Roles

The easiest way to create a new role:

```bash
/olympians:create-olympian
```

The wizard guides you through name, purpose, tools, model, isolation, and system prompt.

Alternatively, copy an existing role and modify it:

```bash
cp examples/stripe-tester.md skills/spawn-team/references/my-role.md
```

## Role File Format

Every role is a markdown file following the template at `skills/create-olympian/templates/custom-olympian.md`. Required fields:

- **Name** (`# Title` heading)
- **When to use** — one sentence
- **Model** — opus, sonnet, or haiku
- **Tools** — all, Read/Grep/Glob (read-only), or custom list
- **Isolation** — worktree or none
- **Phase** — foundation, implementation, review, parallel, any, testing, or planning
- **Color** — red, blue, green, yellow, purple, orange, pink, or cyan
- **Requires** — external dependencies (optional)
- **Capabilities** — bullet list
- **System Prompt** — detailed instructions with Core Responsibilities, Rules, Communication Pattern
- **Done Criteria** — 3-4 concrete completion conditions
- **Spawn Example** — realistic one-liner

## Naming Conventions

- File names: `kebab-case.md` (e.g., `stripe-tester.md`)
- Role titles: Title Case (e.g., `Stripe Tester`)

## Submitting Example Roles

1. Create your role using the template or wizard
2. Test it in a real agent team session
3. Place it in `examples/`
4. Submit a PR with a brief description of what the role does and when to use it

## Guidelines

- Example roles should demonstrate a capability that generic roles do not cover
- Include a `**Requires:**` field listing external dependencies
- Keep system prompts focused — 50-150 lines is the sweet spot
- Include a realistic spawn example that shows task assignment
