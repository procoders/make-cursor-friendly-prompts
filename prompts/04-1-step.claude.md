# Cursor → Claude Code Migration

You are Claude Code running INSIDE this repository with full access to read/write files.

## Goal

Migrate ALL Cursor rules from `.cursor/rules/**` into a Claude Code setup without losing anything, so developers can use either Cursor or Claude Code on the same project.

---

## Hard requirements

- Do NOT delete or modify anything under `.cursor/rules/**` (Cursor stays supported).
- Create/Update a root `CLAUDE.md` that is VERY short: 20–40 lines total.
  - It must contain only the most universally important rules that should matter for ANY request.
  - Prefer content coming from Cursor rules where `alwaysApply: true` (or equivalent "always on" intent).
  - Do NOT paste long rule bodies into `CLAUDE.md`.
- Translate Cursor rules into:
  1) `.claude/rules/**` for deterministic, path-scoped rules (based on globs/paths).
  2) `.claude/skills/**` for "apply intelligently" rules (semantic activation driven by description/intent).
- Preserve coverage: EVERYTHING in `.cursor/rules/**` must end up in Claude Code either as:
  - a line/bullet in `CLAUDE.md`, or
  - a `.claude/rules/*.md` file, or
  - a `.claude/skills/*/SKILL.md` skill (optionally with additional helper files).
- Do not invent new policies. Only reorganize/condense what already exists in `.cursor/rules/**`.
- Keep wording close to source; condense only when necessary, but do not drop meaning.

---

## Step-by-step plan (execute in this order)

### 1) Inventory Cursor rules

- Enumerate and read every file in `.cursor/rules/**` (including `.mdc`).
- For each rule file, extract:
  - any metadata fields (e.g., `alwaysApply`, `globs`, `description`, etc.)
  - the rule body (instructions)
- **Build an internal checklist of "atomic rule points"** so nothing is missed:
  - For EACH rule file, list every distinct bullet/instruction as a numbered item.
  - Track each item's migration status throughout the process.

### 2) Classification logic (IMPORTANT)

For each Cursor rule (or subsection) decide destination:

#### A) Put into `CLAUDE.md` ONLY if:
- It is truly global and always-on (typically `alwaysApply: true` and NOT narrowly path-specific),
- AND it is one of the top universally important constraints/conventions.

Keep this as short bullets; do not exceed 40 lines total.

#### B) Convert into `.claude/rules/*.md` if:
- It has `globs` (or clearly scoped to file paths / file types / directories),
- OR it is deterministic and should apply whenever working in those paths.

**Implementation:**
- Create one `.claude/rules/<same-or-similar-name>.md` per Cursor rule file that has globs/path scope.
- Translate Cursor `globs` → Claude `paths` 1:1.
  - Use YAML frontmatter:
    ```yaml
    ---
    paths:
      - "app/GraphQL/**/*.php"
      - "graphql/**/*.graphql"
    ---
    ```
  - Quote path entries if they contain special characters.
- Put the relevant rule content in the body (bullets, short sections). Keep it actionable.

#### C) Convert into `.claude/skills/**` if it is "Apply Intelligently", i.e.:
- `alwaysApply: false` AND there is a semantic `description` like "Activate when …"
- OR the rule is clearly intended to load only when the task context matches (public API contracts, migrations, release process, incident debugging, etc.)

**Implementation:**
- Create `.claude/skills/<kebab-skill-name>/SKILL.md` with YAML frontmatter:
  ```yaml
  ---
  name: <kebab-case>
  description: <very specific: when to use + trigger keywords + typical tasks>
  ---
  ```
- In SKILL.md include:
  - ## When to use (explicit triggers)
  - ## Instructions (step-by-step)
  - ## Examples (1–3 short examples)
  - ## Gotchas / Anti-patterns (only if present in source rules)
