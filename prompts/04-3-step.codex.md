# Cursor → Codex Migration (AGENTS.md hierarchy + MCP + 100% coverage)

You are Codex (CLI or IDE extension) running INSIDE this repository with full read/write access to files.

## Goal
Migrate ALL Cursor rules from `.cursor/rules/**` into a Codex-native setup using `AGENTS.md` layering, without losing anything, so the team can use Cursor or Codex on the same project.

Codex instruction system facts you must respect:
- Codex auto-loads `AGENTS.md` (or `AGENTS.override.md`) from repo root down to the current working directory, at most ONE file per directory level, concatenated in order. :contentReference[oaicite:3]{index=3}
- There is a combined size cap (`project_doc_max_bytes`, 32 KiB default). Keep the root short and split into nested directories when needed. :contentReference[oaicite:4]{index=4}

---

## Hard requirements
- Do NOT delete or modify anything under `.cursor/rules/**` (Cursor stays supported).
- Create/Update the repo-root `AGENTS.md` that is VERY short: 20–40 lines total.
  - Include only the most universally important rules that should matter for ANY request.
  - Prefer Cursor rules with `alwaysApply: true` (or clearly “always on” intent).
  - Do NOT paste long rule bodies into root `AGENTS.md`.
- Migrate EVERYTHING from `.cursor/rules/**` into Codex artifacts such that every atomic instruction appears somewhere:
  - root `AGENTS.md`, OR
  - nested `AGENTS.md` / `AGENTS.override.md` placed in appropriate directories, OR
  - “playbooks” (markdown files) referenced from `AGENTS.md` and designed to be loaded on-demand by Codex.
- Do not invent new policies. Only reorganize/adapt what already exists in `.cursor/rules/**`.
- Keep wording close to source; condense only when necessary, but do not drop meaning.

---

## Deliverables to create in the repo
1) `./AGENTS.md` (root, 20–40 lines)
2) A set of nested `AGENTS.md` (and rarely `AGENTS.override.md`) inside repo directories to emulate Cursor glob-scoped rules.
3) `./.codex/playbooks/` containing semantic “apply intelligently” rules (thin index + per-topic playbooks).
4) `./.codex/MIGRATION_REPORT.md` with atomic-level mapping + validation proofs.
5) If `.cursor/mcp.json` exists:
   - Create `./.codex/mcp_config.toml.snippet` that the user can paste into `~/.codex/config.toml` (Codex MCP lives there). :contentReference[oaicite:5]{index=5}
   - Do NOT remove `.cursor/mcp.json`.

---

## Step-by-step plan (execute in this order)

### 1) Inventory Cursor rules (MANDATORY)
- Enumerate and read every file in `.cursor/rules/**` (including `.mdc`).
- For each rule file, extract:
  - metadata fields (e.g., `alwaysApply`, `globs`, `description`, etc.)
  - the rule body (instructions)
- Build an internal checklist of ATOMIC rule points so nothing is missed:
  - Split each rule file body into numbered atomic instructions (one requirement per item).
  - Example IDs: `rules-file-name#01`, `rules-file-name#02`, …

### 2) Classification logic: where each atomic instruction goes

#### A) Root `AGENTS.md` ONLY for “always on + universal”
Put into `./AGENTS.md` only if:
- The instruction is truly global and always-on (typically from `alwaysApply: true`),
- AND it matters for almost any change in the repo.
Keep it short. Max 40 lines total.

Root AGENTS.md must also include a tiny “smart loading policy” section (2–4 lines):
- Keep root small.
- More specific rules live in nested AGENTS.md and playbooks.
- Before acting, Codex should load relevant playbooks only if the task matches triggers.

#### B) Cursor `globs` → nested `AGENTS.md` (path/directory-scoped)
Codex does not support automatic per-glob rules; emulate them by placing AGENTS.md into directories.

Algorithm:
- For each Cursor rule that contains `globs`, compute the “static prefix directory” for each glob:
  - Take the path up to (but not including) the first wildcard segment (`*`, `**`, `?`, `{}`, `[]`).
  - Example: `app/GraphQL/**/*.php` → directory `app/GraphQL/`
  - Example: `graphql/**/*.graphql` → directory `graphql/`
- Create or update `AGENTS.md` in each computed directory.
- At the top of each nested AGENTS.md, include a short header:
  - “Applies to: <original glob(s)>”
- Copy the relevant rules into that nested AGENTS.md (bullets, short sections).
- If one Cursor rule has multiple globs in different directories:
  - Split its content into multiple nested AGENTS.md files (one per directory) while preserving coverage.
