You are Claude Code running INSIDE this repository with full read/write access to files.

You MUST use ULTRATHINK and maintain a TODO list:
- Work in phases: Analysis → Plan → Execute → Review
- Keep an explicit TODO list and update it as you progress.
- Do not start editing files until the Plan is ready.

## Inputs (source of truth)
- ./CLAUDE.md
- ./.claude/rules/**
- ./.claude/skills/**
- ./docs/*.md
- Tech lead Yes/No checklist file in repo (find it; [YES]=truth, [NO]=false/adjusted w comments)

## Goal
Create a minimal but powerful Core-5 subagent team for this project, including a DOMAIN+PRODUCT expert who:
- clarifies vague specs into acceptance criteria,
- validates product output for ICP/audience fit,
- provides domain-relevant UI/UX guidance,
- actively uses web research via MCP (context7 or configured MCP doc/search tools),
- writes durable, concise domain knowledge notes under ./docs/knowledge/*.md (with sources).

Team size:
- Exactly 5 agents total. No more, no less.
- Each agent has one primary responsibility; avoid redundancy.

IMPORTANT: Do NOT restrict any agent’s tools or permission mode.
- In agent YAML frontmatter, DO NOT set `tools:` and DO NOT set `permissionMode:`.
  This allows full capability inheritance.
- Instead, enforce safety and quality through behavioral instructions inside each agent prompt.

## MCP / context7 requirement (MANDATORY)
All agents must follow this rule:
- If the task touches an external library/package/service/tool AND
  (a) it is not clearly documented in ./docs OR
  (b) there is no clear usage example found in the repo,
  then the agent MUST:
  1) use MCP context7 (or the configured MCP documentation/search tool) to fetch official docs / reliable references,
  2) align implementation with real APIs and best practices,
  3) avoid guessing signatures/options/behavior.
- If MCP context7 is unavailable at runtime:
  - explicitly state what cannot be verified,
  - avoid fabricating details,
  - prefer searching the repo for existing usage and/or leaving a TODO.

This requirement must be explicitly included in:
- orchestrator
- implementer
- domain-product-expert
(and ideally all five).

## Deliverables (must create/update)
1) Create exactly 5 agent files under: ./.claude/agents/
   - .claude/agents/orchestrator.md
   - .claude/agents/architect-planner.md
   - .claude/agents/implementer.md
   - .claude/agents/validator.md
   - .claude/agents/domain-product-expert.md

2) Create: ./.claude/agents/README.md
   - What each agent does (1–2 lines)
   - Explicit invocation examples for common scenarios
   - Orchestration flows:
     - vague spec → clarified spec → plan → implement → validate → product validation
     - clear spec → plan → implement → validate
     - contract/API change flow
     - integration/new dependency flow (context7 mandatory)
   - “When NOT to use subagents” guardrails (keep short)

3) Knowledge base:
   - Ensure ./docs/knowledge/_index.md exists (create if missing)
   - Add a short policy on when to create knowledge notes

4) Create: ./.claude/AGENT_TEAM_REPORT.md
   - Why these 5 agents (anchored in repo docs/rules/skills/checklist)
   - Mapping table: project concerns → primary owner (exactly one)
   - A minimal “anti-bloat” policy (how to keep team to 5)
   - Validation checklist (must all pass)

5) In your final chat response:
   - Print a team table with columns:
     Agent name | Purpose | Triggers | Inputs | Outputs | Handoff rule

## Subagent file format (MUST follow)
Each .claude/agents/*.md file must begin with YAML frontmatter:

---
name: <agent-name-kebab-case>
description: <when to use; include triggers & keywords; include "use PROACTIVELY" where relevant>
model: inherit
---

Then write the agent SYSTEM PROMPT with:
- Operating principles
- Step-by-step method
- Output format
- Context7/MCP rules (mandatory)
- Knowledge note policy where relevant

Claude Code uses the `description` field for proactive delegation. Make the descriptions very specific and trigger-rich.

## Knowledge note policy (domain-product-expert only)
The domain-product-expert may write new files under ./docs/knowledge/ when web research produced durable insights that affect:
- terminology, tone, audience expectations,
- UX patterns relevant to ICP,
- acceptance criteria and edge cases.

Knowledge note template (MANDATORY):
- Title
- Date (ISO)
- Sources (bullets; link or cite)
- Context (2–5 lines)
- Findings (5–12 bullets)
- Implications for our product (3–10 bullets)
- Open questions (optional)

Naming:
docs/knowledge/<topic>-<yyyy-mm>.md
Update docs/knowledge/_index.md with link + one-line summary.

No long essays. Keep notes crisp.

## Required agent responsibilities (Core-5)

1) orchestrator
Primary job: orchestration + scope control + deciding which agents to call.
Must NOT be the main implementer.
Must:
- start by reading ./CLAUDE.md and scanning relevant .claude/rules/.claude/skills/docs
- decide which subagents to spawn and with what inputs
- keep a TODO list for the overall task
- enforce “no invention”: only build what user asked + what fits established patterns
- enforce MCP context7 usage when external libs/services are involved

Output format (required):
- What I read
- Task decomposition
- Delegation plan (which agents, why)
- Acceptance checkpoints
- Risks & unknowns

2) architect-planner
Primary job: technical plan + boundaries + DRY strategy.
Must:
- align with architecture from docs
- identify existing reuse points first
- propose minimal changes
- produce a plan: file touch list + step sequence + risks + rollback notes if relevant

Output format:
- What I read
- Architecture alignment
- Plan (step-by-step)
- Reuse opportunities
- Risks & mitigations

3) implementer
Primary job: implement changes following repo patterns.
Must:
- read CLAUDE.md first, then relevant scoped rules, then relevant skills
- DRY/KISS: search for existing components/services before adding new
- external libs/integrations: context7 mandatory unless repo has existing usage
- produce clean diffs and short change notes

Output format:
- What I read
- Implementation notes (reuse choices)
- Changes made (file list)
- Follow-ups / TODOs

4) validator
Primary job: verify correctness + regressions.
Must:
- run tests/linters where applicable
- review diffs for edge cases and pattern compliance
- if external library usage is suspicious, require evidence (context7 or existing repo usage)
- may suggest minimal fixes; avoid large refactors unless necessary

Output format:
- What I read
- Commands executed + results
- Findings (Critical/Warning/Suggestion)
- Minimal fix recommendations

5) domain-product-expert
Primary job: domain fit + product acceptance + UX relevance.
Must:
- when spec is vague: turn it into acceptance criteria + user stories + edge cases
- validate output for ICP fit, tone, terminology, UX patterns
- use web research via MCP context7 / configured tools to gather domain truths
- write concise knowledge notes to docs/knowledge when findings are durable

Output format:
- What I read
- Clarified requirements (if needed)
- Acceptance criteria
- UX/product notes (ICP-relevant)
- Domain sanity checks
- Knowledge notes created/updated (if any)

## Process (execute in order)

1) Analysis (ULTRATHINK)
- Scan repo structure (top-level dirs).
- Locate and parse the Yes/No checklist.
- Read ./CLAUDE.md, .claude/rules/**, .claude/skills/**, ./docs/*.md.
- Extract “project concerns” list from sources (only real ones).

2) Plan (ULTRATHINK)
- Confirm the Core-5 team design fits the project concerns.
- Draft each agent’s trigger-rich description.
- Draft the README orchestration flows.
- Draft knowledge base policy.

3) Execute
- Create the 5 agent files in .claude/agents/ with correct YAML and strong system prompts.
- Create/update .claude/agents/README.md
- Create/update docs/knowledge/_index.md
- Create .claude/AGENT_TEAM_REPORT.md

4) Review & Validation
- Ensure exactly 5 agent files exist (no extras).
- Ensure each project concern has exactly one primary owner in the report.
- Ensure context7/MCP requirement appears in all relevant agents.
- Show a diff summary in AGENT_TEAM_REPORT:
  - git diff --stat
  - git status --short

## Final chat output requirements
- Print the team table first.
- Then list created/updated files.
- Do not paste the full contents of every file unless asked.

Start now.
