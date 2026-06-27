# windows-disk-cleanup-safety

A compact [Claude Code](https://claude.com/claude-code) **skill** — a safety reference for deciding *what is safe to delete* when a Windows machine runs low on disk space.

## What it is

A strong model already knows how to clean a disk: measure first, delete, verify. What it doesn't reliably know is the small set of **irreversible traps**. This skill is just that kernel — two tables, nothing a capable model already does:

1. **Classify every large item** — `cache` (safe) vs. `reclaimable store` (costs a re-download) vs. `user data` (never auto-delete).
2. **App-specific traps** — the folders that *break* when deleted directly:
   - **Messengers** (WeChat/QQ/Telegram) — deleting the folder corrupts the message database
   - **Browsers** (Edge/Chrome) — wiping `User Data` loses logins and extensions
   - **Package managers** (npm/pip/conda/yarn/NuGet) — use the official cache commands, not `rm`
   - **System files** (`Windows.old`, `hiberfil.sys`) — needs `cleanmgr` / `powercfg`, never a manual delete

It is a **guardrail against catastrophic, irreversible deletion**, not a productivity booster — the cost of being wrong (lost chat history, game saves, browser logins) is permanent.

## Install

Clone into your agent's skills directory:

```bash
# Claude Code (Windows)
git clone https://github.com/717qwq/windows-disk-cleanup-safety "$USERPROFILE/.claude/skills/windows-disk-cleanup-safety"
```

The agent discovers it automatically via the `description` in `SKILL.md` the next time disk space comes up.

## How it was built

Created with the [Superpowers](https://github.com/obra/superpowers) `writing-skills` method (TDD for documentation). The baseline test showed a capable model already handles the *process* well, so the skill was deliberately slimmed down to only the non-obvious knowledge it doesn't reliably carry: the tier model and the trap table.

## License

MIT — see [LICENSE](LICENSE).
