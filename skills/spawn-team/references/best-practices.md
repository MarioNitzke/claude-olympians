# Agent Teams Best Practices

Comprehensive guide for orchestrating Claude Code Agent Teams effectively.
Consolidated from official Claude Code documentation, community research, and field experience.

---

## Contract-First Spawning

The single most important pattern for agent teams. Without it, teammates collide on
interfaces, produce incompatible code, and waste tokens on rework.

- **Spawn foundational agents first.** These are the teammates that define shared
  contracts: database schemas, API interfaces, type definitions, shared constants.
- Foundational agents finish their work and **publish contracts via `message`** to the
  lead. The lead reviews and approves before moving on.
- **Only then spawn implementation agents** (backend handlers, frontend components,
  service integrations). They consume the contracts, never invent their own.
- **Spawn reviewers last.** They need the finished code to review; spawning them early
  wastes tokens on idle polling.

Typical spawn order:

```
1. Architect / Schema agent   -->  defines DB + API contracts
2. Backend agent(s)           -->  implements handlers against contracts
3. Frontend agent(s)          -->  implements UI against API contracts
4. Reviewer agent             -->  reviews all output
```

Anti-pattern: spawning all agents simultaneously and hoping they converge.

---

## File Ownership

- **NEVER assign the same file to two teammates.** This is the #1 source of merge
  conflicts, lost work, and wasted tokens in agent teams.
- Each teammate owns a clear domain: a directory, a feature slice, a layer.
- If two teammates must contribute to the same file (e.g., a shared config), designate
  one as the owner and have the other send a message requesting the change.
- For write-heavy parallel work, use **worktree isolation** (`EnterWorktree`) so each
  teammate operates on a separate git worktree. This eliminates filesystem conflicts
  entirely.
- Document ownership in the spawn prompt: "You own `src/features/bookings/`. No other
  teammate will touch these files."

---

## Team Size

- **3-5 teammates is the sweet spot.** This balances parallelism with coordination cost.
- Assign **5-6 discrete tasks per teammate.** Fewer means underutilization; more means
  context overflow and degraded quality.
- **Beyond 5 teammates, returns diminish rapidly.** Coordination overhead (messages,
  conflict resolution) grows superlinearly while per-agent productivity plateaus.
- Token cost scales roughly linearly with team size, but rework from coordination
  failures can push it much higher.
- For small features, prefer a single agent over a team. Teams shine on multi-file,
  multi-layer changes.

---

## Communication

- **Use targeted `message` instead of `broadcast`.** Send information only to
  the teammates who need it.
