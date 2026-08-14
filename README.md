# public-skills

Open-source agent skills, portable across coding agents (pi, dsh, Claude Code, CodeBuddy, ...).

## Skills

| skill | description |
| --- | --- |
| [bash-first](skills/bash-first/SKILL.md) | Write shell commands in POSIX/bash syntax so they run on any shell — heredocs, `$( )`, loops, quoting that survive fish/zsh hosts |
| [no-slop](skills/no-slop/SKILL.md) | Score, audit, and fix AI-generated slop in code/diffs/PRs/docs in one pass |

## Install

Copy a `skills/<name>` directory into your agent's skills folder, or reference the SKILL.md directly. Each skill is self-contained (single SKILL.md unless noted).

## License

MIT
