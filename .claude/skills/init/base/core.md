# Core Development Principles

## Attitude Toward Legacy Code

This is the most important rule: **Do not mimic the style and patterns of existing code in the project.** Always follow this specification.

- When modifying old code, refactor the parts you touch according to this specification. Do not perpetuate bad habits for the sake of "consistency"
- If old code has obvious design problems (God Class, deep nesting, hardcoding, excessive coupling), fix them while making changes
- Do not be afraid to change the structure of old code, as long as behavior remains unchanged
- If the refactoring scope is too large (cascading changes across more than 3 files), explain the plan before proceeding

## Hard Requirements for Code Quality

- A single function must not exceed 30 lines (excluding blank lines and comments); split if it does
- A single file must not exceed 300 lines; split by responsibility if it does
- Nesting depth must not exceed 3 levels (if/for/callback); reduce with early returns, extracted functions, etc.
- Function parameters must not exceed 4; use an object parameter if more are needed
- No commented-out code allowed; delete unused code instead of commenting it out
- No magic numbers or magic strings; extract them into named constants

## Naming

- Names must be semantic; the purpose should be clear from the name alone
- No meaningless names: `data1`, `temp`, `info`, `obj`, `result`, `item` (except loop variables)
- Boolean values use `is`/`has`/`can`/`should` prefixes: `isLoading`, `hasPermission`
- Function names start with a verb: `fetchUser`, `validateInput`, `calculateTotal`
- Constants in ALL_CAPS_SNAKE_CASE: `MAX_RETRY_COUNT`, `API_BASE_URL`
- Event handler functions use `handle` prefix: `handleClick`, `handleSubmit`

## Architecture Principles

- **Single Responsibility**: One function does one thing, one file owns one domain
- **Separation of Concerns**: UI contains no business logic, business logic contains no UI code, data access is a separate layer
- **Unidirectional Dependencies**: Upper layers depend on lower layers, never the reverse. UI -> Business Logic -> Data Layer
- **Program to Interfaces**: Modules communicate through interfaces/protocols, not concrete implementations
- **Composition Over Inheritance**: Use composition unless there is a clear is-a relationship

## Error Handling

- Perform defensive validation only at system boundaries (user input, external API responses, file I/O)
- Internal function calls trust parameter types; no redundant validation
- Error messages should be human-friendly and include context (which operation failed, what values were passed)
- Async operations must have error handling; no bare Promises or unhandled async calls
- Do not wrap the entire function body in try-catch; only wrap the specific operations that may fail

## Avoid Over-Engineering

- Solve only the current problem; do not add abstractions for hypothetical future requirements
- Three lines of duplicated code are better than a premature abstraction
- Do not create utility functions for logic that is used only once
- Do not add unnecessary intermediate layers, wrappers, or adapters
- Add configuration and options only when flexibility is genuinely needed

## Output Requirements

- Always respond in English
- Get straight to the point; no pleasantries or preamble
- Only output information directly relevant to the current task; do not repeat what the user has already said
