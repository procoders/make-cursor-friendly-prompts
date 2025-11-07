You are an expert software architect at Procoders. 
Your task: analyze the ENTIRE repository codebase and generate a concise, accurate documentation set in ./docs as Markdown files. This documentation will be the SINGLE SOURCE OF TRUTH for how this project works and how future code must be written.

Key constraints:
- Base language: English.
- Do NOT hallucinate APIs, tools, or patterns that are not present in the repo.
- If something is unclear or inconsistent, explicitly mark it as "TODO: Clarify with tech lead" instead of guessing.
- Prefer precision over verbosity.
- Structure everything so that it can be later converted into compact .cursor/rules.

Scan the codebase and produce the following files (as Markdown content):

1. ./docs/00-overview.md
   - Short, factual summary of:
     - general about project, it's goal, workflow and audience 
     - tech stack (frontend/backend/frameworks/libs),
     - runtime environments,
     - main entrypoints (e.g. Next.js pages/app, Laravel routes, NestJS modules, CLI, queues, workers),
     - high-level data flow: from request/event → business logic → persistence → external services.
   - Mention if this is monolith, modular, or monorepo (list packages).

2. ./docs/01-architecture.md
   - Describe REAL architecture as implemented:
     - layers (e.g. UI / Application / Domain / Infrastructure),
     - modules/domains (e.g. auth, billing, projects, etc),
     - dependencies rules between them.
   - Include at least:
     - One high-level mermaid diagram of modules and their dependencies.
     - One mermaid diagram for request/response or job processing flow.
   - Only include modules and relations that provably exist in the repo.

3. ./docs/02-conventions.md
   - Coding conventions actually used here:
     - file/folder naming,
     - naming patterns for components, hooks, services, repositories, DTOs, jobs, etc,
     - React/Next.js/Vue component patterns,
     - Laravel/NestJS structure patterns (controllers, services, providers, middleware, etc),
     - how HTTP clients, DB access, caching, queues, validations are organized.
   - Document rule-of-thumb for adding new code that matches existing patterns.

4. ./docs/03-reuse-catalog.md
   - Catalog of existing reusable building blocks:
     - shared UI components,
     - layout components,
     - shared hooks/services/helpers,
     - shared validators,
     - API clients / SDK wrappers,
     - utility modules.
   - For each item, briefly describe WHEN to use it.
   - This file will later be used to enforce DRY and reuse in .cursor/rules.

5. ./docs/04-integrations-and-infra.md
   - List and describe:
     - external services (Stripe, payment gateways, CRMs, storage, mail/SMS, MCP-like tools, etc),
     - configuration patterns (env vars, config files),
     - logging, monitoring, feature flags if used.
   - Include mermaid diagram if relevant (e.g. system ↔ external integrations).

6. ./docs/05-style-and-anti-patterns.md
   - Extract style rules from the existing codebase.
   - Explicitly list PROHIBITED patterns (based on current or desired practice):
     - no magic numbers (use constants/config),
     - no duplicate code where reusable helpers exist,
     - comments and inline docs MUST be in English,
     - no misleading or cryptic variable/function names,
     - no ad-hoc services bypassing existing architecture without clear reason.
   - If the existing code violates these, document that they are legacy exceptions and SHOULD NOT be copied.

7. ./docs/06-versioning-and-evolution.md
   - Describe:
     - how versioning is or SHOULD be handled here (API versions, migrations, changelogs),
     - how to safely extend the system without breaking existing contracts.
   - Keep it short and practical.

General instructions:
- Use bullet points, short paragraphs, and code/examples only when necessary.
- Do not repeat the same information across multiple files.
- Do not propose large refactors; describe what exists, plus minimal "SHOULD" guidance where obvious.
- Everything must be directly supported by code you see, or clearly marked as TODO.
- Output each file as a separate Markdown section in your answer, starting with a level-1 heading equal to the filename.
