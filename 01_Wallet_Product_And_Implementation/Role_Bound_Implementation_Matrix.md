# Role-Bound Implementation Matrix

This file is the working source of truth for turning Synergy Testnet-Beta from a generic node runtime into the 19 role-bounded node binaries required by the architecture.

Primary source documents:

- `/Volumes/ExternalStorage/synergy-network-whitepaper/Synergy_Network_Node_Technical_Specification.docx`
- `/Users/devpup/Desktop/Synergy Network Node Architectures and Bootstrapping.pdf`

Non-negotiable requirements from the source documents:

- The network is not a one-daemon system with role labels layered on top.
- Each of the 19 node roles is a first-class profile with a bounded service surface.
- The control panel must provision typed role overlays, not free-form generic configs.
- The runtime must validate `identity.role` against the compiled profile.
- Role mismatch, integrity drift, or cryptographic policy failure must quarantine the node.
- Bootnodes are discovery-only and do not participate in consensus.
- Seed servers are crawler/publisher services and do not carry protocol authority.
- Bootstrapping must use multiple sources: hardcoded bootnodes, DNS `dnsaddr`, and signed seed artifacts.

## Current reality

The current Testnet-Beta implementation does not yet satisfy the source documents.

- `synergy-testnet-beta` currently ships one shared runtime in `synergy-testnet-beta/src/main.rs`.
- The control panel catalog in `synergy-testnet-beta/node-control-panel/src-tauri/src/testnet_beta.rs` describes 19 roles, but that catalog is metadata, not compiled role boundaries.
- The runtime currently exposes a small shared subsystem set: P2P, consensus, sync, RPC, wallet/state, SXCP state, and SynQ/AIVM scaffolding.
- Many role-specific services named by the spec do not yet exist as dedicated runtime modules.

## Required role matrix

### Class I: Consensus

#### Validator Node

- Required service surface:
  - `p2p`
  - `consensus`
  - `mempool`
  - `state`
  - `aegis-verifier`
  - `telemetry`
- Required ports:
  - `38638` P2P
  - `48638` localhost RPC
  - `58638` localhost WS
  - `9090` localhost metrics
- Current code status:
  - Partially present in shared runtime
  - Not yet enforced as a specialized compiled profile

#### Committee Node

- Required service surface:
  - `consensus`
  - `committee-sync`
  - `aegis-verifier`
  - `epoch-rotation listener`
- Required ports:
  - `38638` P2P
  - `9090` localhost metrics
- Current code status:
  - Committee role exists in control-panel metadata
  - No dedicated committee-sync / epoch-rotation runtime boundary exists yet

#### Archive Validator Node

- Required service surface:
  - `state`
  - `archive`
  - `proof-builder`
  - `snapshot`
- Required ports:
  - `38638` P2P
  - `48638` localhost read RPC
  - `9090` localhost metrics
- Current code status:
  - Shared state/sync exists
  - No dedicated archive, proof-builder, or snapshot role runtime exists yet

#### Audit Validator Node

- Required service surface:
  - `consensus-audit`
  - `qc-verifier`
  - `state-diff`
  - `alerting`
- Required ports:
  - `38638` P2P
  - `9090` localhost metrics
- Current code status:
  - No dedicated audit-validator runtime exists yet

### Class II: Interoperability

#### Relayer Node

- Required service surface:
  - `sxcp-relay`
  - `light-client adapters`
  - `attestation-packager`
  - `witness-registry client`
  - `telemetry`
- Required ports:
  - `3040` HTTPS SXCP API
  - `3041` WSS SXCP stream
  - `9090` localhost metrics
- Current code status:
  - SXCP state logic exists
  - No bounded relayer runtime with chain adapters and attestation packaging exists yet

#### Witness Node

- Required service surface:
  - `external-observer`
  - `proof-capture`
  - `witness-submit`
  - `telemetry`
- Required ports:
  - `9090` localhost metrics
- Current code status:
  - No dedicated witness runtime exists yet

#### Oracle Node

- Required service surface:
  - `oracle-fetch`
  - `source-auth`
  - `quote-normalizer`
  - `attestation-submit`
- Required ports:
  - `9090` localhost metrics
- Current code status:
  - No dedicated oracle runtime exists yet

#### UMA Coordinator Node

- Required service surface:
  - `uma-registry client`
  - `identity-refresh orchestration`
  - `mapping verifier`
  - `audit logging`
- Required ports:
  - `9090` localhost metrics
- Current code status:
  - No dedicated UMA coordinator runtime exists yet

