# Documentation Agent Guide (`site/`)

Behavior rules for agents working on the `site/` documentation directory.

## Page Management

### Adding a new page

1. Create the MDX file in the correct directory under `site/`
2. Add the page path (without `.mdx`) to `site/docs.json` in the appropriate tab -> group -> pages array
3. Verify with `cd site && mintlify broken-links` after adding

### Moving or renaming a page

1. Move/rename the MDX file
2. Update the path in `site/docs.json`
3. Add a redirect in the `redirects` array in `site/docs.json`
4. Update any internal links in other MDX files that reference the old path

### Removing a page

1. Remove the path from `site/docs.json`
2. Add a redirect to the most relevant replacement page
3. Search for and update/remove internal links to the deleted page

## Consistency Rules

- Use the same terminology everywhere (see `site-content-style.md` terminology table)
- Use the same code style: TypeScript with viem for SDK examples, Solidity for contract examples, Rego for policy examples
- Use the same Mintlify components consistently (see `site-mintlify.md`)
- Keep imports, chain IDs, and gateway URLs consistent across all pages
- SDK code examples must match current exports in `src/` — when SDK changes, update docs

## Code Examples

- Must be **complete and runnable** — include all necessary imports
- No `...` placeholders without surrounding context that makes them unambiguous
- Use realistic but clearly example values for addresses and keys
- Show both request and response where applicable
- Use `<CodeGroup>` for npm/pnpm/yarn alternatives
- Contract examples should use Solidity ^0.8.x syntax
- TypeScript SDK examples should use `ts twoslash` fence meta for CI verification

## Testing Changes

After making documentation changes:

1. **Preview:** Run `cd site && mintlify dev` and check the affected pages at localhost:3000
2. **Links:** Run `cd site && mintlify broken-links` to verify no broken internal/external links
3. **Navigation:** Verify the page appears in the correct location in the sidebar
4. **Twoslash:** Run `pnpm docs:check` to verify TypeScript code samples type-check

## Docs-SDK Sync

When modifying SDK source code (`src/`), check if documentation needs updating:

| SDK change | Docs to update |
|-----------|---------------|
| New exported function | `site/developers/reference/sdk-reference.mdx` |
| Changed function signature | All docs pages that show usage of that function |
| New chain support | `site/developers/reference/contract-addresses.mdx` |
| New RPC method | `site/developers/reference/rpc-api.mdx` |
| New error code | `site/developers/reference/error-reference.mdx` |

## Documentation Structure

The Developers tab is organized as:

```
Getting Started -> Guides -> Use Cases -> Deep Dives -> Reference -> Advanced -> Resources
```

When adding new content, place it in the appropriate group.
