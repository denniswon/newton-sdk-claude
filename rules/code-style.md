# Code Style Guidelines

## Benchmark

Follow patterns established by [viem](https://github.com/wevm/viem). When in doubt, check how viem does it.

## Formatting: Biome

Configured in `biome.json`:
- Indent: 2 spaces
- Line width: 120
- Single quotes
- Trailing commas (all)

Run with `pnpm lint` (check) or `pnpm lint:fix` (auto-fix). Formatting with `pnpm format`.

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Functions | camelCase | `submitEvaluationRequest` |
| Variables | camelCase | `taskResponse` |
| Constants | SCREAMING_SNAKE_CASE | `GATEWAY_API_URLS` |
| Types/Interfaces | PascalCase | `TaskResponse` |
| Enums | PascalCase | `TaskStatus` |
| Enum values | PascalCase | `TaskStatus.Responded` |
| Files | kebab-case | `task-events.ts` |
| Test files | kebab-case with `.test` | `task-events.test.ts` |
| Type files | kebab-case | `task.ts` |

### Descriptive Names

```typescript
// Good: Clear and descriptive
const gatewayApiUrl = resolveGatewayUrl(chainId)
const isChainSupported = SUPPORTED_CHAINS.includes(chainId)
const taskResponseResult = await waitForTaskResponded(client, taskId)

// Bad: Abbreviated or unclear
const url = getUrl(cid)
const ok = check(chainId)
const res = await wait(c, tid)
```

## TypeScript Patterns

### Strict Mode

All code runs with `strict: true`. No `any` unless absolutely necessary and documented.

### Type Exports

Export types separately from runtime values:

```typescript
// Good: Explicit type exports
export type { TaskResponse, TaskStatus, IntentFromParams }
export { submitEvaluationRequest, waitForTaskResponded }

// Bad: Mixed exports that hurt tree-shaking
export * from './types'
```

### Viem Client Extension Pattern

Follow viem's action creator pattern:

```typescript
// Client action factory — returns an object of methods bound to the client
export function newtonPublicClientActions() {
  return (client: PublicClient) => ({
    waitForTaskResponded: (params: WaitParams) =>
      waitForTaskResponded(client, params),
    getTaskStatus: (params: StatusParams) =>
      getTaskStatus(client, params),
  })
}
```

### Prefer `const` Assertions

```typescript
// Good: Narrow types with const assertion
const SUPPORTED_CHAINS = [1, 11155111, 84532] as const
type SupportedChain = (typeof SUPPORTED_CHAINS)[number]

// Bad: Wide type
const SUPPORTED_CHAINS: number[] = [1, 11155111, 84532]
```

### Discriminated Unions over Optional Fields

```typescript
// Good: Discriminated union
type TaskResult =
  | { status: 'success'; data: TaskResponse }
  | { status: 'error'; error: SDKError }

// Bad: Optional fields
type TaskResult = {
  status: string
  data?: TaskResponse
  error?: SDKError
}
```

### No Default Exports

Always use named exports for tree-shaking:

```typescript
// Good
export function submitEvaluationRequest() { ... }
export { submitEvaluationRequest }

// Bad
export default function submitEvaluationRequest() { ... }
```

## Imports

### Organization

Group imports in order:

1. Node built-ins
2. External packages (viem, eventemitter3, etc.)
3. Internal modules (`@core/...` or relative)
4. Types (with `type` keyword)

```typescript
import { createPublicClient, type Address, type Hash } from 'viem'
import { sepolia } from 'viem/chains'

import { GATEWAY_API_URLS } from '@core/const'
import { AvsHttpService } from '@core/utils/https'

import type { TaskResponse, IntentFromParams } from '@core/types'
```

### Path Aliases

Use the configured `@core/*` alias for `src/*`:

```typescript
// Good
import { GATEWAY_API_URLS } from '@core/const'

// Acceptable for same-module imports
import { normalizeIntent } from './intent'
```

## Error Handling

### SDK Errors

Use the established `SDKError` and `MagicRPCError` classes:

```typescript
// Throw SDK errors with codes
throw new SDKError('Chain not supported', 'UNSUPPORTED_CHAIN')

// RPC errors with proper codes
throw createRpcError({ code: -32602, message: 'Invalid params' })
```

### Never Swallow Errors

```typescript
// Bad: Silent failure
try { await submitTask(intent) } catch { }

// Good: Handle or propagate
try {
  await submitTask(intent)
} catch (error) {
  throw new SDKError(`Task submission failed: ${error.message}`, 'TASK_SUBMIT_FAILED')
}
```

## Comments

### General Rules

- No emojis in comments
- Write in neutral, professional tone
- Comment **why**, not **what**
- No AI-generated context comments
- No commented-out code (delete it)

### JSDoc for Public API

```typescript
/**
 * Submits a policy evaluation request to the Newton Gateway.
 *
 * Returns a PendingTaskBuilder that can be used to wait for the response
 * or subscribe to task events via WebSocket.
 *
 * @param client - viem WalletClient with Newton extensions
 * @param params - Intent parameters and optional overrides
 * @returns PendingTaskBuilder for tracking the submitted task
 * @throws SDKError if the chain is unsupported or the gateway is unreachable
 */
export async function submitEvaluationRequest(
  client: WalletClient,
  params: SubmitEvaluationRequestParams,
): Promise<PendingTaskBuilder> {
```

## Anti-Patterns

| Pattern | Problem | Solution |
|---------|---------|----------|
| `any` type | Loss of type safety | Use `unknown` + type guards |
| Default exports | Hurts tree-shaking | Named exports only |
| Barrel files with side effects | Bundle bloat | Direct imports |
| `as` type assertions | Unsafe | Use type guards or `satisfies` |
| Mutable global state | Race conditions, testing issues | Module-scoped constants or params |
| `console.log` | Leaks to consumer's console | Remove or use debug flag |
