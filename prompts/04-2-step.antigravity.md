# Cursor → Antigravity Migration (Smart-Load + Policy)

You are Google Antigravity Agent running INSIDE this repository with full read/write access to files.

## Mission
Migrate ALL Cursor rules from `.cursor/rules/**` into Antigravity workspace customizations so that developers can use either Cursor or Antigravity on the same repo — with ZERO loss of coverage.

Antigravity target locations:
- Workspace Rules: `./.agent/rules/` :contentReference[oaicite:3]{index=3}
- Workspace Workflows: `./.agent/workflows/` :contentReference[oaicite:4]{index=4}

Antigravity rule activation methods include (use them intentionally):
- Always On
- Model Decision
- Glob (path/file-pattern based)
- Manual (@-mention based) :contentReference[oaicite:5]{index=5}

## Non-negotiable requirements
1) DO NOT delete or modify anything under `.cursor/rules/**` (Cursor remains supported).
2) Create Antigravity equivalents without losing anything:
   Every atomic instruction from Cursor must land in EXACTLY ONE place:
   - `./.agent/rules/AGENTS.md` (always_on, SHORT), or
   - a `./.agent/rules/*.md` rule (glob/model_decision/manual), or
   - a `./.agent/workflows/*.md` workflow (slash command).
3) “Smart loading” policy:
   - `always_on` MUST stay very short: 20–40 lines total.
   - Everything that is not truly universal must be moved OUT of always_on into:
     - `trigger: glob` (if file/path scoped), OR
     - `trigger: model_decision` (semantic activation), OR
     - `./.agent/workflows/` (user-invoked repeatable procedures).
4) Do NOT invent new engineering policies. Reorganize/adapt what already exists in Cursor rules.
5) Cursor-specific instructions must be preserved:
   - If there is no Antigravity equivalent, keep verbatim and annotate as Cursor-specific.
6) At the end you MUST prove 100% coverage with an atomic migration report + diff summary.

---

## Antigravity file format (use this)
Create rule files as markdown with YAML frontmatter `trigger:`.

Examples:
### Always On
---
trigger: always_on
---
(bullets)

### Glob-based
---
trigger: glob
globs: "app/GraphQL/**/*.php, graphql/**/*.graphql"
---
(bullets)

(Use comma-separated patterns in a single string; quote it.) :contentReference[oaicite:6]{index=6}

### Model Decision
---
trigger: model_decision
description: "Activate when modifying public GraphQL or REST contracts (schemas, OpenAPI, breaking changes, versioning). Keywords: contract, schema, OpenAPI, breaking change, deprecate."
---
(bullets + short checklist + examples)

### Manual (only if truly necessary)
---
trigger: manual
description: "Manually activate via @mention when doing <X>."
---
(bullets)

(If manual rules are really “saved prompts”, prefer workflows instead.) :contentReference[oaicite:7]{index=7}

---

## Step-by-step execution plan (do this in order)

### 1) Inventory EVERYTHING in `.cursor/rules/**`
- Enumerate and read every file (including `.mdc`).
- For each file extract:
  - Metadata (e.g., `alwaysApply`, `globs`, `description`, etc.)
  - Body text/instructions

Build an INTERNAL checklist of atomic rule points:
- Split each rule body into minimal atomic instruction items (one requirement per item).
- Number them per source file (e.g., `fileA#01`, `fileA#02`, ...).

### 2) Classify each atomic instruction into Antigravity destinations

#### A) `./.agent/rules/AGENTS.md` (ALWAYS_ON, 20–40 lines total)
Put ONLY the top universal constraints that must apply to ANY request.
Typically sourced from Cursor rules with intent equivalent to `alwaysApply: true` AND not path-specific.
Also include a SHORT “policy” section:
- Keep always_on small; prefer model_decision/glob/workflows for everything else.
- Point to `.agent/rules/` and `.agent/workflows/`.

Create `./.agent/rules/AGENTS.md` using:
---
trigger: always_on
---
Then 5–12 bullets max + 1–2 lines about where to find scoped rules/workflows.

DO NOT paste long rule bodies here.

