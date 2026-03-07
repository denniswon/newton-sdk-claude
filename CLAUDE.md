# Newton SDK - AI Agent Guidelines

## Project Overview

Newton SDK (`@magicnewton/newton-protocol-sdk`) is a TypeScript client-side SDK for interacting with the Newton Protocol. It provides viem client extensions for submitting policy evaluation tasks, managing policies, and receiving attestation results from the Newton Prover AVS.

## Tech Stack

| Component       | Technology                          |
|-----------------|-------------------------------------|
| Language        | TypeScript (strict mode)            |
| Package Manager | pnpm                                |
| Bundler         | Rollup (dual CJS + ESM output)     |
| Linting         | Biome (planned migration from ESLint + Prettier) |
| Testing         | Vitest (planned migration from Jest placeholder) |
| Versioning      | Changesets (planned migration from `auto`) |
| Git Hooks       | simple-git-hooks (planned migration from Husky) |
| Blockchain      | viem ^2.x                           |
| Node            | >= 20                               |

## Architecture

### Viem Client Extension Pattern

The SDK follows viem's composable client actions pattern:

```typescript
import { createPublicClient, createWalletClient } from 'viem'
import { newtonPublicClientActions, newtonWalletClientActions } from '@magicnewton/newton-protocol-sdk'

const publicClient = createPublicClient({ ... }).extend(newtonPublicClientActions())
const walletClient = createWalletClient({ ... }).extend(newtonWalletClientActions())
```

**Public client actions** (read-only): `waitForTaskResponded`, `getTaskStatus`, policy reads, `precomputePolicyId`

**Wallet client actions** (write): `submitEvaluationRequest`, `evaluateIntentDirect`, `submitIntentAndSubscribe`, simulate functions, policy writes

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
│   └── policy/           # Policy contract read/write wrappers
├── service/
│   └── popup.ts          # Newton Wallet popup window management
├── types/
│   ├── index.ts          # Re-exports
│   ├── task.ts           # Intent, Task, TaskResponse, SimulateParams
│   └── policy.ts         # PolicyId, PolicyParamsJson, SetPolicyInput
└── utils/
    ├── https.ts          # AvsHttpService — JSON-RPC over HTTP with API key
    ├── intent.ts         # normalizeIntent, sanitizeIntentForRequest
    ├── task-events.ts    # WebSocket task event subscription
    ├── cache-request.ts  # localStorage-based fetch cache
    ├── promise-tools.ts  # PromiEvent (Promise + EventEmitter hybrid)
    └── events.ts         # Typed EventEmitter
```

### Gateway Communication

The SDK communicates with the Newton Gateway via JSON-RPC 2.0:
- **HTTP**: `AvsHttpService` sends requests with `X-API-Key` header
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
| `pnpm sync-abis` | Sync ABIs from newton-prover-avs (TODO: set up script) |

## Planned Migrations

| From | To | Status |
|------|----|--------|
| ESLint + Prettier | Biome | Planned |
| Jest (placeholder) | Vitest | Planned |
| `auto` versioning | Changesets (`@changesets/cli`) | Planned |
| Husky | simple-git-hooks | Planned |

## Privacy Module (Planned — NEWT-627, NEWT-182)

Client-side-only HPKE encryption for privacy-preserving policy evaluation. Design constraints:
- Zero server round-trips during `encrypt()` — fully offline capable
- Bundle size < 50KB for the privacy module
- HPKE suite: X25519 KEM + HKDF-SHA256 + ChaCha20-Poly1305 (RFC 9180)
- Ed25519 authorization signatures for consent-based data access
- Will expose: `encryptPrivacyData()`, `uploadEncryptedData()`, `authorizePrivacyData()`

See newton-prover-avs `docs/PRIVACY.md` for full protocol spec.

## Key Principles

1. **Use viem, not ethers.js** — this SDK is built on viem's client extension pattern
2. **Never bundle secrets** — no API keys, private keys, or credentials in the SDK bundle
3. **Never edit ABI files directly** — they are synced from newton-prover-avs contracts
4. **Validate chain IDs** — reject unsupported chains early in client action factories
5. **Tree-shakeable exports** — use named exports, avoid barrel files with side effects
6. **Type-safe contract interactions** — leverage viem's type inference from ABIs
7. **Minimal dependencies** — keep the dependency tree small for a client-side library

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

## Related Repositories

- [`newton-prover-avs`](https://github.com/newt-foundation/newton-prover-avs) — AVS backend (Rust + Solidity). Source of truth for contract ABIs and deployment addresses.
- [`newton-prover-avs` docs/RPC_API.md](https://github.com/newt-foundation/newton-prover-avs/blob/main/docs/RPC_API.md) — Gateway JSON-RPC API reference
- [`newton-prover-avs` docs/PRIVACY.md](https://github.com/newt-foundation/newton-prover-avs/blob/main/docs/PRIVACY.md) — Privacy layer protocol spec
- [`newton-prover-avs` docs/POLICY_CLIENT_GUIDE.md](https://github.com/newt-foundation/newton-prover-avs/blob/main/docs/POLICY_CLIENT_GUIDE.md) — Policy client integration guide