- If the content is long, split supporting material into additional files in the skill folder
  (e.g., checklist.md, templates/*) and reference them from SKILL.md.

#### D) Edge case handling (do NOT drop anything)
- If a Cursor rule has BOTH globs AND a semantic "activate when …" description:
  - Keep the hard, path-specific constraints in `.claude/rules/...` (with paths)
  - Put the deeper playbook/procedure content into a Skill
  - Cross-reference them (one line) so it's clear they're related.
- If a rule has no globs but is NOT global and is context-specific → Skill.

---

### 3) Cursor-specific instruction handling (CRITICAL)

When encountering Cursor-specific instructions that don't have a direct Claude Code equivalent:

#### 3a) Attempt interpretation/adaptation:
- **File paths**: `.cursor/rules/*.mdc` → adapt to mention BOTH `.cursor/rules/*.mdc` (Cursor) AND `.claude/rules/*.md` / `.claude/skills/*/SKILL.md` (Claude Code)
- **Cursor commands**: Interpret the intent and provide Claude Code equivalent if exists
- **Cursor-specific features**: Map to nearest Claude Code feature or workflow

#### 3b) If adaptation is possible:
- Rewrite the instruction to be tool-agnostic or dual-tool compatible
- Preserve the original intent while updating the mechanism

#### 3c) If adaptation is NOT possible:
- Keep the original instruction verbatim
- Add a comment/note: `<!-- Cursor-specific: [reason why cannot adapt] -->`
- Log a warning in the migration report

---

### 4) Migrate MCP configuration

If `.cursor/mcp.json` exists, migrate it to `.mcp.json` (project root):

#### Source format (.cursor/mcp.json):
```json
{
  "mcpServers": {
    "server-name": {
      "url": "https://example.com/mcp"
    }
  }
}
```

#### Target format (.mcp.json in project root):
```json
{
  "mcpServers": {
    "server-name": {
      "type": "http",
      "url": "https://example.com/mcp"
    }
  }
}
```

**Notes:**
- Claude Code uses `.mcp.json` at project root (not in `.claude/`)
- Add `"type": "http"` for URL-based servers
- For command-based servers, use: `"command"`, `"args"`, `"env"` fields
- Support environment variable expansion: `${VAR}` or `${VAR:-default}`

---

### 5) Create folders/files

- Create `.claude/rules/` and `.claude/skills/` if missing.
- Generate the migrated files per the classification logic above.
- Keep filenames stable and obvious (prefer matching the original Cursor rule names).

---

### 6) Create/Update `CLAUDE.md` (20–40 lines total)

**Structure:**
- 1 short title line
- 5–12 bullets of global always-on rules (the most important ones only)
- 3–8 lines of "How to work in this repo" only if reliably derived from existing repo scripts/docs
- 1–2 lines pointing to `.claude/rules/` and `.claude/skills/` for details

**Rules:**
- Do NOT include long blocks.
- Do NOT use @import to pull big files.

---

### 7) Validation: prove NOTHING was missed (MANDATORY)

This step is CRITICAL. Do not skip or abbreviate.

#### 7a) Build atomic rule inventory table

For EACH source file, list EVERY bullet point / instruction as a separate row:

| Source File | Rule # | Original Text (first 50 chars) | Migrated To | Status |
|-------------|--------|--------------------------------|-------------|--------|
| 00-rules-generator.mdc | 1 | "Only update docs when..." | CLAUDE.md:25 | ✅ |
| 00-rules-generator.mdc | 2 | "When such a shift occurs..." | CLAUDE.md:26 | ✅ |
| ... | ... | ... | ... | ... |

**Status values:**
- ✅ = Migrated verbatim or with minimal condensing
- 🔄 = Adapted (Cursor-specific → Claude Code equivalent)
- ⚠️ = Partially migrated (explain what was lost)
- ❌ = Not migrated (must have strong justification)

#### 7b) Cross-reference validation

For each source file, run grep/search on the new Claude Code artifacts to verify distinctive phrases appear:

```
Source: "sentryMonitor" from 04-global-reuse.mdc
Found in: CLAUDE.md ✅
```

#### 7c) Summary statistics

```
Total source files: X
Total atomic rules: Y
Migrated verbatim: A
Adapted: B
Partially migrated: C
Not migrated: D (list each with reason)
```

#### 7d) Final diff summary

```bash
git diff --stat
git status --short
```

List all newly created/updated files.

---

## Anti-patterns to AVOID

1. **DO NOT condense multiple distinct rules into one bullet** without tracking each original.
2. **DO NOT skip "meta" rules** (like rules about updating rules) — adapt them.
3. **DO NOT assume** a rule is "obvious" or "implied" — if it's written, migrate it.
4. **DO NOT leave validation incomplete** — every source bullet must have a destination.
5. **DO NOT invent new policies** — only reorganize existing content.

---

## Execution order

Think harder, use todo tasklist, parallel subagents and begin now:

1. **First**: Inventory `.cursor/rules/**` — create atomic rule checklist with numbered items per file.
2. **Second**: Check for `.cursor/mcp.json` and plan MCP migration.
3. **Third**: Generate `.claude/rules/**`, `.claude/skills/**` — track each atomic rule as you migrate.
4. **Fourth**: Generate `.mcp.json` if source exists.
5. **Fifth**: Create the short `CLAUDE.md`.
6. **Finally**: Output the FULL migration report with atomic-level tracking table + diff summary.

**Do not mark the task complete until the validation table shows 100% coverage.**