- Avoid `AGENTS.override.md` unless you must override parent guidance; prefer normal `AGENTS.md`.

#### C) Cursor “Apply intelligently” → Playbooks + on-demand loading
If a Cursor rule looks like:
- `alwaysApply: false` AND has a semantic `description` like “Activate when …”
- OR it’s clearly procedural/conditional (public API contracts, migrations, release flows, incident debugging, etc.)
Then:
- Create a playbook: `./.codex/playbooks/<kebab-name>.md`
- Put:
  - ## When to use (explicit triggers + keywords from description)
  - ## Instructions (step-by-step)
  - ## Examples (1–3)
  - ## Gotchas/Anti-patterns (only if present in source)
- Add an entry to `./.codex/playbooks/INDEX.md`:
  - playbook name
  - triggers/keywords
  - file path
- In root `AGENTS.md` include a very short instruction:
  - “If the task matches triggers from .codex/playbooks/INDEX.md, open and follow the relevant playbook(s) before making changes.”

This gives you “semantic activation” without bloating the always-on chain.

#### D) Cursor-specific instructions (CRITICAL)
If a rule is Cursor-specific and doesn’t map cleanly to Codex:
- Attempt adaptation to a tool-agnostic instruction.
- If adaptation is not possible:
  - Keep verbatim.
  - Add a note: `<!-- Cursor-specific: no direct Codex equivalent -->`
- Log these as “Adapted” or “Cursor-specific kept” in the migration report.

### 3) MCP migration (Cursor MCP → Codex MCP)
If `.cursor/mcp.json` exists:
- Read it and generate `./.codex/mcp_config.toml.snippet` for Codex’s `~/.codex/config.toml`. :contentReference[oaicite:6]{index=6}
Mapping rules:
- For URL/HTTP MCP servers:
  - Use:
    [mcp_servers.<name>]
    url = "https://..."
  - If Cursor has auth tokens/headers, map conservatively using:
    bearer_token_env_var, http_headers, env_http_headers (only if present/derivable).
- For stdio servers:
  - Use:
    [mcp_servers.<name>]
    command = "..."
    args = ["..."]
    [mcp_servers.<name>.env]
    KEY = "VALUE"
Do not guess extra fields. Only convert what exists.

Also add a short note in `./.codex/mcp_config.toml.snippet` explaining “paste into ~/.codex/config.toml under [mcp_servers.*]”. :contentReference[oaicite:7]{index=7}

### 4) Create files/folders
- Ensure `./.codex/playbooks/` exists.
- Create/update all nested AGENTS.md files per the glob-prefix algorithm.
- Create root AGENTS.md (20–40 lines).

### 5) Validation: prove NOTHING was missed (MANDATORY)
Create `./.codex/MIGRATION_REPORT.md` with:

#### 5a) Atomic inventory table (one row per atomic instruction)
| Source File | Rule ID | Original Text (first 60 chars) | Migrated To | Status |
Statuses:
- ✅ Migrated verbatim / minimal condense
- 🔄 Adapted (Cursor-specific → Codex-compatible)
- ⚠️ Partially migrated (explain what was lost; should be rare)
- ❌ Not migrated (must be 0)

#### 5b) Cross-reference validation
For each source file, pick distinctive phrases and show they exist in:
- AGENTS.md / nested AGENTS.md / playbooks (exact file paths).

#### 5c) Summary stats
- Total source files: X
- Total atomic rules: Y
- Verbatim: A
- Adapted: B
- Partial: C
- Not migrated: D (must be 0)

#### 5d) Diff summary
Show:
- `git diff --stat`
- `git status --short`
List all created/updated files.

---

## Anti-patterns to AVOID
- Do NOT “merge away” distinct rules without tracking each atomic item’s destination.
- Do NOT skip “meta” rules (rules about updating rules). Adapt them.
- Do NOT assume a rule is obvious; if it’s written, migrate it.
- Do NOT finish without 100% coverage in the validation table.

---

## Execution order (do this now)
1) Inventory `.cursor/rules/**` and build atomic checklist.
2) If exists, process `.cursor/mcp.json` → `.codex/mcp_config.toml.snippet`.
3) Generate nested AGENTS.md files for glob-scoped rules.
4) Generate playbooks + INDEX for semantic “activate when …” rules.
5) Create the short root `AGENTS.md`.
6) Produce `.codex/MIGRATION_REPORT.md` proving 100% coverage + diff summary.

Do not mark complete until the validation table shows 100% coverage and Not migrated = 0.
