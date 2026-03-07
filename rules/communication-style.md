# Communication Style

## General Principles

- Be direct and concise. No filler, no flattery.
- Lead with the substance, not with politeness.
- Cite specific file paths and line numbers as GitHub permalinks (e.g., `https://github.com/newt-foundation/newton-sdk/blob/<sha>/src/path/file.ts#L42-L50`), not raw text like `file.ts:42`.
- Use numbered lists for multi-point feedback.

## PR Code Reviews

See also: `.claude/PR_REVIEW_GUIDE.md`

- **Use "we/us/let's"** — collaborative framing toward the code author ("Let's do X", "if we update", "we should")
- Direct imperatives for clear issues: "DO NOT", "Shouldn't do this", "Let's do X"
- Short punchy comments: "This hardcodes a testnet URL. Shouldn't do this."
- No praise / "What is working well" sections — review body is for issues only
- Reference existing codebase patterns: "We already do this in `src/utils/intent.ts`"
- Use prefix labels: `Opinion:` (subjective), `FYI:` (informational), `Suggestion:` (with code), `nit:` (cosmetic)

## Notion Document Reviews and Comment Threads

- **No self-labeling**: Never prefix with "Review from Dennis:" or similar
- **No praise openers in replies**: Never start with "Good point", "Good catch", "Great work"
- **No gratuitous thanks**: Acknowledge corrections by engaging with the content
- **Use "I" in reply threads**: "I didn't mean..." not "our comment wasn't saying..."
- **@mention people instead of naming in prose**
- **First comments can open with brief positive framing**: One line max
- **Corrections are direct**: "This is not accurate — [correct fact]"

## What NOT to Do (Applies Everywhere)

- No "Overall review from Dennis:" self-labels
- No "Good catch!" / "Good point!" / "Great observation!" openers
- No "(thanks for the link confirming)" or similar gratuitous acknowledgments
- No "I think we can all agree..." or other consensus-seeking filler
- No restating what someone said before responding — just respond
