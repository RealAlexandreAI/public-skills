---
name: bash-first
description: Write every shell command in POSIX/bash syntax so it runs on any shell — critical when the user's interactive shell is fish or zsh, which reject bash-isms (heredocs, $( ), for..done loops, VAR=x cmd, [[ ]] tests, single-quote escapes). Use whenever generating or executing shell commands, writing one-liners or multi-line scripts, or when the execution shell is not guaranteed to be bash.
---

# Bash-First

Shell commands are written **bash-first**: POSIX/bash syntax that runs correctly no matter which shell actually executes it (bash, zsh, sh). Most coding agents generate bash-shaped commands from training; if the executing shell is fish, those commands fail with confusing errors (heredoc unsupported, `$( )` treated as literal, loops with `done` rejected).

## Hard rules

1. **Write bash syntax, always.** Never emit fish-only syntax (`set -x`, `(cmd)`, `for i in ...; end`, `^` for stderr) unless you explicitly intend to run `fish -c`.
2. **Wrap multi-line / complex logic in bash** — never assume the host shell is bash:
   ```
   bash -lc '...multi-line script...'
   ```
   or write `script.sh` and run `bash script.sh`.
3. **Heredocs only inside bash.** A bare `cat <<'EOF' ... EOF` fails under fish. Either run the whole heredoc inside `bash -lc '...'` or avoid it:
   ```bash
   bash -lc "cat <<'EOF' > file.txt
   line one
   line two
   EOF"
   ```
   Alternative without heredoc: `printf '%s\n' 'line one' 'line two' > file.txt`.
4. **Prefer portable forms for one-liners** so they work under both bash and fish:
   - command substitution: `$(cmd)` — fish 3.4+ accepts it; safe everywhere via `bash -lc`.
   - env prefix: `env VAR=x cmd` (fish rejects bare `VAR=x cmd`).
   - `cd x && cmd` (fish supports `&&`; avoid bare `;` chains that change meaning).
   - tests: `test -f file` (portable) over `[[ -f file ]]` (bash-only) unless inside bash.
5. **When you must run fish** (user aliases/abbreviations): use `fish -c 'simple command'` for trivial cases only, and say so explicitly.

## Fish → bash translation cheat sheet

| Fish | Bash (what you should emit) |
| --- | --- |
| `set -x VAR value` | `export VAR=value` (or `env VAR=value cmd`) |
| `(cmd)` | `$(cmd)` |
| `for i in a b; ...; end` | `for i in a b; do ...; done` |
| `if test -f x; ...; end` | `if [ -f x ]; then ...; fi` |
| `cmd ^& file` | `cmd 2>&1 file` |
| `echo $argv` | `echo "$@"` |
| `string match` | `grep -E` / `sed` |
| `command -v x` | `command -v x` (same) |
| `source file.fish` | `source file.sh` / `. file.sh` |

## Verification

- Before running a suspicious command, syntax-check it: `bash -n -c '<command>'` (or `bash -n script.sh`).
- After a failure, first check the shell: `echo $0` / `echo $SHELL`. If it is fish and the command used bash syntax, re-emit with `bash -lc` — do not fall into trial-and-error rephrasing.
- A command that works under `bash -c '...'` is the safe baseline for every agent tool (pi, dsh, Claude Code, Codex all execute through bash/zsh already).
