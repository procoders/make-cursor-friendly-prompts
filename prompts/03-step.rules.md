You are now configuring Cursor project rules for this repository.

Inputs:
- The finalized ./docs/*.md documentation.
- The reviewed Yes/No checklist from the tech lead, where:
  - [YES] items are confirmed truths.
  - [NO] items indicate incorrect or adjusted assumptions (respect any inline comments there).

Goal:
Generate a SMALL, HIGH-IMPACT set of Cursor project rule files in `.cursor/rules/*.mdc` format.
These files MUST be ready to be created directly in the repo.

Global constraints:
- Language: English only.
- No hallucinations: only use facts from ./docs and confirmed checklist items.
- Rules must be concise and non-redundant; they consume context.
- Encode how AI should work in THIS codebase:
  - Respect the actual architecture and boundaries.
  - Enforce DRY: prefer existing components/services/hooks/utils from ./docs/03-reuse-catalog.md.
  - Avoid magic numbers: use constants/config/env where appropriate.
  - Use clear, descriptive names; avoid cryptic naming.
  - All comments, docs, commit messages suggested by AI MUST be in English.
  - Do NOT invent new features/APIs beyond what the user requests and what fits existing patterns.
- Do NOT restate all documentation. Refer to specific docs files instead.
- Follow Cursor rule best practices from official docs.

Types & structure of rules:
- Each rule is one `.mdc` file with YAML frontmatter and a short markdown body.
- Basic frontmatter format:

  ---
  description: <short 1–2 sentence description>
  globs: <optional glob pattern(s) or leave empty>
  alwaysApply: <true|false>
  ---

- You may leave `globs:` blank for global / agent-requested rules.
- Use `alwaysApply: true` ONLY for a few truly global rules.
- For scoped rules, set `globs` to match REAL paths in this repo.
- Aim for ~8–15 rules total, including the meta-rule.

Special requirements (MANDATORY):

1. Generate a meta-rule:
   - File: `.cursor/rules/00-rules-generator.mdc`
   - `alwaysApply: true`
   - This rule MUST:
     - Explain how and when the AI should:
       - Update ./docs/*.md after completing tasks that introduce significant, fundamental changes:
         - new core modules / bounded contexts,
         - changed data flow or cross-cutting concerns,
         - changes to public APIs or contracts,
         - major introduction/removal of external services.
       - Update or create `.cursor/rules/*.mdc` files when such changes affect architecture, patterns, or constraints.
     - Require that:
       - Only substantial architectural or cross-cutting changes trigger docs/rules updates, NOT minor refactors or trivial edits.
       - Any rule or docs update must be:
         - consistent with existing structure,
         - minimal and precise,
         - clearly referencing what changed and where.
     - Describe how to correctly create `.mdc` files:
       - under `.cursor/rules/`,
       - with valid frontmatter,
       - concise, actionable bullets.
     - Instruct the AI to:
       - use MCP context7 (if available) to:
         - search the codebase and ./docs for existing patterns before modifying rules/docs,
         - inspect existing rules to avoid duplicates or conflicts.
       - NEVER overwrite or delete rules/docs blindly; instead:
         - adjust or extend them in a way that preserves prior intent.

2. Integrate MCP context7 usage into relevant rules:
   - At least one global rule and the meta-rule must explicitly state:
     - When working with external libraries/packages or tools that:
       - are not clearly documented in ./docs,
       - or have no existing usage examples in the repo,
     - the AI should:
       - use MCP context7 (or the configured MCP search/documentation tool) to:
         - fetch official documentation or reliable references,
         - align usage with real APIs and best practices,
       - avoid guessing function signatures, options, or behaviors.
   - If MCP context7 is not available at runtime, the rule should:
     - instruct the AI to clearly mark missing info and avoid fabricating details.

3. Other rules to include (examples, adapt to repo structure from ./docs):
   - 2–4 GLOBAL rules (`alwaysApply: true`, no or blank `globs`):
     - Use ./docs as primary context.
     - Respect architecture from ./docs/01-architecture.md.
     - Enforce DRY, English-only, no magic numbers, no low-quality naming.
     - Encourage MCP context7 usage for lookups and repo-wide search, when available.
   - Several SCOPED rules (`globs` set, `alwaysApply: false`):
     - Frontend (React/Next/Vue) code: how to create components, hooks, pages, API calls consistent with patterns.
     - Backend (Laravel/Nest/Node/PHP) code: how to structure controllers/services/repos/use-cases according to docs.
     - Shared/monorepo libraries: how to use and extend shared modules.
     - Integrations/external services: how to add/change calls consistently with existing integration layer.
     - Tests/specs if present.
   - 1–3 AGENT-REQUESTED style rules (no globs, `alwaysApply: false`):
     - e.g. migrations & versioning, API contract changes.
     - These are loaded when explicitly referenced.

You create files with cursors rules:

- Use EXACTLY this structure:

  File: .cursor/rules/<file-name>.mdc
  ---
  description: <short description>
  globs: <glob pattern(s) or leave empty>
  alwaysApply: <true|false>
  ---
  - Bullet points with concise, actionable instructions.
  - You MAY reference docs, e.g.:
    - "Follow layering rules from ./docs/01-architecture.md."
    - "Before adding a new service, check ./docs/03-reuse-catalog.md."
    - "Use MCP context7 to look up external library docs when no local examples exist."
