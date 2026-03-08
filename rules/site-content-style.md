# Documentation Content Style Guide (`site/`)

Rules for writing and editing content in the `site/` documentation directory.

## Voice & Tone

- **Active voice**, second person ("you"), present tense
- Direct and concise — no filler words
- Technically precise — no marketing language in developer docs
- Confident but not arrogant — state facts, not opinions

### Do

- "You deploy a policy using the CLI."
- "The SDK extends viem's `PublicClient` with Newton-specific actions."
- "Operators evaluate the task and return an attestation."

### Don't

- "We're excited to announce..." (marketing)
- "Simply call the function..." ("simply" undermines complexity)
- "It should work..." (vague — be definitive)

## Audience Tiers

| Tier | Who | Assumes |
|------|-----|---------|
| **Beginner** | New to blockchain/web3 | Explain EVM, transactions, signing |
| **Intermediate** | Web3 familiar | Knows Solidity, viem, wallets |
| **Advanced** | Protocol engineers | Understands AVS, BLS, Rego, WASM |

Default to **intermediate** unless the page explicitly targets another tier.

## Structure

1. **Lead with what** — what does this thing do?
2. **Then why** — why would you use it?
3. **Then how** — step-by-step instructions with code

Every concept page should have a **runnable code example within 2 scrolls** of the page top.

## Formatting

- **Short paragraphs** — 2-4 sentences max
- **Tables** for comparisons, parameter lists, and option matrices
- **Code blocks** with language tags (`solidity`, `typescript`, `bash`, `json`, `rego`)
- **Headings** — use `##` for main sections, `###` for subsections; avoid `#` (reserved for page title)
- **Lists** — use bullet lists for unordered items, numbered lists for sequential steps only

## Terminology

Always use Newton-specific terms consistently:

| Term | Definition | Never say |
|------|-----------|-----------|
| **Policy** | Rego program that evaluates whether an Intent should be approved | rule, ruleset, script |
| **Intent** | Proposed transaction submitted for policy evaluation | request, transaction (alone) |
| **Task** | Pairs an Intent with its Policy for evaluation | job, evaluation, query |
| **Attestation** | Cryptographic proof that operators approved an Intent | certificate, signature (alone) |
| **PolicyClient** | Smart contract mixin that validates attestations | policy contract (imprecise) |
| **PolicyData** | WASM data oracle providing external data to policies | data feed (imprecise), oracle (alone) |
| **Operator** | EigenLayer node that evaluates tasks | validator, node (alone) |
| **Gateway** | JSON-RPC endpoint for submitting tasks | API server, backend |

## Cross-Linking

- Always link to related pages on first mention of a concept
- Use relative paths: `/developers/reference/rpc-api` not full URLs
- Link to the most specific page (e.g., link "SDK" to `/developers/reference/sdk-reference`, not the overview)

## Code Examples

- Must be **complete and runnable** — no `...` placeholders without surrounding context
- Include necessary imports
- Use realistic values (not `0x000...000` for addresses)
- Show both the request and expected response where applicable
- Use `<CodeGroup>` for multi-language/multi-package-manager examples
- Use `ts twoslash` fence meta for TypeScript SDK examples so code is type-checked at CI time
