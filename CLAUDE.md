# Newton SDK - AI Agent Guidelines

## Project Overview

Newton SDK (`@magicnewton/newton-protocol-sdk`) is a TypeScript client-side SDK for interacting with the Newton Protocol. It provides viem client extensions for submitting policy evaluation tasks, managing policies, and receiving attestation results from the Newton Prover AVS.

## Tech Stack

| Component       | Technology                          |
|-----------------|-------------------------------------|
| Language        | TypeScript (strict mode)            |
| Package Manager | pnpm                                |
| Bundler         | Rollup (dual CJS + ESM output)     |
| Linting         | Biome                                   |
| Testing         | Vitest                                  |
| Versioning      | `auto` (Changesets migration planned)   |
| Git Hooks       | simple-git-hooks + lint-staged          |
| Blockchain      | viem ^2.x                           |
| Node            | >= 20                               |

## Architecture

### Viem Client Extension Pattern

The SDK follows viem's composable client actions pattern:

```typescript
import { createPublicClient, createWalletClient } from 'viem'
import { newtonPublicClientActions, newtonWalletClientActions } from '@magicnewton/newton-protocol-sdk'

const publicClient = createPublicClient({ ... }).extend(newtonPublicClientActions())
const walletClient = createWalletClient({ ... }).extend(newtonWalletClientActions({ apiKey: 'your-api-key', policyContractAddress: '0x...' }))
```

**Public client actions** (read-only): `waitForTaskResponded`, `getTaskStatus`, `getTaskResponseHash`, policy reads, `precomputePolicyId`

**Wallet client actions** (write): `submitEvaluationRequest`, `evaluateIntentDirect`, `submitIntentAndSubscribe`, simulate functions, policy writes, privacy functions, identity functions (`connectIdentityWithNewton`, `registerUserData`, `linkApp`, `unlinkApp`)

### Module Structure

```
src/
├── index.ts              # Main entry — exports client action extensions
├── const.ts              # Gateway URLs, contract addresses per chain
├── sdk-exceptions.ts     # SDKError, MagicRPCError
├── abis/                 # Contract ABIs (auto-synced from newton-prover-avs)
│   ├── newtonAbi.ts      # TaskManager + AttestationValidator ABIs
│   └── newtonPolicyAbi.ts # Policy contract ABI
├── modules/
│   ├── avs/              # Core AVS interaction (task submission, evaluation, WebSocket)
│   ├── policy/           # Policy contract read/write wrappers
│   └── privacy/          # HPKE encryption, Ed25519 signing, upload helpers
├── popup-prompt.ts       # Newton Wallet popup overlay renderer
├── service/
│   └── popup.ts          # Newton Wallet popup window management
├── types/
│   ├── index.ts          # Re-exports
│   ├── task.ts           # Intent, Task, TaskResponse, SimulateParams
│   ├── policy.ts         # PolicyId, PolicyParamsJson, SetPolicyInput
│   ├── privacy.ts        # SecureEnvelope, HPKE types
│   ├── identity.ts       # KycUserData
│   └── core/             # Exception types, JSON-RPC types
└── utils/
    ├── https.ts          # AvsHttpService — JSON-RPC over HTTP with API key
    ├── intent.ts         # normalizeIntent, sanitizeIntentForRequest
    ├── task-events.ts    # WebSocket task event subscription
    ├── cache-request.ts  # localStorage-based fetch cache
    ├── promise-tools.ts  # PromiEvent (Promise + EventEmitter hybrid)
    ├── events.ts         # Typed EventEmitter
    ├── json-rpc.ts       # createJsonRpcRequestPayload builder
    ├── jwt.ts            # decodeJWT — JWT header/payload decoder
    └── task.ts           # convertLogToTaskResponse — on-chain log converter
```

### Gateway Communication

The SDK communicates with the Newton Gateway via JSON-RPC 2.0:
- **HTTP**: `AvsHttpService` sends requests with `Authorization: Bearer` header
- **WebSocket**: `getTaskEventsWebSocket` subscribes to task topics with API key as query param
- Gateway URLs are resolved per chain from `GATEWAY_API_URLS` in `const.ts`

### Supported Chains

Validated in `newtonPublicClientActions` and `newtonWalletClientActions`:
- Ethereum mainnet (1)
- Sepolia (11155111)
- Base Sepolia (84532)

### SdkOverrides

All client actions accept optional `SdkOverrides` for custom gateway URL, task manager address, and attestation validator address — primarily for testing and staging environments.

## Auto-Generated Files (DO NOT EDIT)

| File | Source |
|------|--------|
| `src/abis/newtonAbi.ts` | Synced from `newton-prover-avs` contracts via `pnpm sync-abis` |
| `src/abis/newtonPolicyAbi.ts` | Synced from `newton-prover-avs` contracts via `pnpm sync-abis` |
| `dist/` | `pnpm build` output |

## Essential Commands

| Command | Purpose |
|---------|---------|
| `pnpm install` | Install dependencies |
| `pnpm build` | Build with Rollup (CJS + ESM) |
| `pnpm lint` | Run linter |
| `pnpm format` | Run formatter |
| `pnpm test` | Run tests |
| `pnpm typecheck` | Run `tsc --noEmit` |
| `pnpm check:exports` | Validate package exports with publint |
| `pnpm check:size` | Check bundle size limits (50 kB per entry) |
| `pnpm check:unused` | Detect dead code with knip |
| `pnpm check:all` | Run all quality checks (`check:exports` + `check:size`) |
| `pnpm sync-abis` | Sync ABIs from newton-prover-avs (TODO: set up script) |

