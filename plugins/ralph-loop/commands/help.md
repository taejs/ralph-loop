---
description: "Explain Ralph Loop plugin and available commands"
---

# Ralph Loop Plugin Help

Please explain the following to the user:

## What is Ralph Loop?

Ralph Loop implements the Ralph Wiggum technique - an iterative development methodology based on continuous AI loops, pioneered by Geoffrey Huntley.

**Core concept:**
```bash
while :; do
  cat PROMPT.md | claude-code --continue
done
```

The same prompt is fed to Claude repeatedly. Each iteration sees previous work in files and git history, building incrementally toward the goal.

**Each iteration:**
1. Claude receives the SAME prompt
2. Works on the task, modifying files
3. Tries to exit
4. Stop hook intercepts and feeds the same prompt again
5. Claude sees its previous work in the files
6. Iteratively improves until completion

The technique is described as "deterministically bad in an undeterministic world" - failures are predictable, enabling systematic improvement through prompt tuning.

## Available Commands

### /ralph-loop <PROMPT> [OPTIONS]

Start a Ralph loop in your current session.

**Usage:**
```
/ralph-loop "Refactor the cache layer" --max-iterations 20
/ralph-loop "Add tests" --completion-promise "TESTS COMPLETE"
/ralph-loop --name "auth-fix" "Fix the auth bug" --max-iterations 20
```

**Options:**
- `--max-iterations <n>` - Max iterations before auto-stop
- `--completion-promise <text>` - Promise phrase to signal completion
- `--name <id>` - Named loop (letters, numbers, hyphens, underscores)

**Loop isolation:**

By default each session gets its own isolated loop — multiple Claude Code windows in the same project never interfere with each other.

State file: `.claude/ralph-loop-<session_id>.local.md`

Use `--name` to create a named loop that can be shared or resumed across sessions:

```
# Session A - start
/ralph-loop --name "my-task" "Fix the auth bug"

# Session B - resume after crash
/ralph-loop --name "my-task" "Fix the auth bug"
```

State file: `.claude/ralph-loop-<name>.local.md`

**How it works:**
1. Creates a session-scoped (or named) state file under `.claude/`
2. You work on the task
3. When you try to exit, stop hook intercepts
4. Same prompt fed back
5. You see your previous work
6. Continues until promise detected or max iterations

---

### /cancel-ralph

Cancel the active Ralph loop for this session.

**Usage:**
```
/cancel-ralph
```

**How it works:**
- Finds the state file belonging to this session (by session ID)
- Removes the file
- Reports: "Cancelled Ralph loop '<id>' (was at iteration N)"

---

## Key Concepts

### Completion Promises

To signal completion, Claude must output a `<promise>` tag:

```
<promise>TASK COMPLETE</promise>
```

The stop hook looks for this tag. Without it (or `--max-iterations`), Ralph runs infinitely.

### Session Isolation

Each Claude Code session gets its own state file by default. Multiple sessions in the same project directory run independently without interfering.

Use `--name` when you want to:
- Resume a loop after a session crash
- Share loop state across windows (intentionally)

## Examples

### Parallel independent tasks
```
# Window 1
/ralph-loop "Build auth service" --max-iterations 20 --completion-promise "DONE"

# Window 2 (same project, no interference)
/ralph-loop "Build API layer" --max-iterations 20 --completion-promise "DONE"
```

### Cross-session resume
```
/ralph-loop --name "big-refactor" "Refactor the cache layer" --max-iterations 50
# If session crashes, resume in a new window:
/ralph-loop --name "big-refactor" "Refactor the cache layer" --max-iterations 50
```

### Basic bug fix
```
/ralph-loop "Fix the token refresh logic in auth.ts. Output <promise>FIXED</promise> when all tests pass." --completion-promise "FIXED" --max-iterations 10
```

## When to Use Ralph

**Good for:**
- Well-defined tasks with clear success criteria
- Tasks requiring iteration and refinement
- Running independent tasks in parallel sessions
- Long-running tasks that may need session resume

**Not good for:**
- Tasks requiring human judgment or design decisions
- One-shot operations
- Tasks with unclear success criteria

## Learn More

- Original technique: https://ghuntley.com/ralph/
- Plugin source: https://oss.navercorp.com/taerim-shin/claude-plugins
