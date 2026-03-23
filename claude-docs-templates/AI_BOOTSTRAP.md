# AI Bootstrap — Auto-Implement `.claude-docs` for Any Project

> **For AI assistants:** Follow these instructions to set up structured session memory for the user's project.
> **For humans:** Paste this file (or a link to it) into your AI assistant and say "Set up .claude-docs for my project."

---

## Instructions for AI

You are setting up a structured development methodology called `.claude-docs` for the user's project. This system maintains project context across AI sessions, prevents architectural shortcuts, and tracks lessons learned.

### Step 1: Analyze the Project

Before creating any files, explore the codebase to understand:

1. **Project type** — web app, CLI tool, library, mobile app, data pipeline, etc.
2. **Tech stack** — languages, frameworks, databases, deployment targets
3. **Directory structure** — where source code, tests, config, and docs live
4. **Build system** — how to build, test, and run the project
5. **Existing patterns** — coding conventions, naming, architecture decisions already in place
6. **Version tracking** — any existing version files, changelogs, package.json versions
7. **Pain points** — ask the user what causes the most rework or confusion

### Step 2: Create the Directory

```bash
mkdir -p .claude-docs
```

### Step 3: Generate MEMORY.md

Create `.claude-docs/MEMORY.md` populated with actual project details:

- **Project overview** — what you learned from exploring the codebase
- **Environment** — actual paths, ports, database info, runtime versions (check `package.json`, `requirements.txt`, `Dockerfile`, etc.)
- **Version numbers** — extract from package.json, pyproject.toml, version files, etc.
- **Anti-shortcut rules** — based on the project's complexity and the user's stated pain points
- **Recent context** — ask the user what they've been working on recently

### Step 4: Generate STANDARDS.md

Create `.claude-docs/STANDARDS.md` with standards derived from the codebase:

- **Scan for existing patterns** — look at how existing code handles auth, validation, error handling, testing
- **Identify conventions** — naming (camelCase vs snake_case), file organization, import style
- **Extract implicit rules** — if every API endpoint has auth middleware, that's a standard (§)
- **Number everything** — use `§N` references so future sessions can cite specific rules
- **Start with 10-15 standards** — more will be added as patterns emerge

Examples of standards to look for:
- How are API endpoints structured? (auth, validation, response format)
- How is state managed? (Redux, hooks, context, stores)
- How are errors handled? (try/catch patterns, error boundaries, logging)
- How are tests organized? (unit, integration, e2e, naming conventions)
- How are database queries done? (ORM patterns, raw SQL, migration approach)

### Step 5: Generate DEPENDENCIES.md

Create `.claude-docs/DEPENDENCIES.md` with change impact chains:

- **Map the wiring** — for each major feature type, trace what files need updating
- **Look at recent commits** — multi-file commits reveal dependency chains
- **Ask the user** — "What changes tend to require touching multiple files?"
- **Include the version bump reminder** at the top

Common dependency chains to document:
- Adding a new API endpoint (route → schema → frontend → tests → docs)
- Adding a new database model (model → migration → seed → route → frontend)
- Adding a new feature end-to-end (the full chain from backend to frontend)
- Changing configuration (env vars → docker → deploy scripts → docs)

### Step 6: Generate SESSION_LOG.md

Create `.claude-docs/SESSION_LOG.md` with:

- A header explaining the purpose
- An entry for this session: "Session 1 — Initial .claude-docs setup"
- List what was created and any decisions made

### Step 7: Generate LESSONS.md

Create `.claude-docs/LESSONS.md` with:

- A header explaining the purpose
- Ask the user: "What mistakes have caused the most rework in this project?"
- Document each as an L-NNN entry with the problem and correct approach
- If the user can't think of any, add 2-3 universal lessons:
  - L-001: Read before write (always check current state before coding)
  - L-002: Test before commit
  - L-003: Don't assume — verify (check actual data, not assumptions)

### Step 8: Generate SESSION_PROTOCOL.md

Create `.claude-docs/SESSION_PROTOCOL.md` with:

- Session start/end checklists
- How to handle complex tasks (plan mode, todo lists)
- Commit message standards

### Step 9: Create or Update Project Root Config

**For Claude Code:** Create or update `CLAUDE.md` in the project root:
```markdown
## FIRST ACTION: Read Documentation
Before writing ANY code, read these files in order:
1. `.claude-docs/MEMORY.md`
2. `.claude-docs/STANDARDS.md`
3. `.claude-docs/DEPENDENCIES.md`
```

**For Cursor:** Add to `.cursorrules`:
```
Always read .claude-docs/MEMORY.md and .claude-docs/STANDARDS.md before making changes.
Reference standards by § number when applying patterns.
```

**For other AI tools:** Tell the user to paste MEMORY.md content at the start of each session.

### Step 10: Verify and Iterate

- Ask the user to review each generated file
- Make adjustments based on their feedback
- The system improves over time — LESSONS.md and STANDARDS.md grow with each session

---

## For Humans: How to Use This

### Option A: Automated Setup
1. Open your AI coding assistant (Claude Code, Cursor, Copilot Chat, ChatGPT, etc.)
2. Share this file or paste its contents
3. Say: **"Set up .claude-docs for my project at [path]"**
4. The AI will explore your codebase and generate project-specific documentation

### Option B: Manual Setup
1. Copy the templates from the `templates/` directory
2. Fill in the placeholders with your project details
3. Add `CLAUDE.md` (or equivalent) to your project root

### Option C: Hybrid
1. Let the AI generate the initial docs (Option A)
2. Review and customize manually
3. The best `.claude-docs` systems are shaped by real development experience

---

## Universal Prompt (Copy-Paste Ready)

If your AI assistant doesn't have file access, paste this prompt:

```
I want to set up a structured AI development methodology called ".claude-docs"
for my project. This system maintains context across sessions, enforces coding
standards, and tracks lessons learned.

Please help me create the following files in a .claude-docs/ directory:

1. MEMORY.md — Project context, environment, versions, anti-shortcut rules
2. STANDARDS.md — Numbered (§) development patterns I must follow
3. DEPENDENCIES.md — Change impact checklists ("if you change X, also change Y")
4. SESSION_LOG.md — Session history template
5. LESSONS.md — Mistake prevention log
6. SESSION_PROTOCOL.md — Session start/end procedures

My project is: [describe your project]
Tech stack: [list your technologies]
Main pain points: [what causes rework]
```

---

## Why This Works

The `.claude-docs` system succeeds because it solves the three biggest problems with AI-assisted development:

1. **Context amnesia** → MEMORY.md persists state across sessions
2. **Shortcut tendency** → STANDARDS.md + CLAUDE.md enforce quality constraints
3. **Forgotten wiring** → DEPENDENCIES.md ensures changes propagate to all affected files

The methodology was developed over 160+ sessions building an enterprise monitoring platform. It's been battle-tested on a codebase with 23 backend route modules, 7 database drivers, 19 SAP surveillance categories, and 524 passing tests.

**Source:** [monlite-portfolio](https://github.com/josh-lans/monlite-portfolio) by Joshua Lans
