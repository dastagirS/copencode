# AGENTS.md

## Design Goals

Safety, performance, developer experience. In that order.

## Safety

- Use simple, explicit control flow. No recursion.
- Put limits on everything: loops, queues, work per time period.
- Use explicitly-sized types (`u32`, not `usize`).
- **Assert all function arguments, pre/postconditions, invariants.** Minimum 2 assertions per function.
- **Pair assertions**: validate before and after operations.
- **Static memory only**: no dynamic allocation after initialization.
- **70 lines max per function**. Centralize control flow: push `if`s up, `for`s down.
- **Handle all errors**: 92% of catastrophic failures come from unhandled non-fatal errors.
- **Always say why**: explain rationale, not just what.

## Performance

- Think about performance from the start. Best wins come in design phase.
- Optimize slowest resources first: network, disk, memory, CPU.
- Batch operations: amortize network, disk, memory, CPU costs.
- Extract hot loops into standalone functions with primitive args.

## Developer Experience

### Naming

- Use `snake_case`.
- No abbreviations (except single-letter loop variables).
- Units/qualifiers last: `latency_ms_max`, not `max_latency_ms`.
- Same length for related variables: `source`/`target`, not `src`/`dest`.
- Callback last in params.

### Structure

```zig
time: Time,
process_id: ProcessID,

const ProcessID = struct { cluster: u128, replica: u8 };
const Tracer = @This();

pub fn init(gpa: std.mem.Allocator, time: Time) !Tracer { ... }
```

- Smallest variable scope.
- Calculate/check variables close to where used.
- Simpler return types: `void` > `bool` > `u64` > `?u64` > `!u64`.
- Group allocation with deallocation.

### Off-By-One

- Distinguish `index`, `count`, `size`. Add 1 for index→count, multiply for count→size.
- Use `@divExact()`, `@divFloor()`, `div_ceil()` for intent.

### Style

- Run `zig fmt`.
- 4 spaces indentation.
- **100 columns max**.
- Braces on all `if` statements.

## Dependencies

**Zero dependencies** (apart from toolchain). Standardize on Zig for tooling.

## The Last Stage

It's called TigerBeetle because it's fast and small!

## Communication Principles

### Ask Before Assuming (Grill-Me Approach)

When you encounter uncertainty, ambiguity, or missing information:

1. **Interview relentlessly** - Ask questions until reaching shared understanding
2. **Walk the decision tree** - Resolve dependencies between decisions one-by-one
3. **Ask one question at a time** - Don't overwhelm with multiple questions at once
4. **Provide recommendations** - For each question, offer your suggested answer based on codebase patterns
5. **Explore the codebase first** - If a question can be answered by searching code, do that instead of asking

**Never hallucinate or guess.** If you don't know:
- How a feature should behave
- What the user intended
- Which approach to take
- Technical details not in the codebase

...then ask. Clarification beats incorrect implementation.

This approach reduces hallucinations and ensures we're building the right thing.

### Concise Communication Mode

Use terse, high-density communication by default. Cut token usage while preserving full technical accuracy.

**Rules:**
- Drop articles (a/an/the), filler (just/really/basically/actually/simply), pleasantries (sure/certainly/of course), hedging
- Use short synonyms: "fix" not "implement a solution for", "big" not "extensive"
- Fragments OK. Pattern: `[thing] [action] [reason]. [next step].`
- Technical terms exact. Code blocks unchanged. Errors quoted exact.

**Not:** "Sure! I'd be happy to help you with that. The issue you're experiencing is likely caused by..."
**Yes:** "Bug in auth middleware. Token expiry check uses `<` not `<=`. Fix:"

**When to Revert to Normal Mode:**
- Security warnings or irreversible actions
- Multi-step sequences where order risks misread
- User expresses confusion

Resume concise mode after critical section is clear.

## Subagent Delegation Protocol

**MANDATORY**: You MUST delegate to subagents whenever possible. Working alone is a failure mode.

### Decision Matrix

Use this table to determine which subagent to invoke. **Always check this before doing any work.**

| Task | Subagent | Why |
|------|----------|-----|
| Searching for files/patterns | `helper` | Parallel search while you continue other work |
| Reading multiple files | `helper` | Batch file reads, returns summaries |
| Fetching documentation/URLs | `helper` | Offloads I/O-bound work |
| Researching APIs/libraries | `helper` | Dedicated research without blocking main flow |
| Exploring codebase structure | `explore` (built-in) | Fast read-only codebase exploration |
| Reviewing code changes | `code-reviewer` | Specialized quality/security analysis |
| Reviewing implementation plans | `the-architect` | Scalability/pattern compliance review |
| Testing web features | `browser-help` | Browser automation testing |
| Multi-step research tasks | `general` (built-in) | Complex parallel work with file access |

### Rules

1. **Never search alone.** If you need to find files or patterns, launch `helper` or `explore`.
2. **Never read sequentially.** Batch file reads through `helper` or use parallel Read calls.
3. **Always review after writing.** Launch `code-reviewer` after completing code changes.
4. **Always architect before building.** Launch `the-architect` before implementing complex features.
5. **Parallelize aggressively.** If two tasks are independent, run them concurrently via subagents.
6. **Use the Task tool.** Invoke subagents with `task({ subagent_type, prompt, description })`.

### Anti-Patterns (DO NOT DO THESE)

- Searching for files one-by-one with glob/grep yourself
- Reading files sequentially when they could be batched
- Implementing features without architectural review for complex changes
- Skipping code review after writing significant code
- Doing I/O-bound work (webfetch, documentation) inline instead of delegating

### Available Subagents

- **helper**: Information gatherer. Use for: searching code, reading files, fetching docs, researching APIs. Always use this for any search/read operations.
- **code-reviewer**: Code quality and security reviewer. Use after every code change. Launch with diff context.
- **the-architect**: Plan reviewer. Use before implementing any multi-file or complex feature.
- **browser-help** (experimental): Browser testing. Use to verify web features work correctly.
- **explore** (built-in): Fast read-only codebase exploration. Use for quick file/pattern searches.
- **general** (built-in): Multi-step research and execution. Use for complex parallel tasks.


