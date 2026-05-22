# CortexNexus System Guide / 系统指南

This document is the complete technical reference for CortexNexus. If anything breaks, a device needs replacing, or a new tool joins — read this.

本文档是 CortexNexus 的完整技术参考。如果任何组件出问题、需要换设备、或新工具接入——读这份就够了。

---

## 1. Overview / 概述

### What This Is / 这是什么

CortexNexus is a protocol for synchronizing AI memory across multiple devices and tools. It ensures that every AI assistant you use — on any device — has the same understanding of who you are.

CortexNexus 是一个跨设备、跨工具的 AI 记忆同步协议。它确保你在任何设备上使用的任何 AI 助手，都对你有一致的理解。

### How It Works / 运作原理

1. **Memory Engine** (e.g., Hermes) compresses conversation notes into compact summaries
2. **Conversation Tools** (e.g., Claude Code) write brief notes after each session
3. **Git** synchronizes everything across devices
4. **You** trigger sync manually — no automation, no schedules

### Core Principles / 核心原则

| Principle | English | 中文 |
|-----------|---------|------|
| Memory Engine is the brain | All compression handled by one tool | 所有压缩由一个工具完成 |
| Tools are replaceable | Swap the engine or any conversation tool | 可随时替换引擎或对话工具 |
| Files are the bridge | All exchange happens through shared Git repo | 所有交换通过共享 Git 仓库完成 |
| User controls sync | No automation, no reminders | 没有自动化，没有提醒 |
| Dual-layer memory | Short-term (overwritten) + long-term (append-only) | 短时（覆盖式）+ 长时（追加式） |

---

## 2. Repository Structure / 仓库结构

```
my-memory/                         ← [your-path]
├── README.md                      # Project intro (if forking CortexNexus)
├── hermes_to_[member].md          # Memory engine's guide per member (overwritten, with timestamp)
├── hermes_memory_archive.md       # Long-term archive (append-only)
├── [member]_notes_[device].md     # Conversation notes per member per device
├── [member]_skills_[device].md    # Skill inventory per member per device
└── last_sync.md                   # Sync timestamp
```

### File Descriptions / 文件说明

| File | English | 中文 | Writer | Reader |
|------|---------|------|--------|--------|
| `hermes_to_[member].md` | Short-term memory for each member, with timestamp | 每个成员的短时记忆，带时间戳 | Memory Engine | Corresponding member |
| `hermes_memory_archive.md` | Long-term archive, append-only, never deleted | 长期档案，追加式，永不删除 | Memory Engine | Any member (on demand) |
| `[member]_notes_[device].md` | Conversation notes from each tool on each device | 每个工具在每台设备上的对话笔记 | Each tool | Memory Engine |
| `[member]_skills_[device].md` | Skill/tool inventory for each member on each device | 每个成员在每台设备上的技能清单 | Each tool | Each tool |
| `last_sync.md` | Timestamp of last sync operation | 最后同步时间 | Any | Any |

---

## 3. Naming Convention / 命名规范

### Format / 格式

```
[member]_notes_[device].md
[member]_skills_[device].md
hermes_to_[member].md
```

### Rules / 规则

| Part | Specification | Example |
|------|---------------|---------|
| `member` | Tool name, all lowercase | `claude`, `hermes`, `gpt`, `gemini` |
| `device` | User-defined identifier, all lowercase | `home`, `office`, `laptop` |
| Separator | Underscore `_` only | No hyphens, no spaces |

### Validation / 验证

The memory engine checks all filenames on every sync operation:

- **Invalid format** → notifies user, suggests correct name, assists with rename
- **Missing device file** → notifies user, assists with creation
- **Convention forgotten** → reads this document, explains rules to user

---

## 4. Complete Workflow / 完整工作流程

### 4.1 Upload Memory / 上传记忆

Triggered by user saying "Upload memory" / 用户说"上传记忆"时触发：

```
STEP 1 — Conversation Tool / 对话工具
  Input:  User says "Upload memory"
  Action:
    1. Write session notes to [member]_notes_[device].md
    2. Update [member]_skills_[device].md (if changed)
    3. git add . && git commit && git push
  Output: "Done. Tell the memory engine to upload memory."

STEP 2 — Memory Engine / 记忆引擎
  Input:  User says "Upload memory"
  Action:
    1. git pull (fetch notes pushed by conversation tool)
    2. Read timestamp from hermes_to_[member].md
    3. Compare modification times of all *_notes_*.md files
    4. Compress only NEW notes (newer than timestamp)
    5. Overwrite hermes_to_[member].md (update timestamp + source device)
    6. Append to hermes_memory_archive.md
    7. Delete digested old notes (keep last 2-3 weeks)
    8. Update [member]_skills_[device].md (if changed)
    9. Validate all filenames follow naming convention
    10. git add . && git commit && git push
```

