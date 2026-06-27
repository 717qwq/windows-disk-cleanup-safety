---
name: windows-disk-cleanup-safety
description: Use when freeing disk space on a Windows machine (C: drive full, "low disk space") and deciding what is safe to delete — telling cache apart from irreplaceable user data, and avoiding app folders that break when deleted directly.
---

# Windows Disk Cleanup Safety

## Overview

Cache rebuilds itself; data does not. When freeing disk space, the only real risk is irreversibly deleting something the user can never get back. **A folder is not deletable because it is big — it is deletable only once you have identified what it is.** Classify every large item before touching it, and never delete an app's whole data folder when the app has its own cleanup.

## Classify every large item

| Tier | What it is | Rule |
|------|-----------|------|
| **Cache** | Rebuilds itself invisibly, zero cost (HTTP / shader / game-asset caches, `%TEMP%`) | Safe to delete the **contents** (keep the folder) |
| **Reclaimable store** | Rebuilds, but at a cost — re-download, slower first build, offline pain (package stores: `.nuget\packages`, conda / npm / pip / yarn stores, Docker images) | Delete only with the user aware of the cost |
| **User data** | Never regenerates (chat history, game saves, documents, browser logins/extensions, deleted files still in the Recycle Bin) | Never auto-delete — point it out, let the user decide |

A big folder is not deletable because it is big. It is deletable because you identified *what it is*.

## App-specific traps

Most "I deleted a folder and broke something" disasters are here. Never `rm` an app's whole data folder — use its own cleanup path.

| App / area | Trap if you delete the folder | Safe approach |
|------------|-------------------------------|---------------|
| **Messengers** (WeChat, QQ, Telegram) | Corrupts the message database — chat history lost | Use the app's **in-app storage management** |
| **Browsers** (Edge, Chrome) | Wiping `User Data` loses logins, extensions, site data | Delete **only** `Cache`, `Code Cache`, `GPUCache`, `DawnCache`; never `IndexedDB`, `Extensions`, `Local Storage`, `Service Worker` |
| **Collaboration apps** (Feishu/Lark, Slack, Teams) | Forces re-login and re-download of files | Clear cache from the app's **settings** |
| **Package managers** (npm, pip, conda, yarn, NuGet) | Breaks the "no re-download" guarantee or corrupts the store | Official commands: `npm cache clean --force`, `pip cache purge`, `conda clean -a -y`, `yarn cache clean`, `dotnet nuget locals all --clear` |
| **Recycle Bin** | These are the user's own deleted-but-recoverable files, not cache | **Confirm before emptying** (`Clear-RecycleBin`) |
| **System files** (`Windows.old`, `hiberfil.sys`, `Windows\Temp`, update cache) | Manual delete fails or breaks the system; needs admin | Use built-in tools: `cleanmgr` → "Clean up system files", or `powercfg /h off` to remove the hibernation file |
