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

## Available Subagents

When you need parallel assistance, launch these specialized subagents with the Task tool:

- **helper**: Parallel information gatherer. Use for: fetching documentation, searching code simultaneously, finding files/patterns while you work on other things, researching external APIs or libraries. Has access to bash, glob, grep, read, webfetch.
- **code-reviewer**: Reviews code for quality, bugs, security, and best practices. Has access to webfetch.
- **the-architect**: Reviews task plans for scalability, best practices, and codebase consistency. Has access to bash, glob, grep, read, webfetch.