#### B) `./.agent/rules/*.md` with `trigger: glob`
If Cursor rule has `globs` or is clearly path/file-type scoped:
- Create one Antigravity rule file per Cursor rule file (1:1 where possible).
- Convert Cursor `globs` to Antigravity `globs` string 1:1 (comma-separated).
- Keep content actionable (bullets/checklists). Do not drop meaning.

#### C) `./.agent/rules/*.md` with `trigger: model_decision`
If Cursor rule is “Apply intelligently / activate when …”:
- `alwaysApply: false` + meaningful `description`, OR clearly semantic (“when doing migrations”, “public API changes”, “release process”, etc.)
- Create a model_decision rule per Cursor rule file (or split if one file contains multiple distinct activation intents).

Each `model_decision` rule MUST have an excellent `description`:
- When to use (explicit)
- Trigger keywords
- Typical tasks it applies to
This is critical to make Antigravity load the right guidance automatically. :contentReference[oaicite:8]{index=8}

Inside the body include:
- ## When to use (triggers)
- ## Do this (step-by-step)
- ## Examples (1–3)
- ## Pitfalls (only if present in source)

#### D) `./.agent/workflows/*.md` for repeatable procedures (slash commands)
If a Cursor rule is actually a repeatable “do X now” procedure (test generation, release notes, audit checklist, etc.), migrate it into a workflow:
- Create `./.agent/workflows/<kebab-name>.md`
- Workflow content can be a bullet list prompt (keep close to Cursor wording).
Workflows are invoked with `/` in Antigravity. :contentReference[oaicite:9]{index=9}

#### E) Mixed rules (both globs + semantic activation)
If a Cursor rule has BOTH:
- Put strict file/path constraints into a `trigger: glob` rule
- Put deeper procedure/playbook into a `trigger: model_decision` rule or workflow
Cross-reference with one line in each file.

### 3) Cursor-specific instructions (must not lose intent)
When you see Cursor-only features/instructions:
- Try to translate intent into Antigravity equivalents (rules/workflows).
- If no equivalent: keep verbatim + annotate:
  `<!-- Cursor-specific: no Antigravity equivalent found -->`
Also record it in the final report as “Adapted” vs “Verbatim”.

### 4) MCP migration (Cursor MCP → Antigravity)
If `.cursor/mcp.json` exists:
- DO NOT delete it.
- Create a repo file: `./antigravity.mcp_config.json` that mirrors Cursor MCP config structure (`mcpServers`).
- If needed, normalize obvious differences conservatively (do not guess new fields).
- Add a short note in `AGENTS.md` telling developers to copy this into Antigravity’s MCP raw config (`mcp_config.json`). :contentReference[oaicite:10]{index=10}

### 5) Create folders/files
- Ensure `./.agent/rules/` and `./.agent/workflows/` exist.
- Generate all rule/workflow files according to classification.
- Prefer stable filenames matching Cursor rule file names (kebab-case OK).

### 6) Validation (MANDATORY): prove nothing was missed
You MUST output a migration report with:

#### 6a) Atomic inventory table (one row per atomic instruction)
| Source File | Rule ID | Original Text (first 60 chars) | Destination | Status |
|---|---:|---|---|---|
Statuses:
- ✅ Migrated verbatim / minimal condense
- 🔄 Adapted (Cursor-specific → Antigravity equivalent)
- ⚠️ Partially migrated (explain what was lost — should be rare)
- ❌ Not migrated (must not happen without extremely strong reason)

#### 6b) Coverage proof
For each source file, show at least one distinctive phrase and where it appears in new files.

#### 6c) Summary stats
- Total source files
- Total atomic rules
- Verbatim / Adapted / Partial / Not migrated (must be 0)

#### 6d) Diff summary
Show:
- `git diff --stat`
- `git status --short`
List all new/updated files.

---

## Anti-patterns (forbidden)
- Do NOT collapse multiple distinct Cursor requirements into one bullet without tracking each atomic rule’s mapping.
- Do NOT skip “meta rules” (rules about updating rules). Adapt them.
- Do NOT assume “obvious” = can be dropped.
- Do NOT finish without 100% table coverage.

---

## Start now
Use a todo tasklist, work methodically, and do not mark complete until the report shows 100% coverage.
