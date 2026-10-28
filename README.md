# Claude Code Notifier

macOS desktop notifications for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Know when Claude finishes or needs your attention — without watching the terminal.

[中文文档](#中文文档)

<p align="center">
  <img src="assets/notify-complete.png" width="360" alt="Task complete notification" />
  &nbsp;&nbsp;
  <img src="assets/notify-waiting.png" width="360" alt="Waiting for input notification" />
</p>

## Background

Claude Code is a terminal-based AI coding assistant. When you assign it a task, it runs autonomously — sometimes for seconds, sometimes for minutes.

**The problem:** There's no built-in way to know when Claude finishes or needs your input. You either stare at the terminal waiting, or context-switch and miss the moment Claude is done. Both waste your time.

**The solution:** This tool uses Claude Code's [Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks) system to send macOS native desktop notifications with sound alerts. The notification subtitle shows an AI-generated session summary so you can tell *which* task finished at a glance — especially useful when running multiple sessions.

**The result:** You get a notification the instant Claude is done. Switch to another app, grab a coffee, work on something else — the notification will find you.

## Features

- **✅ Complete** — Notification + sound when Claude finishes responding
- **⏳ Waiting** — Notification + sound when Claude needs permission or user input
- **AI Session Summary** — Auto-summarizes the first message via claude-3-haiku into a short title, so you know which task at a glance
- **Local Fallback** — Falls back to local keyword extraction when API is unavailable
- **Caching** — AI is called only once per session; subsequent notifications are instant

## Notification Events

This tool listens to 2 hook events. Claude Code supports more — you can extend it to cover others:

| Hook Event | Title | Sound | When |
|---|---|---|---|
| `Stop` | ✅ Complete | Hero | Claude finished responding |
| `Notification` | ⏳ Waiting | Sosumi | Claude needs permission or user input |

### All Available Claude Code Hook Events

You can add more events by editing `notify.py` and `~/.claude/settings.json`. Here's the full list:

| Hook Event | Description | Suggested Sound |
|---|---|---|
| `Stop` | Claude finished responding | Hero (triumphant) |
| `Notification` | Needs permission / waiting for input | Sosumi (alert) |
| `SubagentStop` | Sub-agent completed | Ping (subtle) |
| `PreToolUse` | Before tool call (e.g., Bash, Write) | Tink (light tap) |
| `PostToolUse` | After tool call | Pop (quick) |
| `SessionStart` | New session started | Blow (start-up) |
| `PreCompact` | Before context compaction | Submarine (low) |
| `UserPromptSubmit` | User submitted input | — (usually silent) |

### macOS System Sounds Reference

All available sounds in `/System/Library/Sounds/`:

| Sound | Style | Best For |
|---|---|---|
| `Hero` | Triumphant, uplifting | Task completion |
| `Sosumi` | Classic Mac alert, distinctive | Attention needed |
| `Funk` | Short, punchy | Errors or warnings |
| `Basso` | Deep, serious | Critical errors |
| `Glass` | Gentle, clear | Subtle notifications |
| `Ping` | Quick, clean | Minor updates |
| `Pop` | Soft, bubbly | Tool completion |
| `Tink` | Light tap | Background events |
| `Blow` | Airy, soft | Session start |
| `Bottle` | Hollow, watery | Neutral events |
| `Frog` | Quirky croak | Fun/custom use |
| `Morse` | Rhythmic beep | Repeated events |
| `Purr` | Soft vibration | Gentle reminders |
| `Submarine` | Low sonar ping | System events |

## Requirements

- **macOS** (uses `osascript` for native notifications)
- **Python 3.6+**
- **Claude Code** installed and configured

## Installation

```bash
git clone https://github.com/antonailog/claude-code-notifier.git
cd claude-code-notifier
bash install.sh
```

Restart Claude Code after installation.

### Optional: Enable AI Session Summary

Install the Anthropic Python SDK to enable AI-powered session name summaries:

```bash
pip install anthropic
```

The notifier will use your existing Claude Code API credentials (`ANTHROPIC_API_KEY` or `ANTHROPIC_AUTH_TOKEN`) automatically. If unavailable, it falls back to local keyword extraction.

## How It Works

```
Claude Code Hook (stdin JSON)
        │
        ▼
   notify.sh (entry point)
        │
        ▼
   notify.py
        │
        ├── Read session transcript
        │       │
        │       ▼
        ├── AI Summary (claude-3-haiku)
        │   or Local keyword extraction
        │       │
        │       ▼
        ├── Cache summary (~/.cache/claude-code-notifier/)
        │
        ▼
   macOS Notification + Sound
```

1. Claude Code triggers the hook with JSON via stdin (contains `session_id`, `transcript_path`, `last_assistant_message`, etc.)
2. The script reads the first user message from the session transcript
3. Generates a short summary (AI or local fallback) and caches it per session
4. Sends a macOS native notification with the summary as subtitle

## Configuration

### Change Notification Sounds

Edit `notify.py` and modify the `sound` values in the `main()` function.

### Manual Hook Configuration

If you prefer to configure hooks manually, add this to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          { "command": "/path/to/claude-code-notifier/notify.sh", "type": "command" }
        ],
        "matcher": ".*"
      }
    ],
    "Notification": [
      {
        "hooks": [
          { "command": "/path/to/claude-code-notifier/notify.sh", "type": "command" }
        ],
        "matcher": ".*"
      }
    ]
  }
}
```

## Uninstall

```bash
cd claude-code-notifier
bash uninstall.sh
```

## Related Projects

If you need more advanced audio hooks or cross-platform support, check out these projects:

- [ctoth/claudio](https://github.com/ctoth/claudio) — Go-based audio plugin with soundpacks and contextual sounds per tool event
- [wyattjoh/claude-code-notification](https://github.com/wyattjoh/claude-code-notification) — Lightweight macOS notification hook with customizable sounds
- [shanraisshan/claude-code-voice-hooks](https://github.com/shanraisshan/claude-code-voice-hooks) — Ding/dong sounds on tool use events
- [farouqaldori/claude-island](https://github.com/farouqaldori/claude-island) — macOS menu bar session manager with notifications
- [soulee-dev/claude-code-notify-powershell](https://github.com/soulee-dev/claude-code-notify-powershell) — Windows PowerShell toast notifications

## License

MIT

---

# 中文文档

macOS 桌面通知工具，适配 [Claude Code](https://docs.anthropic.com/en/docs/claude-code)。无需盯着终端，Claude 完成或需要你操作时自动弹窗提醒。

## 背景

Claude Code 是一个基于终端的 AI 编程助手。当你给它分配任务后，它会自主运行——有时几秒，有时几分钟。

**问题：** Claude Code 没有内置的完成提示音。你要么一直盯着终端等，要么切换到其他工作后错过 Claude 完成的时机。两种方式都浪费时间。

**解决方案：** 利用 Claude Code 的 [Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks) 机制，在任务完成或需要操作时发送 macOS 原生桌面通知 + 提示音。通知副标题会显示 AI 生成的会话摘要，一眼就能看出是哪个任务——多会话并行时尤其有用。

**效果：** Claude 完成的瞬间你就会收到通知。切换到其他 App、喝杯咖啡、做别的事情——通知会找到你。

## 功能

- **✅ 已完成** — Claude 回复完毕时弹出通知 + 提示音
- **⏳ 等待操作** — Claude 需要权限确认或等待输入时弹出通知 + 提示音
- **AI 会话摘要** — 自动调用 claude-3-haiku 将首条消息总结为短标题，一眼看出是哪个任务
- **本地 fallback** — 没有 API 时自动切换到本地关键词提取，不依赖网络
- **缓存机制** — 每个 session 只调一次 AI，后续通知瞬间触发

## 通知事件

本工具监听 2 个 Hook 事件。Claude Code 还支持更多，你可以自行扩展：

| Hook 事件 | 标题 | 声音 | 触发时机 |
|---|---|---|---|
| `Stop` | ✅ 已完成 | Hero | Claude 完成回复 |
| `Notification` | ⏳ 等待操作 | Sosumi | Claude 需要权限或等待输入 |

### 所有可用的 Claude Code Hook 事件

编辑 `notify.py` 和 `~/.claude/settings.json` 即可添加更多事件：

| Hook 事件 | 说明 | 建议声音 |
|---|---|---|
| `Stop` | Claude 完成回复 | Hero（胜利感） |
| `Notification` | 需要权限 / 等待输入 | Sosumi（经典警告） |
| `SubagentStop` | 子 agent 完成任务 | Ping（轻柔） |
| `PreToolUse` | 工具调用前（如 Bash, Write） | Tink（轻叩） |
| `PostToolUse` | 工具调用后 | Pop（短促） |
| `SessionStart` | 新会话开始 | Blow（启动感） |
| `PreCompact` | 上下文压缩前 | Submarine（低沉） |
| `UserPromptSubmit` | 用户提交输入 | —（通常静音） |

### macOS 系统声音参考

`/System/Library/Sounds/` 下所有可用声音：

| 声音 | 风格 | 适合场景 |
|---|---|---|
| `Hero` | 胜利感、振奋 | 任务完成 |
| `Sosumi` | 经典 Mac 警告、辨识度高 | 需要注意 |
| `Funk` | 短促、有力 | 错误或警告 |
| `Basso` | 低沉、严肃 | 严重错误 |
| `Glass` | 轻柔、清脆 | 轻量通知 |
| `Ping` | 快速、干净 | 次要更新 |
| `Pop` | 柔和、活泼 | 工具完成 |
| `Tink` | 轻叩 | 后台事件 |
| `Blow` | 轻风感 | 会话开始 |
| `Bottle` | 空洞、水感 | 中性事件 |
| `Frog` | 蛙鸣、趣味 | 自定义用途 |
| `Morse` | 节奏感 | 重复事件 |
| `Purr` | 轻微振动 | 温和提醒 |
| `Submarine` | 低频声纳 | 系统事件 |

## 安装

```bash
git clone https://github.com/antonailog/claude-code-notifier.git
cd claude-code-notifier
bash install.sh
```

安装后重启 Claude Code 即可生效。

### 可选：启用 AI 会话摘要

安装 Anthropic Python SDK 以启用 AI 会话名称总结：

```bash
pip install anthropic
```

会自动使用你现有的 Claude Code API 凭证（`ANTHROPIC_API_KEY` 或 `ANTHROPIC_AUTH_TOKEN`）。如不可用，自动回退到本地关键词提取。

## 卸载

```bash
cd claude-code-notifier
bash uninstall.sh
```

## 相关项目

如果你需要更丰富的音频 Hook 或跨平台支持，可以参考这些项目：

- [ctoth/claudio](https://github.com/ctoth/claudio) — Go 编写的音频插件，支持音色包和按工具事件播放不同声音
- [wyattjoh/claude-code-notification](https://github.com/wyattjoh/claude-code-notification) — 轻量 macOS 通知 Hook，支持自定义系统声音
- [shanraisshan/claude-code-voice-hooks](https://github.com/shanraisshan/claude-code-voice-hooks) — 工具使用时播放 ding/dong 声音
- [farouqaldori/claude-island](https://github.com/farouqaldori/claude-island) — macOS 菜单栏会话管理器，带通知功能
- [soulee-dev/claude-code-notify-powershell](https://github.com/soulee-dev/claude-code-notify-powershell) — Windows PowerShell toast 通知