## Planned Migrations

| From | To | Status |
|------|----|--------|
| `auto` versioning | Changesets (`@changesets/cli`) | Planned |

## Privacy Module

Client-side HPKE encryption for privacy-preserving policy evaluation. Shipped on main.

Exports: `createSecureEnvelope`, `getPrivacyPublicKey`, `uploadEncryptedData`, `uploadSecureEnvelope`, `generateSigningKeyPair`, `storeEncryptedSecrets`, `signPrivacyAuthorization`

Key properties:
- Zero server round-trips during encryption — fully offline capable
- HPKE suite: X25519 KEM + HKDF-SHA256 + ChaCha20-Poly1305 (RFC 9180)
- Ed25519 signing with caller-provided `Uint8Array` key (caller owns buffer lifecycle)
- `uploadSecureEnvelope` for uploading pre-built envelopes separately from encryption

See newton-prover-avs `docs/PRIVACY.md` for full protocol spec.

## Key Principles

1. **Use viem, not ethers.js** — this SDK is built on viem's client extension pattern
2. **Never bundle secrets** — no API keys, private keys, or credentials in the SDK bundle
3. **Never edit ABI files directly** — they are synced from newton-prover-avs contracts
4. **Validate chain IDs** — reject unsupported chains early in client action factories
5. **Tree-shakeable exports** — use named exports, avoid barrel files with side effects
6. **Type-safe contract interactions** — leverage viem's type inference from ABIs
7. **Minimal dependencies** — keep the dependency tree small for a client-side library

## Documentation Site (`site/`)

The `site/` directory contains the full [docs.newton.xyz](https://docs.newton.xyz) documentation site, hosted via [Mintlify](https://mintlify.com). It was imported from the `foundation-docs` repo via `git subtree`.

### Structure

```
site/
├── docs.json                    # Mintlify config (navigation, redirects, SEO)
├── style.css                    # Custom CSS overrides
├── favicon.svg
├── developers/                  # Developers tab (34 pages)
│   ├── overview/                #   Getting Started
│   ├── guides/                  #   Guides
│   ├── concepts/                #   Deep Dives
│   ├── reference/               #   Reference (SDK, RPC API, CLI, Contracts, Errors, Glossary)
│   ├── advanced/                #   Advanced (Rego, Secrets, WASM)
│   └── resources/               #   Resources (FAQ, Testing, Deployment)
├── whitepaper/                  # Whitepaper tab (10 pages)
├── protocol/                    # Protocol tab (8 pages)
├── newton-protocol/             # Legacy pages (redirected)
├── images/                      # Content images
├── logo/                        # Light/dark logos
└── og/                          # OpenGraph/Twitter social images
```

### Key Commands

| Command | Purpose |
|---------|---------|
| `cd site && mintlify dev` | Local preview at localhost:3000 |
| `cd site && mintlify broken-links` | Check for broken links |
| `pnpm docs:check` | Verify TypeScript code samples via twoslash-cli |

### Mintlify Deployment

Mintlify deploys from this repo's `site/` directory via its GitHub App integration (monorepo mode, documentation path: `/site`). Auto-deploys on push to main.

### Docs-SDK Sync

When modifying SDK source (`src/`), update corresponding docs in `site/developers/`. The `/docs-sync` skill handles this. Key sync points:

- New/changed exports → `site/developers/reference/sdk-reference.mdx`
- New chain support → `site/developers/reference/contract-addresses.mdx`
- New RPC methods → `site/developers/reference/rpc-api.mdx`

## Modular Rules

Detailed guidelines are in `.claude/rules/`:

| File | Contents |
|------|----------|
| `code-style.md` | TypeScript style, naming, formatting (viem conventions) |
| `architecture.md` | SDK modules, viem patterns, gateway communication |
| `testing.md` | Vitest patterns, test organization |
| `security.md` | Client-side crypto, key handling, bundle safety |
| `communication-style.md` | PR review tone, comment conventions |
| `lessons.md` | Recurring mistakes and prevention rules |
| `site-content-style.md` | Documentation writing style, voice, terminology |
| `site-mintlify.md` | Mintlify components, docs.json, page management |
| `site-newton-knowledge.md` | Newton Protocol knowledge base for docs accuracy |
| `site-agent-guide.md` | Agent behavior rules for `site/` directory work |

## Related Repositories

- [`newton-prover-avs`](https://github.com/newt-foundation/newton-prover-avs) — AVS backend (Rust + Solidity). Source of truth for contract ABIs and deployment addresses.
- [`newton-prover-avs` docs/RPC_API.md](https://github.com/newt-foundation/newton-prover-avs/blob/main/docs/RPC_API.md) — Gateway JSON-RPC API reference
- [`newton-prover-avs` docs/PRIVACY.md](https://github.com/newt-foundation/newton-prover-avs/blob/main/docs/PRIVACY.md) — Privacy layer protocol spec
- [`newton-prover-avs` docs/POLICY_CLIENT_GUIDE.md](https://github.com/newt-foundation/newton-prover-avs/blob/main/docs/POLICY_CLIENT_GUIDE.md) — Policy client integration guide