### 4.2 Download Memory / 下载记忆

Triggered by user saying "Download memory" / 用户说"下载记忆"时触发：

```
STEP 1 — Memory Engine / 记忆引擎
  Input:  User says "Download memory"
  Action:
    1. git pull (fetch latest from remote)
    2. Read timestamps from all hermes_to_*.md files
    3. Compare with all *_notes_*.md modification times
    4. Compress new notes if any; skip if already current
    5. Overwrite hermes_to_[member].md (update timestamp if new insights)
    6. Append to hermes_memory_archive.md (if new insights)
    7. Validate all filenames follow naming convention
    8. git add . && git commit && git push

STEP 2 — Conversation Tool / 对话工具
  Input:  User says "Download memory"
  Action:
    1. git pull
    2. Read hermes_to_[member].md
  Output: Tool now understands the user
```

### 4.3 Why Two Steps? / 为什么分两步？

The memory engine and conversation tools are separate processes. They cannot invoke each other directly. The shared Git repo is the bridge.

记忆引擎和对话工具是独立进程，无法直接调用对方。共享 Git 仓库是桥梁。

**Order matters / 顺序很重要：**
- Upload: Conversation tool writes FIRST → Memory engine compresses SECOND
- Download: Memory engine pulls & compresses FIRST → Conversation tool reads SECOND

---

## 5. Hermes Prompt / Hermes 提示词

Paste this into your Hermes instance. Replace `[MEMORY_REPO_PATH]` and `[DEVICE_ID]`.

将以下内容粘贴到你的 Hermes 实例中。替换 `[MEMORY_REPO_PATH]` 和 `[DEVICE_ID]`。

```
You are the core of a shared memory system — the memory engine.

Your memory repository is at [MEMORY_REPO_PATH]

If the repository does not exist, clone it first:
git clone [YOUR_REPO_URL] [MEMORY_REPO_PATH]

Then read README.md and SYSTEM_GUIDE.md to understand the full system.

Your device identifier: [DEVICE_ID]

Your role: Long-term memory manager + mentor for all members.

## Naming Convention
You are responsible for validating all filenames in the repository:
- Notes files: [member]_notes_[device].md
- Skills files: [member]_skills_[device].md
- Memory engine guides: hermes_to_[member].md
If you find non-compliant files, notify the user and assist with fixing.

## Core Responsibility
Distill insights from all members' conversation notes into compact, actionable guidance files.
You are the brain of this system. All members learn about the user through the hermes_to_[member].md files you write.

## When user says "Upload memory"
1. cd [MEMORY_REPO_PATH] && git pull
2. Read timestamp from hermes_to_claude.md (and all hermes_to_*.md)
3. Compare modification times of all *_notes_*.md files
4. Compress only notes newer than the timestamp
5. Overwrite hermes_to_claude.md (update timestamp + source device)
6. Update other hermes_to_[member].md files if applicable
7. Append to hermes_memory_archive.md (date-stamped, never delete)
8. Delete digested old notes (keep last 2-3 weeks)
9. Update hermes_skills_[your_device].md (if changed)
10. Validate all filenames follow naming convention
11. git add . && git commit -m "Hermes compress: [date]" && git push

## When user says "Download memory"
1. cd [MEMORY_REPO_PATH] && git pull
2. Read timestamps from all hermes_to_*.md files
3. Compare with all *_notes_*.md modification times
4. Compress new notes if any; skip if already current
5. Overwrite hermes_to_[member].md (update timestamp if new insights)
6. Update other hermes_to_[member].md files if applicable
7. Append to hermes_memory_archive.md (if new insights)
8. Validate all filenames follow naming convention
9. git add . && git commit -m "Hermes update: [date]" && git push

## New Member Onboarding
When a new AI tool joins the system:
1. Tell it to read README.md and SYSTEM_GUIDE.md
2. Create [member]_notes_[device].md following naming convention
3. Create [member]_skills_[device].md following naming convention
4. Create hermes_to_[member].md (your dedicated guide for this member)
5. Tell the member: write notes to your own file, Hermes will digest them
6. Tell the member: read hermes_to_[member].md on startup to understand the user
7. Update README.md file table

## New Device Onboarding
When a new device is added:
1. Confirm the device identifier with the user
2. Create corresponding notes and skills files for all existing members
3. Update README.md file table
4. Ensure all filenames follow naming convention

## Naming Error Handling
If you find malformed filenames:
1. Notify user: "Found file XXX with non-standard naming"
2. Read this document's naming convention section
3. Suggest correct name
4. Help user rename the file
5. Update README.md

If user forgot the naming convention:
1. Read this document's naming convention section
2. Explain the rules to user
3. Help audit and fix all files

## Key Principles
- hermes_to_[member].md is your most important output — keep it concise, overwrite each time
- hermes_memory_archive.md is the complete history — append only, never delete
- You absorb notes from ALL members — compress uniformly,不分成员
- Strictly maintain naming convention — fix violations immediately
- No fixed schedule, no reminders — user triggers all sync operations
```

