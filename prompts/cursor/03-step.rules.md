You are Claude Code running INSIDE this repository with full read/write access to files.

Inputs (source of truth):
- The finalized ./docs/*.md documentation.
- The reviewed Yes/No checklist from the tech lead, where:
  - [YES] items are confirmed truths.
  - [NO] items indicate incorrect or adjusted assumptions (respect any inline comments there).

Goal:
Create a SMALL, HIGH-IMPACT Claude Code configuration that encodes how AI should work in THIS codebase, without bloating context.
You must create files that are ready to commit to the repo.

Output artifacts (must be created/updated):
1) ./CLAUDE.md  (VERY short: 20–40 lines total)
2) ./.claude/rules/*.md  (modular rules; mostly path-scoped via YAML `paths:`)
3) ./.claude/skills/*/SKILL.md  (semantic “apply intelligently” rules as Skills)
4) ./.claude/RULES_GENERATION_REPORT.md  (traceability + validation)

Optional (only if repo already has MCP config or docs/checklist mention it):
5) ./.mcp.json  (project-scoped MCP servers)

Global constraints:
- Language: English only (rules, comments, docs, commit messages suggested by AI).
- No hallucinations: only use facts from ./docs and confirmed [YES] checklist items.
- Be concise and non-redundant; these files consume context.
- Encode real architecture + boundaries (follow ./docs/01-architecture.md).
- Enforce DRY: prefer existing components/services/hooks/utils from ./docs/03-reuse-catalog.md.
- Avoid magic numbers: use constants/config/env where appropriate.
- Use clear, descriptive names; avoid cryptic naming.
- Do NOT invent new features/APIs beyond what the user requests and what fits existing patterns.
- Do NOT restate all documentation. Prefer referencing specific ./docs files in short bullets.

SMART-LOADING POLICY (critical):
- Keep always-on content tiny:
  - ./CLAUDE.md must stay 20–40 lines.
  - Only 1–2 unscoped .claude/rules/*.md files are allowed, and they must be short.
- Everything else MUST be either:
  - path-scoped rules via YAML `paths:` in .claude/rules/*.md, OR
  - Skills under .claude/skills/ (semantic activation via description metadata).
- Prefer Skills for “Activate when …” situations (contracts, migrations, releases, incidents, security review).

Claude Code rule format requirements:
- .claude/rules files are Markdown.
- For path-scoped rules, use YAML frontmatter with `paths`.
- To avoid YAML parsing issues, ALWAYS use a YAML list and ALWAYS quote each pattern, e.g.:

  ---
  paths:
    - "src/api/**/*.ts"
    - "{src,lib}/**/*.ts"
  ---

- Rules without `paths` apply globally, so keep them rare and short.

Claude Skills format requirements:
- Each Skill is a directory: .claude/skills/<kebab-name>/SKILL.md
- SKILL.md must start with YAML frontmatter:
  ---
  name: <kebab-case>
  description: <very specific: what it does + when to use + trigger keywords>
  ---
- Keep each Skill focused; 1 capability per Skill.
- The description is what drives discovery—include “when to use” + keywords users will mention.

MCP / context7 requirement (MANDATORY in content, optional in config):
- At least one always-on rule AND the meta/governance rule must instruct:
  - When using external libraries/packages/tools not documented in ./docs AND no examples exist in repo:
    - use the configured MCP documentation/search tool (e.g., context7) to consult official docs / reliable references;
    - do NOT guess function signatures/options/behavior.
  - If MCP isn’t available at runtime, explicitly state missing info and avoid fabricating details.
- If you find existing MCP config (e.g., .cursor/mcp.json or docs mention MCP), generate .mcp.json in repo root:
  - Preserve servers; do not delete the existing config.
  - For URL-based servers, include `"type":"http"` and `"url"`.
  - Support env expansion patterns like ${VAR} and ${VAR:-default} when present.
  - If you cannot safely infer a field, leave it out and document the omission in the report.

REQUIRED rule set design (aim ~8–15 total files across rules+skills, not counting report):
A) ./CLAUDE.md (20–40 lines total)
- 1 short title line
- 6–12 bullets maximum:
  - “Use ./docs as primary context; follow architecture in ./docs/01-architecture.md”
  - “Prefer reuse catalog ./docs/03-reuse-catalog.md (DRY)”
  - “English-only outputs”
  - “No magic numbers”
  - “Use MCP/context7 for external libs when no local examples exist; don’t guess”
