# Newton Protocol Knowledge Base (`site/`)

Reference knowledge for writing accurate documentation. Verify claims against source repos before publishing.

## What Newton Is

Newton Protocol is a **policy engine for onchain transaction authorization**, built as an EigenLayer Actively Validated Service (AVS). It allows developers to encode, verify, and enforce rules — such as spend limits, jurisdictional gating, or fraud prevention — directly within smart contracts.

## Core Concepts

| Concept | Definition |
|---------|-----------|
| **Policy** | A Rego program defining conditions an Intent must meet to be approved. Stored on IPFS, referenced by CID. |
| **Intent** | A proposed transaction containing EVM fields (`from`, `to`, `value`, `data`, `chainId`, `functionSignature`) plus higher-level context. |
| **Task** | The atomic evaluation unit — pairs an Intent with its Policy and configuration. |
| **Attestation** | Cryptographic proof (BLS signature) that Newton operators evaluated a policy and approved/rejected the Intent. |
| **PolicyClient** | A smart contract mixin (`NewtonPolicyClient`) that validates attestations before executing transactions. |
| **PolicyData** | A WASM data oracle that fetches/computes external data at evaluation time. Output is fed to the Rego policy as `data.data`. |
| **Operator** | An EigenLayer node registered with Newton AVS that independently evaluates tasks. |
| **Gateway** | The JSON-RPC 2.0 endpoint that receives tasks and coordinates operator evaluation. |

## Architecture (3 Layers)

| Layer | Purpose | Components |
|-------|---------|------------|
| **Policy Layer** | Defines policies, schemas, rules, thresholds, offchain inputs | Policy Registry, Policy Library |
| **Compute & Consensus Layer** | Offchain policy evaluation by AVS operators, outputs verifiable proofs | AVS Operators, Aggregator, Consensus Proofs |
| **Verification & Execution Layer** | Onchain proof verification and outcome enforcement | NewtonVerifier, PolicyClient, SDKs |

## End-to-End Flow

1. **Developer deploys a Policy** — publishes reusable policy to the Newton registry
2. **User configures a Policy Client** — instantiates a PolicyClient contract with a chosen policy and configuration parameters
3. **Calling address submits a Task** — an Intent is paired with the policy client and sent to the Newton Gateway
4. **AVS Operators evaluate** — operators independently fetch data, evaluate the Rego policy, and produce individual attestations. The Aggregator collects signatures into a single BLS Consensus Proof once quorum is reached.
5. **Proof returned** — the aggregated attestation is returned to the caller, who submits it onchain. The NewtonVerifier contract validates the proof and returns ALLOW / REJECT / CAP.

## Gateway RPC Methods

| Method | Purpose |
|--------|---------|
| `newt_createTask` | Create a task for policy evaluation (returns attestation) |
| `newt_sendTask` | Submit a task and wait for evaluation result |
| `newt_simulateTask` | Simulate task evaluation without submitting onchain |
| `newt_simulatePolicy` | Simulate policy evaluation with provided inputs |
| `newt_simulatePolicyData` | Simulate a PolicyData WASM oracle execution |
| `newt_simulatePolicyDataWithClient` | Simulate PolicyData using stored secrets (requires ownership) |
| `newt_storeEncryptedSecrets` | Store encrypted secrets for a PolicyClient |
| `newt_getPrivacyPublicKey` | Fetch the gateway's X25519 HPKE public key |
| `newt_uploadEncryptedData` | Upload HPKE-encrypted data to the gateway |
| `newt_registerWebhook` | Register a webhook for task completion notifications (gateway-only, no SDK wrapper) |

**Authentication:** All API requests require a Bearer token in the `Authorization` header.

## Key Smart Contracts

| Contract | Purpose |
|----------|---------|
| `NewtonProverTaskManager` | Core task management and verification |
| `NewtonPolicyFactory` | Creates and registers new policies |
| `PolicyClientRegistry` | Tracks registered PolicyClient contracts |
| `IdentityRegistry` | Maps identities for policy evaluation |
| `AttestationValidator` | Validates BLS attestation proofs onchain |

## Supported Chains

| Chain | Chain ID | Status |
|-------|----------|--------|
| Ethereum Sepolia | `11155111` | Active |
| Base Sepolia | `84532` | Active |
| Ethereum Mainnet | `1` | Active |

## Gateway URLs

| Network | URL |
|---------|-----|
| Sepolia | `https://gateway.testnet.newton.xyz/rpc` |
| Base Sepolia | `https://gateway.testnet.newton.xyz/rpc` |
| Mainnet | `https://gateway.newton.xyz/rpc` |

## Source of Truth

When writing or editing documentation, verify technical claims against:

| Claim type | Verify against |
|-----------|---------------|
| RPC API parameters/responses | `newton-prover-avs/docs/RPC_API.md` |
| Smart contract interfaces | `newton-prover-avs/contracts/src/` |
| CLI commands and flags | `newton-prover-avs/crates/cli/` |
| SDK methods and types | `src/` (this repo's SDK source) |
| Contract addresses, URLs | `src/const.ts` and `site/developers/reference/sdk-reference.mdx` |

Do not fabricate contract addresses, RPC parameters, or API responses. If you cannot verify a claim, flag it with a TODO comment.
