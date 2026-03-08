---
name: daily
description: Generate a daily planner from GitHub issues and pending Obsidian tasks, organized by project
---

# Daily Planner Generator

Generate a daily planner note by aggregating incomplete tasks from Obsidian and GitHub, organized by project/focus area.

## Prerequisites

GitHub CLI needs project read scope. If you get a scope error, run:
```bash
gh auth refresh -s read:project
```

## Configuration

```
Vault: <YOUR_OBSIDIAN_VAULT_PATH>
Daily notes: {vault}/<YOUR_DAILY_NOTES_FOLDER>/YYYY-MM-DD.md
Weekly notes: {vault}/<YOUR_WEEKLY_NOTES_FOLDER>/YYYY week W.md
GitHub Project: <OWNER>/projects/<PROJECT_NUMBER>
```

> **Setup:** Replace the placeholders above with your own paths and GitHub project reference.

## Optional Integrations

The skill can pull tasks from additional sources if you have the relevant MCP servers configured.

### Google Docs (gworkspace-mcp)

If the user asks to look at Google Docs for tasks/context:

- `mcp__gworkspace-mcp__search_drive` - Find docs by name or content
- `mcp__gworkspace-mcp__read_file` - Read doc content (use file_id, format="markdown")

Example triggers: "check my planning doc", "look at team priorities doc", "include tasks from [doc name]"

### Project Management Tools (vault-mcp or similar)

If the user asks to check a project management tool for project info/tasks:

- Get project details, milestones, blockers
- Get user's active projects
- Search for projects by keyword

Example triggers: "check my projects", "look at [project name]", "what's on my plate"

### Slack (slack-mcp)

If the user asks to check Slack for tasks/reminders:

- Get saved messages and reminders
- Search messages for action items

Example triggers: "check my slack reminders", "any saved messages", "look at slack for action items"

---

Extract tasks, action items, or priorities from any of these sources and include in the daily/weekly plan.

## Workflow

### Step 0: Ask About Projects Sequentially

Ask about projects and tasks **one at a time**, in order:

**First project:**
```
What's the first project you're working on today?
```

**Tasks for first project:**
```
What tasks do you have for [Project 1]?
```

If a task seems too large (multi-day, vague), suggest breaking it down:
```
"Ship inference pipeline" seems broad. Want to break it down?
- Get data design PR approved (#35897)
- Address review feedback
- Ship base models PR (#36167)

Or keep as single task?
```

**Second project:**
```
Any other project you're working on?
```

If user says "no" / "none" / "that's it" → skip to personal tasks.

If user names another project → ask for tasks, then ask about next project.

**Continue this loop** until user indicates no more projects.

### Step 0.6: Ask About Personal Tasks

**Finally, ask about personal tasks:**

```
Any personal tasks for today?
```

Add these under a separate "Personal" section at the bottom of the note.

### Step 1: Read Obsidian Notes

Read from the configured vault path:

1. **Find most recent daily note** in the daily notes folder (typically yesterday, or search for most recent YYYY-MM-DD.md)
2. **Find current week's weekly note** in the weekly notes folder using format: `YYYY week W.md` (e.g., "2026 week 4.md")
3. **Extract all unchecked tasks** matching pattern `- [ ]` from both notes

### Step 1.5 (Optional): Check Additional Sources

Only if the user explicitly asks, check configured MCP integrations for additional tasks (Google Docs, project management tools, Slack, etc.).

### Step 2: Fetch GitHub Issues

Use gh CLI to fetch project items:

```bash
gh project item-list <PROJECT_NUMBER> --owner <OWNER> --format json --limit 100
```

Filter the results:
- Items assigned to you
- Items in current iteration OR previous iteration (if still incomplete)
- Extract: title, URL, labels, iteration, status

### Step 3: Organize by Project/Focus Area

Instead of priority-based sections, organize tasks by project/focus area:

1. **Create sections** based on user's stated focus areas
2. **Group tasks** under the appropriate section
3. **Add a "Learning" section** for videos, readings, courses
4. **Add an "Other" section** for miscellaneous tasks that don't fit elsewhere

### Step 4: Present for Confirmation (in Terminal)

**Important**: Display the proposed task list in the terminal and get explicit confirmation before writing to Obsidian.

Format:

