---
name: claude-code
description: Invoke Claude Code CLI for coding tasks. Use this skill when you need to call Claude Code to write code, fix bugs, or do complex development tasks.
metadata:
  {
    "openclaw": { "emoji": "🧑‍💻", "requires": { "anyBins": ["claude"] } },
  }
---

# Claude Code Skill

Use this skill to invoke Claude Code CLI for coding tasks.

## Important Notes

1. **Claude Code requires a git repository** - Always initialize a git repo in a temp directory before running Claude Code
2. **Use `--dangerously-skip-permissions`** - To avoid permission prompts in automated mode
3. **Use `-p` flag** - For print/non-interactive mode that exits after completion

## Communication Requirements (IMPORTANT)

When communicating with Claude Code, you MUST use **professional PM-level English**. This means:

1. **Clear, structured prompts** - Use proper grammar, bullet points, and numbered lists
2. **Professional tone** - Like a senior project manager giving instructions
3. **Detailed requirements** - Specify exactly what needs to be done with context
4. **Expected deliverables** - Clearly state what success looks like

### Example of Professional PM Communication

**❌ Bad (casual):**
```
claude -p "fix this bug"
```

**✅ Good (professional PM):**
```
claude -p --dangerously-skip-permissions "Please investigate and fix the null pointer exception in userService.ts. 

Requirements:
- Identify the root cause of the NPE
- Implement a proper fix following our coding standards
- Write unit tests to prevent regression
- Ensure the fix doesn't break existing functionality

Deliverable: Working fix committed to the repository."
```

## Non-Blocking Task Execution with Auto-Callback

This is the recommended way to run Claude Code tasks while continuing to handle other requests.

### Step 1: Configure the Stop Hook

Create the hook script at `~/.claude/hooks/notify-agi.sh`:

```bash
#!/bin/bash
RESULT_DIR="$HOME/.claude-code-results"
META_FILE="${RESULT_DIR}/task-meta.json"

mkdir -p "$RESULT_DIR"

# 防重复 (30秒内不重复触发)
LOCK_FILE="${RESULT_DIR}/.hook-lock"
if [ -f "$LOCK_FILE" ]; then
    LOCK_TIME=$(stat -f %m "$LOCK_FILE" 2>/dev/null || echo 0)
    NOW=$(date +%s)
    AGE=$(( NOW - LOCK_TIME ))
    if [ "$AGE" -lt 30 ]; then
        exit 0
    fi
fi
touch "$LOCK_FILE"

# 读取任务名
TASK_NAME="unknown"
if [ -f "$META_FILE" ]; then
    TASK_NAME=$(jq -r '.task_name // "unknown"' "$META_FILE" 2>/dev/null || echo "unknown")
fi

# 发送消息到 Feishu
openclaw message send --channel feishu --target "USER_ID" --message "🤖 Claude Code 任务完成: $TASK_NAME"

exit 0
```

Make it executable:
```bash
chmod +x ~/.claude/hooks/notify-agi.sh
```

### Step 2: Register the Hook

Add to `~/.claude/settings.json`:

```json
{
  "hooks": {
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "~/.claude/hooks/notify-agi.sh"
      }]
    }]
  }
}
```

### Step 3: Write Task Metadata Before Running

Before running Claude Code, create task metadata:

```bash
mkdir -p ~/.claude-code-results
echo '{"task_name": "your-task-name", "telegram_group": ""}' > ~/.claude-code-results/task-meta.json
rm -f ~/.claude-code-results/.hook-lock
```

### Step 4: Run the Task

```bash
cd /tmp
git init -q

claude -p --dangerously-skip-permissions "Your task description here"
```

### Workflow Summary

1. **Write task metadata** to `~/.claude-code-results/task-meta.json`
2. **Remove lock** (`rm -f ~/.claude-code-results/.hook-lock`)
3. **Run Claude Code** in background with `&`
4. **Continue handling** other requests normally
5. **Receive Feishu notification** when task completes automatically

## Available Flags

| Flag | Description |
|------|-------------|
| `-p`, `--print` | Print response without interactive mode |
| `--dangerously-skip-permissions` | Skip all permission prompts |
| `--model` | Specify model (sonnet, opus, haiku) |
| `--output-format` | Output format (text, json, stream-json) |
| `--add-dir` | Additional working directories |
| `-c`, `--continue` | Continue most recent conversation |
| `-r`, `--resume` | Resume specific session |
| `--system-prompt` | Custom system prompt |

## Examples

### Simple task
```bash
cd /tmp && git init -q
claude -p --dangerously-skip-permissions "Create a hello world HTML file"
```

### With task metadata and callback
```bash
echo '{"task_name": "my-task"}' > ~/.claude-code-results/task-meta.json
rm -f ~/.claude-code-results/.hook-lock
cd /tmp && git init -q
claude -p --dangerously-skip-permissions "Create a game in /path/to/game.html"
```

### Professional PM style prompt
```bash
cd /tmp && git init -q
claude -p --dangerously-skip-permissions "Create a REST API for the todo app.

Requirements:
1. Use Express.js with TypeScript
2. Implement CRUD endpoints for todos
3. Add authentication with JWT
4. Write unit tests

Deliverable: Complete working API committed to the repository."
```

## Troubleshooting

- **"Not in a git repository"**: Initialize git repo first with `git init`
- **Permission prompts**: Use `--dangerously-skip-permissions`
- **Session hangs**: Use `-p` for non-interactive mode

## Best Practices

1. Always use a git repository (create temp if needed)
2. Use `--dangerously-skip-permissions` for automation
3. Use `-p` for one-shot tasks that should exit
4. Use professional PM English for task descriptions
5. Configure Stop hook for automatic callbacks
