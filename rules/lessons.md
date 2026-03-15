# Lessons Learned

Living document. When a mistake recurs or a non-obvious pattern causes confusion, add an entry.

## Format

Each entry:
- **What happened**: Brief description
- **Why**: Root cause
- **Rule**: What to do differently

---

## Entries

### ABI files treated as source

- **What happened**: Agent edited `src/abis/newtonAbi.ts` directly
- **Why**: File looked like normal TypeScript source
- **Rule**: Files in `src/abis/` are auto-synced from newton-prover-avs contracts. Run `pnpm sync-abis` to update. Never edit manually.

### Vitest exits 1 with no test files

- **What happened**: `pnpm test` failed in CI because no `.test.ts` files exist on main branch
- **Why**: Vitest returns exit code 1 when it finds zero matching test files
- **Rule**: Use `--passWithNoTests` flag in the test script: `"test": "vitest run --passWithNoTests"`

### Package exports validation catches real issues

- **What happened**: `publint` detected broken `./viem` and `./networks` export paths that pointed to non-existent files
- **Why**: Exports were added to package.json but the corresponding source files were removed without updating exports
- **Rule**: Run `pnpm check:exports` after modifying the `exports` field in package.json. The CI workflow validates this automatically.

### BigInt serialization in JSON-RPC

- **What happened**: `BigInt` values in intent parameters caused `JSON.stringify` to throw `TypeError: Do not know how to serialize a BigInt`
- **Why**: JSON.stringify cannot handle BigInt natively
- **Rule**: Always convert BigInt to hex string (`0x` prefixed) via `sanitizeIntentForRequest` before sending to the gateway. The `normalizeIntent` function converts string amounts to BigInt for local use, but `sanitizeIntentForRequest` converts back to hex for gateway communication.

### Regorus is a git submodule, not a directory

- **What happened**: Tried to fetch identity extension source files from `libs/regorus/` in newton-prover-avs via GitHub API, got 404s
- **Why**: `libs/regorus` is a git submodule pointing to `https://github.com/newt-foundation/regorus.git`. GitHub API returns the submodule as a single tree entry with a commit SHA, not browseable files.
- **Rule**: To read regorus files from a PR: (1) get the PR head commit SHA, (2) find the submodule SHA from the commit's tree, (3) fetch from `newt-foundation/regorus` at that SHA directly. Check `.gitmodules` for the submodule URL.

### macOS base64 decoding differs from Linux

- **What happened**: `base64 -d` failed when decoding GitHub API content responses on macOS
- **Why**: macOS uses `base64 -D` (capital D), not `base64 -d` (Linux convention)
- **Rule**: Use `base64 -D` on macOS. Always strip newlines with `tr -d '\n'` before decoding GitHub API base64 content.

### Identity Rego namespace migration (flat → domain-namespaced)

- **What happened**: Documentation referenced `newton.identity.check_approved()` (flat namespace) instead of `newton.identity.kyc.check_approved()` (domain-namespaced)
- **Why**: Early design used flat `newton.identity.*` prefix. The implementation migrated to `newton.identity.<domain>.*` to support multiple identity domains, but some docs weren't updated.
- **Rule**: All identity Rego built-ins use domain-namespaced prefix: `newton.identity.kyc.*`. The only non-namespaced function is `newton.identity.get(field_name)` which works across all domains. When updating identity docs, grep for `newton.identity.` without `kyc.` or `get` to catch stale references.

### Docs must cover new RPC methods

- **What happened**: `newt_sendIdentityEncrypted` was implemented in the gateway but not documented in `site/developers/reference/rpc-api.mdx`
- **Why**: SDK code and gateway code were developed in parallel; the RPC reference wasn't updated when the new method was added
- **Rule**: When adding a new gateway RPC method, always add it to `rpc-api.mdx`. Verify against the gateway source (`crates/gateway/src/rpc/api/` and `crates/gateway/src/rpc/types/`) for accurate parameter and response schemas.
