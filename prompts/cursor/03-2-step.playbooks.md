You are Claude Code running inside this repo.

Goal: create a small set of high-impact playbooks that standardize how we execute work with the existing Core-5 subagents.

Inputs: ./CLAUDE.md, .claude/rules/**, .claude/skills/**, ./docs/*.md, tech-lead checklist.

Deliverables:
- Create .claude/playbooks/INDEX.md
- Create exactly these playbooks (one file each):
  - .claude/playbooks/feature.md
  - .claude/playbooks/bugfix.md
  - .claude/playbooks/refactor.md
  - .claude/playbooks/api-contract-change.md
  - .claude/playbooks/migration.md
  - .claude/playbooks/integration-new-dependency.md

Each playbook must include:
- When to use (trigger keywords)
- Required subagent sequence (Core-5) and handoffs
- Step-by-step checklist
- Context7/MCP rule: if external libs/APIs and no local examples -> MUST use context7, do not guess
- Definition of Done (tests + product validation when applicable)
- Common pitfalls (only if supported by docs/checklist)

Constraints:
- No hallucinations: derive repo-specific steps from docs/rules/checklist.
- Keep each playbook 40–120 lines max.
- Don’t duplicate rules; link to relevant docs/rules/skills.

Finally: update .claude/agents/README.md to reference these playbooks and show 2–3 invocation examples.
