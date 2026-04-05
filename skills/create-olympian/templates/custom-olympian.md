# {NAME}

<!-- NAME: Display name for this olympian. Used in spawn prompts and team rosters.
     Examples: "Backend Builder", "Security Reviewer", "Schema Architect" -->

**When to use:** {DESCRIPTION}
<!-- DESCRIPTION: One-sentence explanation of when to spawn this olympian.
     Example: "When implementing new API endpoints following the vertical slice pattern." -->

**Model:** {MODEL}
<!-- MODEL: The Claude model to use for this olympian.
     Options: "opus" for complex reasoning, "sonnet" for balanced speed/quality, "haiku" for fast lightweight tasks.
     Consider: reviewers can use cheaper models; architects benefit from opus. -->

**Tools:** {TOOLS}
<!-- TOOLS: Comma-separated list of tools this olympian is allowed to use.
     Examples:
       Builder:  "Read, Edit, Write, Bash, Grep, Glob"
       Reviewer: "Read, Grep, Glob" (read-only to prevent accidental modifications)
       Architect: "Read, Grep, Glob, Write" (write only for contract/schema files)
     Restricting tools enforces role boundaries and prevents teammates from overstepping. -->

**Isolation:** {ISOLATION}
<!-- ISOLATION: How this olympian is isolated from other teammates.
     Options:
       "none" - works in the same worktree (suitable when file ownership is non-overlapping)
       "worktree" - gets its own git worktree (use for write-heavy parallel work)
     Rule of thumb: if two olympians might touch nearby files, use worktree isolation. -->

**Phase:** {PHASE_HINT}
<!-- PHASE_HINT: When in the team lifecycle this olympian should be spawned.
     Options:
       "foundation" - spawned first, defines contracts and schemas
       "implementation" - spawned after contracts are finalized
       "review" - spawned last, after implementation is complete
       "parallel" - independent of feature phases, can run alongside others
       "any" - flexible, parallel with or after implementation
       "testing" - E2E/integration testing, after feature implementation
       "planning" - autonomous planning, before implementation begins
     This guides the team lead on spawn ordering (contract-first pattern). -->

**Color:** {COLOR}
<!-- COLOR: Terminal color for this olympian's output in the team lead view.
     Options: "red", "blue", "green", "yellow", "purple", "orange", "pink", "cyan"
     Assign distinct colors to each olympian for easy visual identification. -->

**Requires:** {REQUIRES}
<!-- REQUIRES: External dependencies this olympian needs to function.
     List CLIs, plugins, MCP servers, or services that must be available.
     Examples:
       "Stripe CLI, stripe plugin"
       "agent-browser CLI, gemini plugin, codex plugin"
       "Docker Compose with GreenMail"
       "none" (if no external dependencies)
     This helps users understand what to install before spawning this olympian. -->

## Capabilities

{CAPABILITIES}
<!-- CAPABILITIES: Bulleted list of what this olympian can do.
     Be specific -- these guide the team lead on task assignment.
     Example:
       - Create and modify database entity classes
       - Write EF Core migrations
       - Define API request/response DTOs
       - Implement FluentValidation validators -->

## System Prompt

{SYSTEM_PROMPT}
<!-- SYSTEM_PROMPT: The system-level instructions injected into this olympian's context.
     This is the most important field. It should include:
       1. Role definition ("You are a backend implementation agent...")
       2. Project patterns the olympian must follow (e.g., IQueryHandler, PascalCase DTOs)
       3. Hard constraints ("NEVER modify files outside your owned directories")
       4. Communication protocol ("Message the lead when your contract is ready")
       5. Done criteria ("You are done when all endpoints pass validation")

     Keep it focused. Teammates do NOT inherit the lead's conversation history,
     so everything they need must be in this prompt or provided via messages. -->

## Spawn Example

"{SPAWN_EXAMPLE}"
<!-- SPAWN_EXAMPLE: A concrete example of how the team lead would spawn this olympian.
     Should demonstrate realistic task assignment with proper context.
     Example:
       "Spawn a Backend Builder to implement the CreateBooking feature.
        Owns: src/Features/Bookings/Commands/CreateBooking/
        Contract: BookingRequest { roomId: int, startTime: datetime, duration: int }
        Must follow: IQueryHandler pattern, FluentValidation, PascalCase DTOs.
        Done when: all 7 files created, handler compiles, validator covers all fields." -->
