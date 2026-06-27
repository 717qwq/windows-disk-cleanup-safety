# windows-disk-cleanup-safety（中文说明）

[English](README.md) | **中文**

一个 [Claude Code](https://claude.com/claude-code) **技能(skill)**——当 Windows 磁盘空间不足时,判断**"什么能安全删除"**的安全参考。

## 这是什么

一个强模型本来就会清盘:先测量、再删、最后验证。它**不一定每次都记牢**的,是那一小撮"删了就找不回"的陷阱。这个技能只保留那个内核——两张表,不重复模型本来就会的事:

1. **给每个大文件分类** —— `缓存`(可删) / `可重建存储`(删了要重新下载) / `用户数据`(绝不自动删)。
2. **应用特有陷阱** —— 直接删会出事的目录:
   - **消息类应用**(微信 / QQ / Telegram)—— 删目录会**损坏聊天数据库**
   - **浏览器**(Edge / Chrome)—— 删整个 `User Data` 会丢**登录态和扩展**
   - **包管理器**(npm / pip / conda / yarn / NuGet)—— 用官方缓存命令,别 `rm`
   - **系统文件**(`Windows.old`、`hiberfil.sys`)—— 用 `cleanmgr` / `powercfg`,别手动删

它是防止**不可逆的灾难性删除**的**安全护栏**,不是提效工具——删错的代价(聊天记录、游戏存档、浏览器登录态永久丢失)是无法挽回的。

## 安装

克隆到你的 agent 技能目录:

```bash
# Claude Code (Windows)
git clone https://github.com/717qwq/windows-disk-cleanup-safety "$USERPROFILE/.claude/skills/windows-disk-cleanup-safety"
```

下次涉及磁盘空间时,agent 会通过 `SKILL.md` 里的 `description` 自动发现并加载它。

## 它是怎么造出来的

用 [Superpowers](https://github.com/obra/superpowers) 的 `writing-skills` 方法创建(给文档做 TDD)。基线测试显示:强模型本来就能把**流程**处理得很好,所以这个技能被**刻意精简**,只保留它不一定记牢的非显而易见知识——**分级模型 + 陷阱表**。

## 许可

MIT —— 见 [LICENSE](LICENSE)。
