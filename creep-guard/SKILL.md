---
name: creep-guard
description: >
  Prevents Claude Code from silently introducing scope creep during tasks.
  Use this skill when starting any task where staying focused matters, when
  the user says "just do X and nothing else", or when working in a codebase
  where unintended changes are costly. Also auto-triggers when Claude notices
  it is about to make a change that wasn't part of the original request.
  Supports two modes: proactive (Claude monitors autonomously) and reactive
  (user invokes explicitly before a task).
mode: proactive
---

# Creep Guard

Keeps Claude focused on the stated task. When something outside the original
request is detected, pause and ask — don't silently do it.

## Modes

- **proactive** (default): Monitor autonomously throughout any task. Interrupt
  whenever drift is detected.
- **reactive**: User invokes `/creep-guard` before a task. Hold that task
  statement as the source of truth and check against it as you work.

### Setting the Mode

- **Persistent default**: edit the `mode:` field in this file's frontmatter
  (`proactive` or `reactive`). This applies whenever the skill auto-triggers
  or is invoked with no argument.
- **Per-task override**: invoke with the mode name as the argument, e.g.
  `/creep-guard reactive` or `/creep-guard proactive`. This overrides the
  frontmatter default for the current task only and does not change the file.
- **Precedence**: if an argument is given at invocation, use it. Otherwise,
  fall back to the frontmatter `mode:` value.

## Source of Truth

The scope is defined by **the user's prompt for the current task only**. No
other documents, prior conversations, or inferred intent. If the task wasn't
stated explicitly, ask for clarification before starting.

## When to Flag

Flag something as potential scope creep only when **both** are true:

1. It was not mentioned in the original task prompt
2. It would require touching files or logic outside what the task already requires

Do not flag:
- Fixing a syntax error in a line you're already editing
- Adding a missing import required by code you were asked to write
- Handling an obvious error case in a function you were asked to create

Do flag:
- Refactoring something adjacent that "looks messy"
- Adding auth, validation, or error handling to a system you weren't asked to touch
- Creating new files, modules, or abstractions beyond what the task requires
- Upgrading dependencies while in the neighborhood
- Fixing bugs you noticed that are unrelated to the task
- Adding tasks to a plan or .md file that was beyond what was requested by the user

## The Interaction

When scope creep is detected, stop and say:

> "I noticed I could also **[specific thing]**. This wasn't part of the original task.
> What would you like to do?
>
> - **Now** — do it as part of this task
> - **Later** — log it to `tech_debt.md` and skip it for now
> - **Never** — ignore it and move on"

Wait for the user's response before continuing. Do not proceed with the
out-of-scope change while waiting.

## On "Later": Writing to tech_debt.md

Append to `tech_debt.md` in the project root (create it if it doesn't exist):

```markdown
- [ ] [YYYY-MM-DD] [one-line description of what was deferred] — deferred during: "[brief summary of original task]"
```

Keep entries short and actionable. The file is append-only — never rewrite or
reorganize existing entries.

## Confidence

Only interrupt if you are reasonably confident something qualifies. A low-confidence
hunch is not worth breaking the user's flow. When genuinely uncertain, lean toward
flagging — one unnecessary check is less costly than silently expanding scope.
