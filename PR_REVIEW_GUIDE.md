# PR Review Workflow

## Quick Start

```bash
# Review current branch changes
/review-pr

# Re-review after author addresses feedback
/re-review-pr https://github.com/newt-foundation/newton-sdk/pull/123
```

## What Gets Reviewed

Blockers — type safety violations, runtime errors, missing input validation, broken viem patterns, leaked secrets, XSS vectors, bundle size regressions.

Concerns — error handling (missing catches, swallowed errors), missing tests for new functionality, unnecessary dependencies, breaking API changes without documentation, `any` types.

Minor notes — naming clarity, import organization, JSDoc completeness.

## Comment Style Guidelines

### General Principles

- Be **direct and concise** - state the issue immediately without preamble
- No bold category prefixes like "**Critical:**" or "**Warning:**"
- No formal headers like "Suggested fix:" - just provide the fix
- No markdown headings (`##`, `###`) in the review body — use `**bold**` sparingly
- Reference existing patterns in the codebase when applicable
- Use conversational directives: "Let's not do this way" or "something like below can work"
- Use "ditto:" for repeated issues in different files
- Use "nit:" for minor/cosmetic issues
- For follow-up reviews: reference previous comments casually ("still not addressed from previous review")
- Keep the review body focused on what's wrong, not what's been fixed
- Don't comment on things that are correctly done
- Bold direct asks to the author: **Document this in the PR description.**
- Escalate clearly when blocking: "A blocker for this PR considering [reason]."

### Voice and Framing

- **Use "we/us" not "you"** — collaborative framing. "Let's do X", "if we update", "we should"
- **Direct imperatives for clear issues** — "DO NOT", "Shouldn't do this", "Let's do X"
- **Prefix labels for non-blocking items:**
  - `Opinion:` — subjective feedback, architectural preferences
  - `FYI:` — informational, good to know
  - `Suggestion:` — code improvement with example
  - `nit:` — minor/cosmetic
- **No praise sections** — review body is for issues only
- **No severity grouping** — flat numbered list with prefix labels
- **Short punchy comments preferred**
- **End with clear merge criteria** — "LGTM once above are addressed."

### Formatting Rules — DO NOT USE:

- Emoji headers or category prefixes
- Bold-label patterns like "**Problem:**", "**Risk:**", "**Fix:**"
- Markdown headings (`##`, `###`) in the review body
- Severity grouping headers
- Long code blocks in the review body (code suggestions go in inline comments)
- "What is working well" or praise sections
- Using "you" — use "we/us/let's"

### Inline Comment Format

Good:

```text
The `fetchWithCache` stale time defaults to 30s but this call caches policy data that changes infrequently. Let's bump to 5 minutes or make it configurable per call.
```

```text
Let's not do this way. Use viem's `getAddress()` for checksumming. We already do this in `src/utils/intent.ts`.
```

### Code Suggestions

Only include code examples when the fix is non-obvious. Introduce casually: "something like below can work."

### Cross-References

Always use GitHub permalinks with commit SHA and line range:

- "We already do this in https://github.com/newt-foundation/newton-sdk/blob/abc123/src/utils/intent.ts#L42-L50"

### TypeScript/SDK-Specific Concerns

When reviewing TypeScript SDK code, additionally check:

- **Type safety**: No unnecessary `any`, proper generic constraints, discriminated unions
- **Tree-shaking**: Named exports, no barrel file side effects, no default exports
- **Bundle impact**: New dependencies justified? Could be done without?
- **API surface**: Breaking changes documented? Backward compatible?
- **viem patterns**: Following viem conventions for client extensions, contract interactions?
- **Error handling**: SDK errors with proper codes? Consumer-facing errors clear?
