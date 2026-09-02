---
name: bash-first
version: 1.0.0
display_name: Bash 优先
display_name_en: Bash First
description: Write every shell command in POSIX/bash syntax so it runs on any shell — critical when the user's interactive shell is fish or zsh, which reject bash-isms (heredocs, $( ), for..done loops, process substitution, ${} expansion, $?/$@/$!). Use whenever generating or executing shell commands, writing one-liners or multi-line scripts, or when the execution shell is not guaranteed to be bash.
description_zh: 所有 shell 命令按 POSIX/bash 语法编写，保证在 fish/zsh 等宿主 shell 下不炸 —— 含 fish 陷阱清单与翻译速查表
description_en: Write every shell command in POSIX/bash syntax so it runs on any shell
---

# Bash-First

Shell commands are written **bash-first**: POSIX/bash syntax that runs correctly no matter which shell actually executes it (bash, zsh, sh). Coding agents generate bash-shaped commands from training; if the executing shell is fish, those commands fail with confusing errors — or worse, **silently misbehave** (see Silent Traps).

## Hard rules

1. **Write bash syntax, always.** Never emit fish-only syntax (`set -x`, `(cmd)`, `for i in ...; end`, `$status`, `$argv`) unless you explicitly intend to run `fish -c`.
2. **Wrap multi-line / complex logic in bash** — never assume the host shell is bash:
   ```
   bash -lc '...multi-line script...'
   ```
   or write `script.sh` and run `bash script.sh`.
3. **Heredocs only inside bash.** A bare `cat <<'EOF' ... EOF` fails under fish. Either run it inside `bash -lc`:
   ```bash
   bash -lc "cat <<'EOF' > file.txt
   line one
   line two
   EOF"
   ```
   or avoid it: `printf '%s\n' 'line one' 'line two' > file.txt`.
4. **Process substitution, parameter expansion, arrays — bash only.** `<(...)`, `${var:-default}`, `${var//a/b}`, `"${arr[@]}"`, `$!` have no fish equivalent; fish errors on `<(...)` and treats `${...}` as a literal string (silent). Use `bash -lc`, or rewrite with portable forms (`tmp=$(mktemp)` + redirect; `string replace` is fish-only — use `sed`).
5. **Check exit status with `$?` only inside bash.** Under fish it's `$status`; a bare `$?` becomes an empty string. When in doubt, run inside `bash -lc` and use `$?`.
6. **Prefer portable one-liners** that work under both bash and fish:
   - command substitution: `$(cmd)` works in bash; fish needs `(cmd)` — so inside a shared command use `bash -lc "$(cmd)"`.
   - env prefix: `env VAR=x cmd` (fish rejects bare `VAR=x cmd`).
   - chaining: `cd x && cmd` (fish 3.0+ supports `&&`; avoid bare `;` chains).
   - tests: `test -f file` (portable) over `[[ -f file ]]` (bash-only).
7. **Never rely on `set -e` / `set -u` / `pipefail`.** In fish, `set -e` is *variable deletion*, not an option — the script silently continues after failures. Enforce strictness only inside `bash -lc` (`set -euo pipefail`).
8. **When you must run fish** (user aliases/abbreviations): `fish -c 'simple command'` for trivial cases only, and say so explicitly.

## Fish → bash translation cheat sheet

| Fish | Bash (emit this) |
| --- | --- |
| `set -x VAR value` | `export VAR=value` or `env VAR=value cmd` |
| `(cmd)` | `$(cmd)` |
| `$status` | `$?` (inside bash) |
| `$argv` / `$argv[1]` | `"$@"` / `"$1"` |
| `$last_pid` | `$!` (inside bash) |
| `for i in a b; ...; end` | `for i in a b; do ...; done` |
| `if test -f x; ...; end` | `if [ -f x ]; then ...; fi` |
| `switch $x; case a; ...; end` | `case "$x" in a) ... ;; esac` |
| `2> file` | `2> file` (same in both) |
| `string replace a b -- $s` | `sed 's/a/b/'` / `${s//a/b}` (bash only) |
| `string match` | `grep -E` / `sed` |
| `pwd` (physical path) | `pwd -P` in bash to match, else `$PWD` is logical |
| `command -v x` | `command -v x` (same) |
| `read var` | `read -r var` (inside bash) |
| `source file.fish` | `source file.sh` / `. file.sh` |
| `type -a x` | `type -a x` (same) |

## Silent traps (fish accepts the command but does something else)

These are the dangerous ones — no error, wrong behavior:

- `set -e` / `set -u`: fish treats `set` as variable assignment → strict-mode options silently never apply. Run inside `bash -lc`.
- `${var:-default}` / `${var//a/b}`: fish treats `${...}` as a literal string (or errors on some forms) — no expansion, no warning.
- `$?`: empty string in fish (always "success" to naive checks).
- `"${arr[@]}"`: fish array expansion differs; quoting semantics can split wrongly.
- `VAR=x cmd`: in fish, `VAR=x` is parsed as a bare command name → "Unknown command" (at least it errors). Use `env VAR=x cmd`.

## Verification

- Syntax-check before running: `bash -n -c '<command>'` (or `bash -n script.sh`).
- On failure, check the shell first: `echo $0` / `echo $SHELL`. If it is fish and the command used bash syntax, re-emit with `bash -lc` — don't trial-and-error.
- A command that works under `bash -c '...'` is the safe baseline for every agent tool (pi, dsh, Claude Code, Codex all execute through bash/zsh already).