- **Broadcast costs scale linearly** with team size. A broadcast to 5 teammates costs 5x
  a targeted message. Use it only for true team-wide announcements (e.g., "all contracts
  are finalized, begin implementation").
- **Keep messages concise** -- under 100 tokens when possible. Teammates receive messages
  as context, and bloated messages eat into their effective context window.
- Structure messages as actionable items: "The `BookingResponse` type now includes a
  `cameraUrl` field (string, nullable). Update your component accordingly."
- Avoid back-and-forth chatter between teammates. If coordination is complex, route it
  through the lead.

---

## Delegate Mode

- **Press Shift+Tab** to toggle delegate mode for the team lead.
- Delegate mode prevents the lead from implementing code. The lead can only coordinate,
  spawn, message, and review.
- **Use it when the lead starts "stealing" work** -- writing code instead of delegating
  to teammates. This is a common failure mode that defeats the purpose of a team.
- In delegate mode, the lead focuses on: task decomposition, spawn prompts, contract
  review, progress monitoring, conflict resolution, and final integration.
- Toggle it off temporarily if the lead needs to make a small fix that does not warrant
  a full teammate spawn.

---

## Plan Approval

- For complex or risky tasks, use **"require plan approval before implementation."**
- When enabled, each teammate produces a plan (what files to change, what approach to
  take) and submits it to the lead before writing any code.
- The lead reviews and either **approves** (teammate proceeds) or **rejects with
  feedback** (teammate revises in plan mode, no code written yet).
- This catches architectural misunderstandings, contract violations, and scope creep
  before any tokens are spent on implementation.
- Especially valuable for: database migrations, security-sensitive code, shared
  infrastructure changes.

---

## Read-Only Reviewers

- Restrict reviewer teammates' tools to **Read, Grep, Glob** only.
- This prevents accidental file modifications during review. A reviewer with write
  access might "helpfully" fix something and break another teammate's work.
- Ratio: **1 reviewer per 3-4 builders.** More reviewers slow down the pipeline without
  proportional quality gains.
- Reviewer spawn prompt should include: coding standards, security checklist, known
  anti-patterns, and the contracts produced by foundational agents.
- Reviewer output should be structured: file path, line range, severity
  (critical/warning/nit), description, suggested fix.

---

## Lifecycle Management

- **Always shut down through the team lead.** Use `requestShutdown` which triggers
  proper cleanup: saving state, committing partial work, reporting final status.
- **Never manually kill teammates** (Ctrl+C, process kill). This can leave files in
  inconsistent states, uncommitted changes, and leaked worktrees.
- After shutdown, the lead should produce a summary: what was completed, what remains,
  any known issues.
- **`/resume` does NOT restore in-process teammates.** It resumes the lead's
  conversation but teammates are gone. Plan accordingly -- ensure teammates commit
  frequently so work is not lost.
- If a teammate hangs or loops, ask the lead to shut it down gracefully rather than
  force-killing.

---

## Worktree Merge Protocol

When teammates use worktree isolation, their work exists on separate branches. The team lead
must merge these worktrees before declaring the team's work complete.

**Merge Order:**
1. Merge foundational worktrees first (db-specialist, architect)
2. Merge implementation worktrees next (backend-developer, frontend-developer)
3. Merge test worktrees last (test-writer)
4. Resolve any merge conflicts -- the lead should understand both sides from teammate messages

**Before Merging:**
- Verify the teammate reported completion successfully
- Check that the teammate's branch builds/compiles
- Review any warnings or notes in the teammate's completion message

**After Merging:**
- Run the full build on the merged branch
- Run the full test suite
- If conflicts arise, consult the affected teammates' messages for context

---

## Cost and Token Management

- Each teammate runs its own context window. A 5-agent team uses roughly 5x the tokens
  of a single agent for the same wall-clock period.
- Front-load context in spawn prompts so teammates do not waste tokens re-discovering
  project structure.
- Set clear "done" criteria in spawn prompts so teammates stop when finished rather
  than gold-plating.
- Monitor teammate progress. If one is stuck in a loop, intervene early -- loops burn
  tokens fast.

---

## When to Use Teams (and When Not To)

**Use teams when:**
- The task spans 3+ files across different layers (DB, backend, frontend).
- Subtasks are genuinely parallelizable (not sequential dependencies).
- Total estimated work exceeds 15-20 minutes for a single agent.

**Do NOT use teams when:**
- The task is sequential (each step depends on the previous).
- Only 1-2 files are involved.
- The task is exploratory/investigative (use a single agent with deep context).
- You need tight iterative feedback (single agent is more responsive).

---

## Start with Research, Not Code

If you're new to agent teams, start with read-only tasks: reviewing a PR, researching a library, investigating a bug. These show the value of parallel work without the coordination challenges of parallel implementation.

---

## Monitor and Steer

Don't set it and forget it. Check teammates' progress, redirect approaches that aren't working, and synthesize findings as they come in. An unattended team burns tokens on dead ends.

---

## Pre-Approve Permissions

Teammate permission requests bubble up to the lead, creating friction. Pre-approve common operations in your permission settings before spawning to reduce interruptions.

---

## Predictable Teammate Names

Tell the lead what to name each teammate in your spawn instruction. Predictable names let you reference them in later prompts: "message the api-builder" instead of guessing auto-generated names.

---

## Limitations to Know

- **One team per session** — clean up the current team before starting a new one
- **No nested teams** — teammates cannot spawn their own teams
- **Lead is fixed** — can't promote a teammate to lead
- **Permissions set at spawn** — all teammates inherit the lead's mode (changeable after)

---

## Anti-Patterns

| Anti-Pattern | Why It Fails | Fix |
|---|---|---|
| Spawning all agents at once | No contracts, everyone guesses interfaces | Contract-first spawning |
| Using teams for sequential tasks | Teammates idle-wait, wasting tokens | Single agent instead |
| Letting agents modify AGENTS.md/CLAUDE.md | Config drift, conflicting instructions | Lead-only or manual edits |
| Vague task descriptions in spawn prompts | Teammates wander, produce wrong output | Specific files, contracts, done criteria |
| Not providing context in spawn prompts | Teammates lack project knowledge (they do NOT inherit lead history) | Include architecture, patterns, relevant file paths |
| Assigning same file to multiple teammates | Merge conflicts, lost work | Strict file ownership |
| Over-broadcasting | Token waste, context pollution | Targeted messages |
| Skipping plan approval on complex tasks | Expensive rework after wrong approach | Require plan approval |
| Not shutting down through lead | Inconsistent state, leaked resources | Always use requestShutdown |
| Spawning > 5 teammates | Diminishing returns, high cost | Cap at 5, increase tasks per agent |
| Gold-plating (teammate exceeds scope) | Wasted tokens, bugs in finished code | Clear done criteria + file ownership boundaries |

---

## Spawn Prompt Checklist

Every spawn prompt should include:

1. **Role and goal** -- what this teammate is and what it must accomplish.
2. **Owned files/directories** -- explicit list of what it may modify.
3. **Contracts/interfaces** -- the types, schemas, and APIs it must conform to.
4. **Done criteria** -- how the teammate knows it is finished.
5. **Constraints** -- what it must NOT do (e.g., "do not modify any file outside `src/features/payments/`").
6. **Project patterns** -- coding conventions, naming rules, architecture patterns relevant to its task.
7. **Communication instructions** -- who to message and when.

---

## Keyboard Shortcuts & Navigation

- **Shift+Down** / **Shift+Up** — cycle through teammates (in-process mode)
- **Ctrl+T** — toggle shared task list
- **Enter** on focused teammate — view their session
- **Escape** — interrupt teammate's current turn
- **Shift+Tab** — toggle delegate mode (lead can't code, only coordinate)

---

## Quality Gate Hooks

- **`TeammateIdle` hook** — runs when a teammate finishes its current turn. Exit code 2 sends feedback to the teammate (e.g., "you missed a file").
- **`TaskCompleted` hook** — runs when a task is marked complete. Exit code 2 prevents completion and returns the teammate to work with the hook's stderr as feedback.
- **`TaskCreated` hook** — runs when a new task is created. Exit code 2 prevents creation (useful for enforcing naming conventions or blocking duplicate tasks).

Hooks are shell scripts placed in `.claude/hooks/` and configured in `settings.json` under `"hooks"`.

---

## Competing Hypotheses (for Debugging)

When investigating a complex bug, sequential analysis causes **context bias** — the agent
anchors to its first plausible theory and stops looking. Agent teams solve this:

1. **Spawn 3-5 teammates**, each assigned a different hypothesis (e.g., database issue,
   network latency, cache invalidation, race condition, configuration error).
2. **Make them adversarial** — instruct each teammate to not only investigate their own
   theory but actively try to **disprove the others'** by sharing evidence via messages.
3. The lead **synthesizes** findings after all teammates report. The theory that survives
   adversarial challenge is far more likely to be the actual root cause.

Example spawn instruction:
> "Spawn 3 teammates to debug the payment timeout. Agent 1: investigate database slow
> queries. Agent 2: investigate Stripe webhook latency. Agent 3: investigate connection
> pool exhaustion. Each agent must share findings and challenge the other theories."

This pattern is documented in the official Agent Teams docs and consistently recommended
by the community as one of the highest-value team use cases.

---

## Cross-Model Validation

When an agent reviews its own work, it tends to confirm rather than challenge — a
phenomenon called **self-review drift**. To counter this:

- Use a **different model for the reviewer** than the implementer. For example, if the
  implementer runs on Opus, have the reviewer run on Sonnet (or vice versa).
- The reviewer's different training biases catch issues the implementer is blind to.
- This is especially important for autonomous agents running long sessions without
  human check-ins.

In practice: set `model: sonnet` on your qa-reviewer while builders use `model: opus`.

---

## Color Assignment

Claude Code supports 8 teammate colors: red, blue, green, yellow, purple, orange, pink, cyan. With 15+ available roles, color duplicates are unavoidable. The generic roles have priority colors. When composing a team, avoid pairing two roles with the same color in the same phase. If needed, tell the lead to assign a different color at spawn time: "Spawn the customer-tester teammate with color orange."

---

## Visual Monitoring with tmux

For teams with 3+ teammates, use tmux split-pane mode to monitor all sessions simultaneously. Each pane shows one teammate's live output, making it easy to spot stuck agents, loops, or errors without switching contexts. The team lead's pane should be the largest (top or left), with teammate panes tiled below or to the right.

---

*Last updated: 2026-04-05*