---

## 6. New Device Setup / 新设备接入

### Step 1: Install Prerequisites / 安装前置工具

1. **Hermes** (required): Follow [Hermes installation guide](https://github.com/nicepkg/aide)
2. **Git**: Pre-installed or install from https://git-scm.com
3. **GitHub CLI** (optional): `winget install --id GitHub.cli -e` (Windows) or `brew install gh` (macOS)
4. **A conversation tool** (optional): Claude Code, Gemini CLI, Codex, OpenCode, etc.

### Step 2: Authenticate / 登录

```bash
gh auth login
```

### Step 3: Configure Git / 配置 Git

```bash
git config --global user.name "your-username"
git config --global user.email "your-email"
```

### Step 4: Clone Memory Repo / 克隆记忆仓库

```bash
git clone https://github.com/YOUR_USERNAME/my-memory.git [YOUR_PATH]
```

### Step 5: Choose Device Identifier / 选择设备标识

Pick a short, lowercase identifier: `home`, `office`, `laptop`, `desktop`, etc.

### Step 6: Create Files / 创建文件

For each existing member in the system, create:

```bash
cp templates/notes_template.md [member]_notes_[your_device].md
cp templates/skills_template.md [member]_skills_[your_device].md
```

### Step 7: Feed Prompts / 注入提示词

- **Hermes**: Paste the prompt from [Section 5](#5-hermes-prompt--hermes-提示词)
- **Conversation tools**: Use the template from [README.md New Member Onboarding](README.md#new-member-onboarding--新成员接入)

### Step 8: Test / 测试

1. Chat with your conversation tool
2. Say "Upload memory" — verify notes are written and pushed
3. Tell Hermes "Upload memory" — verify compression and push
4. On another device, say "Download memory" — verify the full flow

---

## 7. Disaster Recovery / 灾难恢复

| Scenario | Impact | Recovery |
|----------|--------|----------|
| Conversation tool memory lost | Tool doesn't know user, but hermes_to_[member].md still exists | Re-feed prompt; tool reads hermes_to_[member].md |
| Memory engine memory lost | Engine forgets compression history, but archive still exists | Re-feed prompt; engine reads hermes_memory_archive.md |
| Local repo corrupted | Local files lost | `git clone` again from remote |
| GitHub repo lost | Remote backup gone | `git push` from any device that still has the repo |
| Everything lost | All memory gone | Recreate repo from scratch; manually tell memory engine basics |
| Tool reinstalled on device | Fresh install, no memory | Re-feed prompt; all memory is in the Git repo |
| File naming corrupted | Memory engine can't identify note sources | Tell memory engine "check naming convention"; it reads this doc and fixes |
| New device setup failed | Sync not working | Verify: git clone succeeded, gh auth login passed, correct device identifier |

---

## 8. Extensibility / 可扩展性

### Adding a New Memory Engine / 更换记忆引擎

When a better memory engine emerges:
1. Install the new engine
2. Give it the Hermes prompt (replace tool-specific parts)
3. The new engine reads the same files, follows the same protocol
4. Remove Hermes if no longer needed

### Adding a New Conversation Tool / 添加新对话工具

1. Create `[new_tool]_notes_[device].md`
2. Create `[new_tool]_skills_[device].md`
3. Create `hermes_to_[new_tool].md`
4. Feed the new member prompt (from README.md)
5. Memory engine verifies

### Adding a New Device / 添加新设备

See [Section 6](#6-new-device-setup--新设备接入).

---

## Version History / 版本历史

- 2026-05-23: Initial release — protocol, templates, documentation
