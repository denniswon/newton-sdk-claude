# Security Guidelines

## Client-Side SDK Threat Model

This is a client-side library that runs in browsers and Node.js. The threat model differs from the AVS backend:

| Concern | Risk Level | Mitigation |
|---------|------------|------------|
| Bundle contains secrets | Critical | Never bundle API keys, private keys, or credentials |
| XSS via SDK inputs | High | Validate and sanitize all user inputs before contract calls |
| Man-in-the-middle on gateway | High | HTTPS only, validate gateway URLs |
| Supply chain attack | High | Minimal dependencies, lockfile pinning |
| Sensitive data in localStorage | Medium | Cache only non-sensitive data, clear on logout |
| Popup hijacking | Medium | Validate postMessage origins |

## Never Bundle Secrets

The SDK MUST NOT contain or embed:
- API keys (passed by consumer at runtime)
- Private keys (consumer manages their own wallet)
- RPC URLs with auth tokens (consumer provides their own transport)
- Internal service endpoints

```typescript
// Good: Consumer provides API key at runtime
const service = new AvsHttpService(chainId, apiKey)

// Bad: Hardcoded in SDK source
const API_KEY = 'sk-...'
```

## Input Validation

### Chain ID Validation

Validate early, fail fast:

```typescript
const SUPPORTED_CHAINS = [1, 11155111, 84532] as const

function validateChain(chainId: number): asserts chainId is SupportedChain {
  if (!SUPPORTED_CHAINS.includes(chainId as any)) {
    throw new SDKError(
      `Chain ${chainId} is not supported`,
      'UNSUPPORTED_CHAIN',
    )
  }
}
```

### Address Validation

Use viem's type system and validation:

```typescript
import { isAddress, getAddress } from 'viem'

// Good: Validate and checksum
if (!isAddress(address)) {
  throw new SDKError('Invalid address', 'INVALID_ADDRESS')
}
const checksummed = getAddress(address)

// Bad: Trust raw input
const result = await client.readContract({ address: rawInput, ... })
```

### Intent Parameter Validation

Validate intent parameters before sending to gateway:

```typescript
function validateIntent(intent: IntentFromParams): void {
  if (!intent.sender || intent.sender === '0x0000000000000000000000000000000000000000') {
    throw new SDKError('Invalid sender address', 'INVALID_SENDER')
  }
  if (intent.value !== undefined && BigInt(intent.value) < 0n) {
    throw new SDKError('Value cannot be negative', 'INVALID_VALUE')
  }
}
```

## Gateway Communication Security

### HTTPS Only

Gateway URLs must use HTTPS in production:

```typescript
// Validate gateway URL protocol
function validateGatewayUrl(url: string): void {
  const parsed = new URL(url)
  if (parsed.protocol !== 'https:' && !isLocalhost(parsed.hostname)) {
    throw new SDKError('Gateway must use HTTPS', 'INSECURE_GATEWAY')
  }
}
```

### API Key Handling

- API keys passed as `Authorization: Bearer` header (HTTP) or query param (WebSocket)
- Never log API keys
- Never store API keys in localStorage
- Consumer is responsible for key rotation

### Response Validation

Validate gateway responses before trusting them:

```typescript
// Validate JSON-RPC response structure
function validateRpcResponse(response: unknown): asserts response is JsonRpcResponse {
  if (!response || typeof response !== 'object') {
    throw new SDKError('Invalid RPC response', 'INVALID_RESPONSE')
  }
  if (!('jsonrpc' in response) || response.jsonrpc !== '2.0') {
    throw new SDKError('Invalid JSON-RPC version', 'INVALID_RESPONSE')
  }
}
```

## Popup Security (Newton Wallet)

### Origin Validation

Always validate `postMessage` origins:

```typescript
window.addEventListener('message', (event) => {
  // Validate origin against expected Newton Wallet domain
  if (!ALLOWED_ORIGINS.includes(event.origin)) {
    return // Silently ignore messages from unknown origins
  }
  handleWalletMessage(event.data)
})
```

### Window References

- Check popup window is not null before interacting
- Handle popup blockers gracefully
- Clean up event listeners on close

## Privacy Module Security (Planned)

When implementing the privacy module (NEWT-627):

### Key Material

- HPKE ephemeral keypairs generated per encryption — never reused
- Gateway public key fetched via `newt_getPrivacyPublicKey` RPC and cached
- Ed25519 signing keys derived from user's wallet — never exported
- All key material lives in memory only — never persisted to storage

### Encryption

- HPKE Base mode (RFC 9180): X25519 KEM + HKDF-SHA256 + ChaCha20-Poly1305
- AAD binding: `keccak256(policyClient, chainId)` prevents cross-context replay
- Envelope format includes ephemeral public key for recipient decryption

### Authorization Signatures

- Dual Ed25519 signatures (user + app) required for privacy data access
- Signature covers: `keccak256(policyClient, intentHash, refIds...)`
- Prevents unauthorized use of encrypted data across policy contexts

## Dependency Security

### Minimal Dependencies

Keep the dependency tree small:
- `viem` — blockchain interaction (required)
- `eventemitter3` — typed events (small, well-audited)
- `jose` — JWT/JWK (for auth tokens, well-audited)

Avoid adding dependencies for:
- Simple utilities (write them inline)
- Polyfills (target modern runtimes only)
- Framework-specific bindings (keep SDK framework-agnostic)

### Lockfile

Always commit `pnpm-lock.yaml`. Review dependency updates before merging.

## Sensitive Data Handling

### localStorage Cache

The `fetchWithCache` utility uses localStorage. Rules:
- Cache ONLY non-sensitive, public data (policy CIDs, static config)
- NEVER cache: API keys, signatures, private data, task responses with PII
- Set appropriate stale times (default 30s)
- Provide cache-clearing utility for consumers

### Console Output

- No `console.log` in production code
- No logging of request/response bodies (may contain sensitive intent data)
- Errors should expose codes and messages, not raw payloads