#### Cross-Chain Verifier Node

- Required service surface:
  - `proof-verifier`
  - `finality-checker`
  - `scope-binder`
  - `receipt-issuer`
- Required ports:
  - `3030` HTTPS verify API
  - `9090` localhost metrics
- Current code status:
  - No dedicated cross-chain verifier runtime exists yet

### Class III: Execution, Data, and Cryptography

#### SynQ Execution Node

- Required service surface:
  - `synq-runtime`
  - `trace`
  - `determinism-check`
  - `predeploy-sim`
- Required ports:
  - `9090` localhost metrics
- Current code status:
  - `synq/` runtime scaffolding exists
  - Not yet exposed as a dedicated role-bounded execution node

#### Analytics and Simulation Node

- Required service surface:
  - `simulator`
  - `risk-model`
  - `anomaly-detector`
  - `reporting`
- Required ports:
  - `9090` localhost metrics
- Current code status:
  - No dedicated analytics/simulation runtime exists yet

#### Aegis Cryptography Node

- Required service surface:
  - `aegis-verify`
  - `kms-bridge`
  - `key-lifecycle`
  - `attestation-sign`
  - `audit-log`
- Required ports:
  - `3050` Aegis verify
  - `3051` private mTLS KMS
  - `9090` localhost metrics
- Current code status:
  - PQ crypto primitives exist
  - No dedicated Aegis verify / KMS bridge node runtime exists yet

#### Data-Availability Node

- Required service surface:
  - `da-store`
  - `shard-serve`
  - `proof-index`
  - `availability-audit`
- Required ports:
  - `9090` localhost metrics
- Current code status:
  - No dedicated DA runtime exists yet

### Class IV: Governance and Security

#### Governance Auditor Node

- Required service surface:
  - `governance-audit`
  - `vote-integrity`
  - `scope-check`
  - `reporting`
- Required ports:
  - `9090` localhost metrics
- Current code status:
  - No dedicated governance-auditor runtime exists yet

#### Treasury Controller Node

- Required service surface:
  - `treasury-exec`
  - `multisig-orchestrator`
  - `disbursement-audit`
- Required ports:
  - `9090` localhost metrics
- Current code status:
  - No dedicated treasury-controller runtime exists yet

#### Security Council Node

- Required service surface:
  - `emergency-scope`
  - `aegis-emergency-auth`
  - `incident-logging`
- Required ports:
  - `9090` localhost metrics
- Current code status:
  - No dedicated security-council runtime exists yet

### Class V: Service and Access

#### RPC Gateway Node

- Required service surface:
  - `rpc`
  - `ws`
  - `rate-limit`
  - `authn/authz`
  - `edge-cache`
- Required ports:
  - `48638` core RPC upstream
  - `58638` core WS upstream
  - `8545` EVM HTTP
  - `8546` EVM WS
- Current code status:
  - Generic RPC server exists
  - No dedicated gateway runtime with rate limiting, authn/authz, and upstream routing exists yet

#### Indexer and Explorer Node

- Required service surface:
  - `indexer-ingest`
  - `query-api`
  - `search`
  - `explorer-ui backend`
- Required ports:
  - `3010` ingest
  - `3011` indexer API
  - `3020` explorer API
- Current code status:
  - No dedicated indexer/explorer runtime exists yet

#### Observer / Light Node

- Required service surface:
  - `header-sync`
  - `light-proof-check`
  - `wallet-feed`
- Required ports:
  - implementation-specific read-only light API
  - `9090` localhost metrics
- Current code status:
  - No dedicated light-node runtime exists yet

## Implementation constraints from the specs

- One codebase is acceptable.
- One generic runtime with role names is not acceptable.
- The correct target is one codebase with multiple role profiles and role-specific builds or binaries.
- Each role binary must compile out or refuse to start subsystems outside its allowed surface.
- The control panel must provision role templates that map to those bounded runtime profiles.
- Nodes must quarantine on role/profile mismatch.

## Immediate implementation sequence

1. Add a canonical role/profile layer to `synergy-testnet-beta` that encodes all 19 roles, their service surfaces, and their startup invariants.
2. Split startup into role-gated service activation rather than unconditional shared startup.
3. Add role-specific binary entrypoints or build targets for all 19 roles.
4. Update the control panel to provision real runtime configs that match the canonical config schema and compiled profile.
5. Refuse provisioning/startup for any role whose required runtime services are not actually implemented.
6. Implement the missing role services until every role in this matrix is real.
