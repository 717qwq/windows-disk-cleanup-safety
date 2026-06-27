# reclaiming-windows-disk-space

A [Claude Code](https://claude.com/claude-code) **skill** for safely freeing disk space on Windows — without ever deleting the user's data, game saves, or chat history.

## What it does

When a Windows machine is low on disk space, an agent's biggest risk isn't deleting too little — it's deleting something the user can never get back. This skill encodes the one judgment that matters (**cache vs. reclaimable store vs. user data**) plus a table of app-specific traps that cause "I deleted a folder and broke something" disasters:

- **Messengers** (WeChat/QQ/Telegram) — deleting the folder corrupts the message database
- **Browsers** (Edge/Chrome) — wiping `User Data` loses logins and extensions
- **Package managers** (npm/pip/conda/yarn/NuGet) — `rm` breaks the store; use the official cache commands
- **System-level** (`Windows.old`, `hiberfil.sys`, …) — needs admin / `cleanmgr` / `powercfg`, never a manual delete

It teaches the *method* (measure → classify → confirm → verify) and the *traps*, not a script to run blindly — because every machine hides space in a different place.

## Install

Clone into your agent's skills directory:

```bash
# Claude Code (Windows)
git clone https://github.com/717qwq/reclaiming-windows-disk-space "$USERPROFILE/.claude/skills/reclaiming-windows-disk-space"
```

The agent discovers it automatically via the `description` in `SKILL.md` the next time disk space comes up.

## How it was built

This skill was created with the [Superpowers](https://github.com/obra/superpowers) `writing-skills` method — TDD applied to documentation:

1. **RED** — ran the cleanup task on a real machine *without* the skill, recorded the baseline.
2. **GREEN** — wrote the minimal skill capturing the non-obvious judgments the baseline missed or got inconsistently.
3. **REFACTOR** — pressure-tested it (impatient "just delete the biggest folders" + system-level/no-admin scenarios) and closed the gaps it surfaced.

## License

MIT — see [LICENSE](LICENSE).
