---
name: reclaiming-windows-disk-space
description: Use when a Windows machine is low on disk space (C: drive full, "low disk space" warnings, near-zero free space) and you need to safely free space without deleting user data, game saves, or chat history.
---

# Reclaiming Windows Disk Space

## Overview

Freeing disk space safely is one judgment repeated many times: **is this byte a cache or is it data?** Caches rebuild themselves; data does not. The danger is never "deleting too little" — it is deleting something the user can never get back. So: **measure before you touch anything, classify every large item, and never delete a thing you cannot prove is a cache.**

There is no fixed list of what to delete — every machine hides space in different places. This skill gives you the *method* and the *traps*, not a script to run blindly.

## The Core Distinction (the only judgment that matters)

| Tier | What it is | Rule |
|------|-----------|------|
| **Cache** | Rebuilds itself invisibly, zero cost (HTTP caches, shader caches, game asset caches, `%TEMP%`) | Safe to delete |
| **Reclaimable store** | Rebuilds, but at a cost — re-download, slower first build, offline pain (package stores: `.nuget\packages`, conda/npm/pip/yarn stores, Docker images) | Delete only with the user's explicit awareness of the cost |
| **User data** | Never regenerates (chat history, game saves, documents, browser logins/extensions, app accounts) | Never auto-delete. Point it out; let the user decide |

A big folder is not deletable because it is big. It is deletable because you identified *what it is*.

## Method

1. **Measure first (read-only).** Find the real hogs before forming any plan. Do not trust assumptions about where space went.
   ```powershell
   # Top space consumers under the user profile (read-only)
   Get-ChildItem $env:USERPROFILE -Directory -Force -EA SilentlyContinue | ForEach-Object {
     [PSCustomObject]@{ GB = [math]::Round((Get-ChildItem $_.FullName -Recurse -File -Force -EA SilentlyContinue |
       Measure-Object Length -Sum).Sum / 1GB, 2); Path = $_.FullName }
   } | Sort-Object GB -Descending | Select-Object -First 15
   ```
   Then drill into the biggest directories until you can name what each large block actually is.
2. **Classify** every large item into the three tiers above.
3. **Confirm before deleting.** Present the plan grouped by tier and the space each frees. Delete only what the user approves. Never auto-delete Tier 3.
4. **Delete the contents, keep the folder.** Caches rebuild into the existing directory; removing the folder itself can confuse the app.
5. **Close the app first** if its cache is locked, or accept that in-use files are skipped (`-ErrorAction SilentlyContinue`).
6. **Re-measure** to verify and report the actual space freed.

## App-Specific Traps (the non-obvious part)

Most "I deleted a folder and broke something" disasters are here. Never `rm` an app's whole data folder — use its own cleanup path.

| App / area | Trap if you delete the folder | Safe approach |
|------------|-------------------------------|---------------|
| **Messengers** (WeChat, QQ, Telegram, etc.) | Corrupts the message database — chat history lost | Use the app's **in-app storage management** to clear media/files |
| **Browsers** (Edge, Chrome) | Wiping `User Data` loses logins, extensions, site data | Delete **only** `Cache`, `Code Cache`, `GPUCache`, `DawnCache`. Never `IndexedDB`, `Extensions`, `Local Storage`, `Service Worker` |
| **Collaboration apps** (Feishu/Lark, Slack, Teams) | Forces re-login and re-download of files | Clear cache from the app's **settings**, not the filesystem |
| **Package managers** (npm, pip, conda, yarn, NuGet) | Breaks the "no re-download" guarantee or corrupts the store | Use official commands: `npm cache clean --force`, `pip cache purge`, `conda clean -a -y`, `yarn cache clean`, `dotnet nuget locals all --clear` |
| **Game asset caches** (e.g. VRChat `Cache-*`) | Usually safe | Delete the `Cache-*` folders only — never `Saved`, saves, or config |

## Where Space Hides on Windows (usual suspects checklist)

- `%TEMP%` and `C:\Windows\Temp`
- Browser caches (per-profile `Cache` subfolders)
- Package-manager stores/caches: `.nuget\packages`, `.conda\pkgs`, npm/pip/yarn caches
- Game asset caches and saves under `%LOCALAPPDATA%` / `%APPDATA%\..\LocalLow`
- Shader caches: NVIDIA `DXCache`, `D3DSCache`
- Recycle Bin — **confirm before emptying**: it holds the user's own deleted-but-recoverable files, not cache
- **System-level (needs admin):** `Windows.old` (after an upgrade), `WinSxS`, `SoftwareDistribution`, Delivery Optimization, restore points
- **Do not touch manually:** `hiberfil.sys`, `pagefile.sys` (manage via system settings only)

## Permission Blind Spot

Without admin rights you cannot accurately measure or clean system directories (`C:\Windows\Temp`, update residue, restore points). When you hit this wall, do not guess — **escalate to the built-in tools**: tell the user to run **Disk Cleanup (`cleanmgr`) → "Clean up system files"** or enable **Storage Sense**, which delete system junk under a safety guarantee you cannot provide from a normal shell.

## Common Mistakes

- Deleting a folder because it is large, without identifying what it is.
- Deleting an app's whole data folder instead of using its built-in cleanup.
- Treating a package store as a free cache and ignoring the re-download / first-build / offline cost.
- Removing the cache *folder* instead of its *contents*.
- Forgetting to close the app, then reporting space that was never actually freed.
- Emptying the Recycle Bin without confirming — it holds the user's own deleted files, not cache.
- Skipping the final re-measure, so you never verify the result.
