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

### BigInt serialization in JSON-RPC

- **What happened**: `BigInt` values in intent parameters caused `JSON.stringify` to throw `TypeError: Do not know how to serialize a BigInt`
- **Why**: JSON.stringify cannot handle BigInt natively
- **Rule**: Always convert BigInt to hex string (`0x` prefixed) via `sanitizeIntentForRequest` before sending to the gateway. The `normalizeIntent` function converts string amounts to BigInt for local use, but `sanitizeIntentForRequest` converts back to hex for gateway communication.