- 2–4 lines: “Where to find scoped rules and skills” (point to .claude/rules and .claude/skills)
- Do NOT import large files via @… in CLAUDE.md.

B) .claude/rules (modular rules; mostly path-scoped)
You MUST create a meta/governance rule:
- File: .claude/rules/00-rules-governance.md
- Unscoped (no paths) but SHORT and high impact.
- It must define:
  - When to update ./docs/*.md and Claude rules after changes that are substantial:
    - new core modules / bounded contexts,
    - changed data flow or cross-cutting concerns,
    - changes to public APIs/contracts,
    - major introduction/removal of external services.
  - What does NOT trigger updates:
    - minor refactors, trivial edits, formatting changes.
  - How to update:
    - minimal diffs, preserve intent, reference exactly what changed and where.
  - Tooling behavior:
    - use MCP/context7 (if available) to check existing patterns before updating docs/rules;
    - NEVER overwrite/delete docs/rules blindly; extend or adjust.

Add 1–2 additional GLOBAL rules (unscoped) only if truly universal, e.g.:
- .claude/rules/01-global-quality.md (short)
Everything else must be SCOPED via `paths:`:
- frontend rules (components/hooks/pages) ONLY if the repo actually has frontend (derive paths from ./docs + tree)
- backend rules (controllers/services/use-cases) ONLY if present
- shared libs/monorepo packages rules if present
- integrations/external services layer rules if present
- tests rules if present

IMPORTANT: For each scoped rule:
- Verify the paths exist in the repo (use ls/find/glob) before writing patterns.
- Keep the rule body short: 6–15 bullets.

C) .claude/skills (semantic “apply intelligently”)
Create 2–5 Skills maximum (only those supported by docs/checklist), examples:
- public-api-contracts (GraphQL/OpenAPI/versioning/breaking changes)
- migrations-versioning (db migrations, backward compatibility, rollout/rollback)
- integrations-changes (3rd-party APIs, auth, retries, idempotency)
- incident-debugging (logs/traces, reproduction, safe mitigation)
- security-review (authz/authn, secrets, injection risks)
Each Skill must:
- Have a very specific description with “when to use” + trigger keywords.
- Include:
  - ## Instructions (step-by-step)
  - ## Examples (1–3)
  - ## Pitfalls (only if derived from docs/checklist)
- Mention MCP/context7 usage when touching unfamiliar external systems.

Process (execute in this exact order):
1) Read ./docs/*.md and the YES/NO checklist. Extract ONLY confirmed constraints and repo-specific facts.
2) Build an internal “atomic constraints list”:
   - each item = one distinct constraint or convention
   - each item must have a source reference:
     - docs file + section heading OR checklist [YES] line
3) Decide the minimal rule set:
   - Which constraints belong in CLAUDE.md (tiny, universal)
   - Which become .claude/rules (path-scoped)
   - Which become Skills (semantic “activate when …”)
4) Create/update the files accordingly.
5) Validation (MANDATORY):
   - Ensure every [YES] checklist item is covered by at least one rule bullet (CLAUDE.md, rules, or skills).
   - Ensure no rule references non-existent paths.
   - Ensure no duplication (same instruction repeated across multiple files).
6) Write .claude/RULES_GENERATION_REPORT.md containing:
   a) File list created/updated
   b) A traceability table (one row per atomic constraint):
      | Constraint ID | Source (docs/checklist) | Implemented In (file + section) |
   c) A coverage summary:
      - YES items total: N
      - YES items covered: N (must be 100%)
      - Rules count: X
      - Skills count: Y
   d) Diff summary:
      - git diff --stat
      - git status --short
   e) Any uncertainty/missing info explicitly noted (no guessing).

Anti-patterns to avoid:
- Do NOT dump docs content into memory files.
- Do NOT create broad “kitchen sink” rules.
- Do NOT add unscoped .claude/rules unless absolutely necessary.
- Do NOT invent tech stack details; derive from docs and repo tree.
- Do NOT finish without 100% coverage of [YES] checklist items.

Begin now: scan the repo, read ./docs and checklist, then generate the Claude Code artifacts.
