# Spec Interview Command

Read and interview the user about: $ARGUMENTS

---

## Step 1: Read Input

Parse `$ARGUMENTS`:
- If contains `/` or ends with `.md` → Read file with `Read` tool
- Otherwise → Treat as instructions text

---

## Step 2: Interview Loop (CRITICAL)

**YOU MUST use the `AskUserQuestion` tool repeatedly** to interview the user.

Continue interviewing until ALL areas are thoroughly covered. Do NOT stop after 1-2 rounds.

### Interview Rules

1. **Use AskUserQuestion tool** for EVERY question round
2. **Ask 2-4 questions per round** (use multi-question format)
3. **Questions must be NON-OBVIOUS** — don't ask things with obvious answers
4. **Be very in-depth** — dig into details, edge cases, implications
5. **Build on previous answers** — reference what user said before
6. **Continue until complete** — typically 5-10+ rounds minimum

### Question Categories (cycle through ALL of these)

**Technical Implementation:**
- Component/module architecture decisions
- Data models, relationships, storage strategy
- API contracts and integration points
- State management approach
- Caching, optimization, performance targets

**UI/UX Decisions:**
- User flows and interaction patterns
- Feedback mechanisms (loading, success, error states)
- Empty states and first-time experience
- Responsive/mobile behavior specifics
- Accessibility requirements

**Edge Cases & Error Handling:**
- What happens when external service X fails?
- How to handle partial success/failure?
- Race conditions and concurrent operations
- Validation rules and error messages
- Retry logic, timeouts, fallback behavior

**Tradeoffs & Concerns:**
- Performance vs. complexity tradeoffs
- Consistency vs. availability choices
- Build vs. buy vs. integrate decisions
- Technical debt implications
- Security and privacy considerations

**Non-Obvious Questions (IMPORTANT):**
- What is explicitly OUT OF SCOPE?
- What existing patterns should this follow? Break?
- What's the migration path from current state?
- What metrics define success/failure?
- What assumptions are we making?
- What would make this feature fail in production?
- Who are the edge-case users we're ignoring?
- What's the rollback plan if this goes wrong?

### DON'T Ask These (Too Obvious)

- "Should we use TypeScript?" — already in stack
- "Should it be responsive?" — default expectation
- "Should we handle errors?" — always yes
- "Should we add loading states?" — always yes
- "Should it work on mobile?" — default expectation

### DO Ask These (Good Examples)

- "If the GitHub API rate-limits us mid-sync, should we queue remaining items or fail the whole batch?"
- "When a user has 500+ servers, should we paginate, virtualize, or limit the display?"
- "If this feature conflicts with the existing X behavior, which takes priority?"
- "Should optimistic updates revert silently or show an error toast on failure?"

---

## Step 3: Completion Check

Before finishing, verify ALL areas covered:

- [ ] Core functionality defined
- [ ] All UI states (success, error, loading, empty)
- [ ] Edge cases identified and handled
- [ ] Technical approach decided
- [ ] Integration points clarified
- [ ] Scope boundaries explicit (what's IN and OUT)
- [ ] No critical unknowns remain

Ask: **"Any other areas to discuss before I write the final spec?"**

---

## Step 4: Write Spec

Ask user where to save, then write:

```markdown
# Feature: [Name]

## Overview
[2-3 sentence description]

## Requirements

### Functional
- [Requirement with acceptance criteria]

### Non-Functional
- [Performance/security/scalability targets]

## Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| [Area] | [What we chose] | [Why] |

## UI/UX Specifications

### States
| State | Behavior |
|-------|----------|
| Loading | [Description] |
| Empty | [Description] |
| Error | [Description] |
| Success | [Description] |

### User Flows
[Key interaction flows]

## Edge Cases

| Scenario | Behavior | Rationale |
|----------|----------|-----------|
| [Case] | [How handled] | [Why] |

## Out of Scope
- [Explicitly excluded item and why]

## Open Questions
- [Any remaining unknowns]

## Implementation Notes
[Technical details, dependencies, migration steps]
```
