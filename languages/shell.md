# Shell/Bash Guidelines

## Core Principles

- Always start scripts with `#!/usr/bin/env bash`
- Always set `set -euo pipefail` on the second line
- Scripts must not exceed 200 lines; extract reusable functions or split into multiple scripts
- No magic numbers; define named constants at the top of the script
- Every script must include a usage/help function

## Naming

- Environment variables and constants: `UPPER_SNAKE_CASE` (`MAX_RETRY_COUNT`, `DB_HOST`)
- Local variables and functions: `lower_snake_case` (`user_name`, `process_file`)
- File names: `lower-kebab-case` or `lower_snake_case` (`deploy-app.sh`, `run_tests.sh`)
- Booleans: use `true`/`false` strings; never `0`/`1` for boolean logic

## Script Header

- Every script begins with shebang, strict mode, and a brief description

```bash
#!/usr/bin/env bash
set -euo pipefail

# Description: Deploy the application to the target environment.
# Usage: ./deploy.sh <environment> [--dry-run]

readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly DEFAULT_TIMEOUT=30
```

## Variable Quoting

- Quote all variable expansions: `"${var}"` not `$var`
- Quote command substitutions: `"$(command)"`
- Use `"${array[@]}"` to expand arrays safely
- Only skip quotes in arithmetic contexts `$(( ))` or inside `[[ ]]` on the right side of `=~`

```bash
# Bad
if [ $filename = *.txt ]; then
    cp $filename $dest_dir
fi

# Good
if [[ "${filename}" == *.txt ]]; then
    cp "${filename}" "${dest_dir}/"
fi
```

## Conditionals and Tests

- Use `[[ ]]` over `[ ]` — it handles empty strings, pattern matching, and logical operators safely
- Use `$(command)` over backticks `` `command` ``
- Use `(( ))` for arithmetic comparisons

```bash
# Bad
if [ `echo $count` -gt 5 ]; then

# Good
if (( count > 5 )); then
```

## Functions

- Declare functions with `function_name() { }` (no `function` keyword)
- Declare all variables inside functions as `local`
- Return exit codes (0 for success, non-zero for failure); do not return strings via `return`
- Use `printf` over `echo` for portable output; use `echo` only for simple messages

```bash
# Bad
function processFile {
    result="processed"
    return $result  # wrong: return only takes integers
}

# Good
process_file() {
    local input_file="$1"
    local output_file="$2"

    if [[ ! -f "${input_file}" ]]; then
        printf 'Error: File not found: %s\n' "${input_file}" >&2
        return 1
    fi

    # process the file ...
    return 0
}
```

## Error Handling

- Use `trap` for cleanup on exit, error, and interrupt signals
- Always check return codes for critical commands or use `set -e`
- Redirect error messages to stderr: `>&2`
- Never use `cd` without `|| exit` or `|| return`

```bash
# Bad
cd /some/directory
rm -rf build/

# Good
cleanup() {
    rm -rf "${tmp_dir:-}"
}
trap cleanup EXIT

cd "/some/directory" || { printf 'Failed to cd\n' >&2; exit 1; }
rm -rf build/
```

## Input Validation

- Validate the number of arguments at the start
- Provide clear error messages with usage hints
- Use `readonly` for constants and variables that must not change

```bash
main() {
    if (( $# < 1 )); then
        printf 'Usage: %s <environment> [--dry-run]\n' "$(basename "$0")" >&2
        exit 1
    fi

    local environment="$1"
    local dry_run=false

    if [[ "${2:-}" == "--dry-run" ]]; then
        dry_run=true
    fi

    deploy "${environment}" "${dry_run}"
}

main "$@"
```

## Command Substitution and Pipes

- Use `$(command)` not `` `command` `` — it nests cleanly and is easier to read
- Pipe into `while read` with care; use `< <(command)` (process substitution) to avoid subshell variable scope issues
- Use `xargs` or `find -exec` over `for f in $(ls)` loops

```bash
# Bad: word splitting, misses files with spaces
for f in $(find . -name "*.log"); do
    rm "$f"
done

# Good
find . -name "*.log" -exec rm {} +

# Good: reading lines safely
while IFS= read -r line; do
    process_line "${line}"
done < <(get_lines)
```

## Prohibited Patterns

- No `eval` — it is a code injection vector
- No unquoted variables — they cause word splitting and globbing bugs
- No `cd` without `|| exit` or `|| return`
- No backticks for command substitution; use `$()`
- No `#!/bin/bash` — use `#!/usr/bin/env bash` for portability
- No `set -e` disabled mid-script without re-enabling
- No parsing `ls` output; use globs or `find`
- No `cat file | grep` (useless use of cat); use `grep pattern file`

```bash
# Bad
eval "$user_input"
cat file.txt | grep "pattern"
files=`ls *.txt`

# Good
grep "pattern" file.txt
files=(*.txt)
```

## Logging and Output

- Use a consistent log function with timestamps and severity levels
- Send informational output to stdout, errors to stderr
- Use `set -x` only for debugging; remove before committing

```bash
log() {
    local level="$1"
    shift
    printf '[%s] [%s] %s\n' "$(date '+%Y-%m-%d %H:%M:%S')" "${level}" "$*" >&2
}

log "INFO" "Deployment started for ${environment}"
log "ERROR" "Failed to connect to ${host}"
```