```
Proposed Daily Tasks for YYYY-MM-DD

##### Project Alpha
- [ ] Ship data design PR (#123)
- [ ] Address review feedback

##### Infrastructure
- [ ] Check on migration status (PR #456)

##### Learning
- [ ] Watch engineering talk

Ready to save to Obsidian? (yes / make changes)
```

### Step 5: Create or Append Note

**IMPORTANT**: Always fetch today's date from system bash before writing:
```bash
date +%Y-%m-%d
```
Never rely on dates mentioned earlier in conversation - always verify with system.

**Path**: `<DAILY_NOTES_FOLDER>/YYYY-MM-DD.md` (use today's date from bash)

**If note does NOT exist** - create with project-based sections:

```markdown
#### Work

##### [Project 1]
- [ ] Task 1
- [ ] Task 2

##### [Project 2]
- [ ] Task 1

##### Learning
- [ ] Task 1

#### Personal
- [ ] Task 1

```

**If note EXISTS** - append new tasks under existing sections, or create new sections if needed:

```markdown

---
## Added via /daily (HH:MM)

##### [New Section]
- [ ] New task 1
```

## Section Naming Guidelines

Use clear, concise project names:
- "AI Inference Pipeline" (not "Working on inference stuff")
- "BigQuery Optimization" (not "BQ work")
- "PR Reviews" (for review tasks)
- "Learning" (for videos, readings, courses)
- "Meetings" (if tracking meeting prep)
- "Other" (for miscellaneous)

## Task Formatting

**Important**: Never show raw URLs in tasks. Always write a clear summary of what needs to be done, with the URL hyperlinked.

Good:
- `- [ ] [Ship data design - staging models](https://github.com/.../35897)`
- `- [ ] [Review tier change PR](https://github.com/.../36543)`
- `- [ ] Watch [Eng Talk](https://example.com/video/...)`

Bad:
- `- [ ] PR #35897`
- `- [ ] https://github.com/org/repo/pull/35897`
- `- [ ] Ship data design https://github.com/...`

## Date Calculations

- Today: `date +%Y-%m-%d`
- Day of week: `date +%u` (1=Monday, 7=Sunday)
- Current week number: `date +%V`
- Current year: `date +%Y`
- Weekly note name: `YYYY week W.md` (e.g., "2026 week 4.md")

## Error Handling

- If no previous daily note found: Skip that source, proceed with others
- If weekly note not found: Skip that source, proceed with others
- If GitHub fetch fails: Report error, proceed with Obsidian tasks only
- If vault path doesn't exist: Error and stop

## Example Session

```
User: /daily

Claude: What's the first project you're working on today?

User: AI inference pipeline

Claude: What tasks do you have for AI Inference Pipeline?

User: Ship data design PR #123 and base models PR #456

Claude: Any other project you're working on?

User: Database maintenance

Claude: What tasks do you have for Database Maintenance?

User: Check with team on billing migration PR #789

Claude: Any other project you're working on?

User: Nope, that's it. Also want to watch an eng talk today.

Claude: Any personal tasks for today?

User: nope

Claude: Proposed Daily Tasks for 2026-01-26

##### AI Inference Pipeline
- [ ] [Ship data design - staging models](https://github.com/.../123)
- [ ] [Ship base models - core models](https://github.com/.../456)

##### Database Maintenance
- [ ] [Check with team on billing migration](https://github.com/.../789)

##### Learning
- [ ] Watch eng talk

Ready to save to Obsidian? (yes / make changes)

User: yes

Claude: Created: 2026-01-26.md
```

## Example with Task Breakdown

```
User: /daily

Claude: What's the first project you're working on today?

User: GPU cost tracking

Claude: What tasks do you have for GPU Cost Tracking?

User: Need to finish the whole pipeline

Claude: "Finish the whole pipeline" seems broad. Want to break it down?

Suggested:
- Get data design PR approved (#123)
- Address review feedback
- Ship base models PR (#456)
- Update dashboard

Or keep as single task?

User: Let's go with the breakdown but skip the dashboard for now

Claude: Any other project you're working on?

User: No, that's it

Claude: Any personal tasks for today?

User: Need to pick up groceries

Claude: Proposed Daily Tasks for 2026-01-26

##### GPU Cost Tracking
- [ ] [Get data design PR approved](https://github.com/.../123)
- [ ] Address review feedback on data design
- [ ] [Ship base models PR](https://github.com/.../456)

#### Personal
- [ ] Pick up groceries

Ready to save to Obsidian? (yes / make changes)
```
