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

## How to Invoke Claude Code

### Basic Pattern

```bash
# Create temp git repo and run Claude Code
SCRATCH=$(mktemp -d)
cd $SCRATCH
git init -q
git config user.email "test@test.com" 2>/dev/null
git config user.name "Test" 2>/dev/null

claude -p --dangerously-skip-permissions "Your prompt here"
```

### With Working Directory

```bash
# Run in a specific directory
cd /path/to/project
git init -q 2>/dev/null || true
claude -p --dangerously-skip-permissions "Your prompt here"
```

### Advanced Options

```bash
# With specific model
claude -p --model sonnet "prompt"

# With JSON output
claude -p --output-format json "prompt"

# With custom system prompt
claude -p --system-prompt "You are a Python expert" "prompt"

# With additional directories
claude -p --add-dir /another/path "prompt"

# Continue previous session
claude -c -p "continue task"

# Resume specific session
claude -r session-name "prompt"
```

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

### Fix a bug
```bash
cd /path/to/project
claude -p --dangerously-skip-permissions "Fix the null pointer exception in userService.ts"
```

### Write tests
```bash
cd /path/to/project
claude -p --dangerously-skip-permissions "Write unit tests for the auth module"
```

### Create a new feature
```bash
cd /path/to/project
claude -p --dangerously-skip-permissions "Add a dark mode toggle to the settings page"
```

### Code review
```bash
cd /path/to/project
claude -p --dangerously-skip-permissions "Review this code for security issues"
```

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

## Troubleshooting

- **"Not in a git repository"**: Initialize git repo first with `git init`
- **Permission prompts**: Use `--dangerously-skip-permissions`
- **Slow startup**: Check network connection for API calls
- **Session hangs**: Use `-p` for non-interactive mode

## Best Practices

1. Always use a git repository (create temp if needed)
2. Use `--dangerously-skip-permissions` for automation
3. Use `-p` for one-shot tasks that should exit
4. Specify working directory with `cd` before running
5. For long tasks, consider running in background with `pty:true`

## Non-Blocking Task Execution (Recommended for Long Tasks)

For tasks where you want to continue handling other requests while Claude Code works in the background, use this pattern:

### Step 1: Configure the Hook

First, set up the Claude Code Stop hook to notify you when tasks complete:

```bash
# Create hook script
cat > ~/.claude/hooks/notify-agi.sh << 'EOF'
#!/bin/bash
RESULT_DIR="$HOME/.claude-code-results"
META_FILE="${RESULT_DIR}/task-meta.json"

mkdir -p "$RESULT_DIR"

# 防重复
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
EOF
chmod +x ~/.claude/hooks/notify-agi.sh
```

Add to ~/.claude/settings.json:
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

### Step 2: Dispatch Task

For tasks where you want to continue handling other requests while Claude Code works in the background, use this pattern:

### Step 1: Dispatch Task to Background

```bash
# Run Claude Code in background with callback
SCRATCH=$(mktemp -d)
cd $SCRATCH
git init -q
git config user.email "test@test.com" 2>/dev/null
git config user.name "Test" 2>/dev/null

claude -p --dangerously-skip-permissions "Your task description here.

When completed, run this command to notify me:
openclaw system event --text 'Claude Code task completed: [brief result summary]' --mode now" 2>&1 &
```

### Step 2: Monitor Task (Optional)

```bash
# Check if background job is still running
ps aux | grep claude | grep -v grep
```

### Step 3: Receive Callback

When Claude Code finishes, OpenClaw will receive a system event with the result. You then relay this to the user.

### Full Example

```bash
# Background execution with professional PM communication
SCRATCH=$(mktemp -d)
cd $SCRATCH
git init -q 2>/dev/null

claude -p --dangerously-skip-permissions "Create a new SwiftUI app in /Users/jerf/openclaw/workspace/MyApp.

Requirements:
1. Use SwiftUI with MVVM architecture
2. Implement a user authentication flow
3. Create a settings page with dark mode toggle
4. Build for macOS target

Deliverable: Complete Xcode project committed to the workspace." 2>&1 &

# Now you can handle other requests while Claude Code works
echo "Claude Code is running in background..."
```

## Callback Mechanism

The key is appending this to your Claude Code prompt:

```
When completed, run: openclaw system event --text 'Your result summary' --mode now
```

This will:
1. Wake up OpenClaw immediately when Claude Code finishes
2. Provide a brief summary of what was accomplished
3. Allow you to relay the detailed results to the user
