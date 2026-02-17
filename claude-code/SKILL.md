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
