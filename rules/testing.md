# Testing Guidelines

## Framework: Vitest

### Vitest Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'
import { resolve } from 'path'

export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
    },
    environment: 'node',
    globals: true,
  },
  resolve: {
    alias: {
      '@core': resolve(__dirname, 'src'),
    },
  },
})
```

## Test Organization

### Co-located Tests (Viem Pattern)

Place test files next to the source they test:

```
src/
├── modules/
│   ├── avs/
│   │   ├── index.ts
│   │   └── index.test.ts      # Tests for AVS module
│   └── policy/
│       ├── index.ts
│       └── index.test.ts      # Tests for policy module
├── utils/
│   ├── intent.ts
│   ├── intent.test.ts         # Tests for intent utils
│   ├── cache-request.ts
│   └── cache-request.test.ts
└── types/
    └── (no tests — type-only files)
```

### Test Naming Convention

```
function_name.condition.expected_behavior
```

```typescript
describe('submitEvaluationRequest', () => {
  it('returns PendingTaskBuilder on successful submission', async () => {})
  it('throws SDKError for unsupported chain', async () => {})
  it('passes SdkOverrides to gateway URL resolution', async () => {})
})

describe('normalizeIntent', () => {
  it('converts string amounts to bigint', () => {})
  it('preserves existing bigint values', () => {})
  it('handles zero values correctly', () => {})
})
```

## Test Categories

### Unit Tests

Test pure functions and type transformations in isolation:

```typescript
import { normalizeIntent, sanitizeIntentForRequest } from './intent'

describe('normalizeIntent', () => {
  it('converts string value to bigint', () => {
    const intent = { value: '1000000000000000000', to: '0x...' }
    const normalized = normalizeIntent(intent)
    expect(normalized.value).toBe(1000000000000000000n)
  })
})
```

### Integration Tests

Test module interactions with mocked gateway/contracts:

```typescript
import { createPublicClient, http } from 'viem'
import { sepolia } from 'viem/chains'
import { newtonPublicClientActions } from '@core/index'

describe('newtonPublicClientActions', () => {
  it('extends public client with Newton methods', () => {
    const client = createPublicClient({
      chain: sepolia,
      transport: http(),
    }).extend(newtonPublicClientActions())

    expect(client.waitForTaskResponded).toBeDefined()
    expect(client.getTaskStatus).toBeDefined()
  })

  it('rejects unsupported chains', () => {
    expect(() => {
      createPublicClient({
        chain: { id: 999999 },
        transport: http(),
      }).extend(newtonPublicClientActions())
    }).toThrow('Unsupported chain')
  })
})
```

### Contract Tests

Test contract interaction wrappers with anvil fork:

```typescript
import { createTestClient } from 'viem'
import { foundry } from 'viem/chains'

describe('policy module', () => {
  it('reads policy CID from contract', async () => {
    const client = createTestClient({
      chain: foundry,
      mode: 'anvil',
      transport: http('http://localhost:8545'),
    })

    const cid = await getPolicyCid(client, {
      policyAddress: KNOWN_POLICY_ADDRESS,
    })

    expect(cid).toMatch(/^Qm/)
  })
})
```

## Mocking

### Gateway HTTP Mocking

Mock `AvsHttpService` for unit tests:

```typescript
import { vi } from 'vitest'

vi.mock('@core/utils/https', () => ({
  AvsHttpService: vi.fn().mockImplementation(() => ({
    Post: vi.fn().mockResolvedValue({
      result: { taskId: '0x123...', response: { ... } },
    }),
  })),
}))
```

### Viem Client Mocking

Use viem's test utilities:

```typescript
import { createTestClient, publicActions } from 'viem'
import { foundry } from 'viem/chains'

const testClient = createTestClient({
  chain: foundry,
  mode: 'anvil',
  transport: http(),
}).extend(publicActions)
```

## What to Test

- **Type transformations**: Intent normalization, hex conversion, ABI encoding
- **Chain validation**: Supported chain checks in action factories
- **Error paths**: Gateway errors, contract reverts, timeout handling
- **Contract reads**: Policy data retrieval with known test contracts
- **Gateway RPC**: Request formatting, response parsing
- **WebSocket**: Connection, subscription, event handling

## What NOT to Test

- **viem internals**: Trust viem's contract read/write — test your wrappers
- **ABI correctness**: ABIs are auto-synced — test at the integration level
- **Trivial getters**: If it just re-exports a constant, skip it
- **Type-only files**: `types/*.ts` have no runtime behavior

## Before Committing

```bash
pnpm typecheck    # Type checking
pnpm lint         # Linting
pnpm test         # Run tests
pnpm build        # Verify build
```
