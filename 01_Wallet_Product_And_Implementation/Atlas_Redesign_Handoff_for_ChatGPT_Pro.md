# Atlas Redesign Handoff for ChatGPT Pro

Prepared: 2026-04-24  
Workspace audited: `/Users/devpup/Desktop/Testnet-Beta`  
Primary Atlas repo audited: `/Users/devpup/Desktop/Testnet-Beta/explorer-app`

This document compiles the current Atlas/explorer implementation, live endpoints, API shape, DAG reality, indexer/search limits, SynQ workflow status, ecosystem navigation, design assets, and launch-readiness gaps so the Atlas redesign can start from verified facts.

## Executive Summary

Current Atlas is architecturally salvageable as a linear Testnet-Beta block explorer:

- React 18 + Vite + TypeScript frontend.
- Fastify API backed by PostgreSQL.
- Polling TypeScript indexer against Synergy JSON-RPC.
- Static frontend deployed behind nginx.
- API/indexer deployed as PM2 processes.
- Current live explorer works for network summary, blocks, contracts, addresses, transactions, and validators, though live transaction and validator datasets are currently empty.

Current Atlas is not yet the full "network intelligence platform" described by the redesign brief:

- DAG views are intentionally disabled in the frontend.
- The current protocol/indexer data model is linear block-chain oriented, not a persisted DAG model.
- There is no canonical backend search endpoint.
- There is no fuzzy, relationship, semantic, or time-travel search architecture.
- There is no WebSocket or streaming Atlas API.
- SynQ deploy/verify is currently a UI/roadmap concept, not an implemented Atlas backend workflow.
- Risk labels, wallet labels, audit indicators, and trust/risk scoring do not have a production schema or policy implementation yet.
- Swagger is registered in the backend but public `/docs` currently serves the frontend SPA, not usable backend API docs.

Recommended planning stance:

1. Treat the current Atlas as a useful explorer shell and live data baseline.
2. Preserve working API-backed pages and existing deployment scripts where practical.
3. Redesign the frontend shell intentionally, but do not promise DAG/search/SynQ/risk intelligence until the backend data contracts exist.
4. For the full Atlas product, plan a backend/indexer rebuild or major expansion alongside the frontend redesign.

## Current URLs, Repo, and Source Access

### Local source

- Repo path: `/Users/devpup/Desktop/Testnet-Beta/explorer-app`
- Git remote: `https://github.com/synergy-network-hq/explorer.git`
- Branch audited: `main`
- Latest local commit observed: `3a74e06 Fix Atlas indexing and explorer routes`
- Package name: `testbeta-explorer-app`
- Dirty worktree note: `backend/src/index.ts` was already modified before this handoff audit. This document did not edit that file.

### Live and referenced endpoints

Verified on 2026-04-24 around 15:27 UTC:

- Explorer frontend: `https://testnet-explorer.synergy-network.io`
- Atlas API: `https://testnet-atlas-api.synergy-network.io/api/v1/network/summary`
- Core RPC: `https://testnet-core-rpc.synergy-network.io`

Listed in repo docs but not deeply exercised in this audit:

- Core WebSocket: `wss://testnet-core-ws.synergy-network.io`
- General API endpoint: `https://testnet-api.synergy-network.io`

The frontend URL returned HTTP 200 and page title `Synergy Testnet-Beta Explorer`.

The API root `https://testnet-atlas-api.synergy-network.io` returns a JSON 404 unless a versioned `/api/v1/...` route is used.

### Sandbox domain status

- `https://wallet.synergysandbox.xyz` returned HTTP 200.
- `https://atlas.synergysandbox.xyz` did not resolve during this audit.
- `https://synergysandbox.xyz` resolved to `71.86.65.178` but did not successfully connect on HTTPS during this audit.

Do not assume the Atlas sandbox route is live until DNS/hosting is fixed.

## Current Live Network Snapshot

Sample from `GET /api/v1/network/summary` on 2026-04-24:

```json
{
  "env": "testnet-beta",
  "chainId": "338639",
  "latestBlock": "8393",
  "avgBlockTimeSeconds": 11.25,
  "totalTransactions": "0",
  "activeValidators": "0",
  "totalValidators": "0",
  "averageSynergyScore": 0,
  "peerCount": "5",
  "indexedAt": "2026-04-24T15:27:38.776Z",
  "sourceRpc": "https://testnet-core-rpc.synergy-network.io",
  "fallbackRpc": null
}
```

Current implications:

- The chain is producing blocks.
- Indexed transaction count is currently zero.
- Indexed validator count is currently zero.
- Contracts endpoint has genesis/system contract data.
- `peerCount` reports five peers.
- Atlas should design empty states intentionally because several pages are live but sparse.

## Frontend Application Inventory

### Framework and app shape

- React 18
- Vite 7
- TypeScript
- Framer Motion
- Hash-based routing through `src/App.tsx`
- API client in `src/lib/api.ts`
- Feature flags in `src/lib/features.ts`

Vite config allows:

- `explorer.synergy-network.io`
- `testnet-explorer.synergy-network.io`
- localhost

Dev proxy:

- `/api` proxies to `http://localhost:3020`

### Current routes

Implemented or routed in `src/App.tsx`:

- `#/`
- `#/blocks`
- `#/block/:id`
- `#/dag`
- `#/dag/topology`
- `#/transactions`
- `#/tx/:hash`
- `#/contracts`
- `#/contracts/:address`
- `#/tokens`
- `#/wallet`
- `#/account`
- `#/validators`
- `#/validator/:address`
- `#/address/:address`
- `#/directory`
- `#/search/:query`
- `#/legal`

### Feature flag state

`src/lib/features.ts` currently disables DAG:

```ts
export const isDagExplorerEnabled = false;
export const DAG_COMING_SOON_MESSAGE = 'DAG views are not ready yet for Testnet Beta and remain disabled in Atlas.';
```

If a route starts with `/dag`, the app alerts the user and redirects to the dashboard.

### Current navigation model

Footer links:

- Explorer Home
- DAG Explorer, disabled/coming soon
- Atlas Docs, disabled/coming soon
- Synergy Portal, external `https://synergy-network.io`
- Terms/Privacy, `#/legal`
- Governance, disabled/coming soon

Bottom navigation:

- Home
- Blocks
- DAG, disabled when DAG flag is false
- Txs
- Tokens, disabled
- Contracts
- Wallet, disabled

### Search routing behavior

Search is client-side route classification only. There is no canonical backend search endpoint yet.

Current classification behavior:

- Numeric input routes to `/block/:number`.
- 64-character hex values, with or without `syntxn`/`synxxn` prefixes, route to `/tx/:hash`.
- `synv...` validator-shaped values route to `/validator/:address`.
- `syngrp...` or general `syn...` address-shaped values route to `/address/:address`.
- DAG terms only route to DAG pages if DAG is enabled.
- Everything else routes to `/search/:query`.

### Existing UX surfaces

The current UI includes:

- Dashboard KPIs
- Latest blocks
- Live transactions
- Blocks list/detail
- Transactions list/detail
- Addresses/detail
- Validators list/detail
- Contracts list/detail
- Tokens shell
- Legal notices
- DAG overview/topology shells, currently disabled

Dashboard copy implies future surfaces:

- Address Intelligence
- Smart Contract Workbench
- SynQ Deploy Studio

Those are not currently backed by full backend workflows.

## Current Atlas API

### API base

Frontend API base:

- `VITE_EXPLORER_API_BASE || '/api/v1'`

Backend route prefix:

- `/api/v1`

Backend framework:

- Fastify
- PostgreSQL
- pino logging
- CORS
- Swagger and Swagger UI registered, but public docs are not currently reachable as backend docs through the live frontend domain.

### Public endpoints in backend

Health/version endpoints:

- `GET /healthz`
- `GET /readyz`
- `GET /version`

Versioned Atlas endpoints:

- `GET /api/v1/network/summary`
- `GET /api/v1/blocks?page=&limit=`
- `GET /api/v1/blocks/:id?includeTx=true|false`
- `GET /api/v1/txs?page=&limit=`
- `GET /api/v1/txs/:hash`
- `GET /api/v1/address/:address`
- `GET /api/v1/validators?page=&limit=`
- `GET /api/v1/validators/:address`
- `GET /api/v1/contracts?page=&limit=&type=`
- `GET /api/v1/contracts/:address`

Admin/reset endpoints:

- `POST /api/v1/admin/reindex-from-genesis`
- `POST /v1/admin/reindex-from-genesis`

The reset endpoint is protected by `SYNERGY_EXPLORER_RESET_TOKEN` if set. If no token is configured, it only allows localhost callers. It also has an in-memory 60-second rate limit. It truncates `transactions`, `blocks`, `validators_current`, and `network_snapshots`; it does not truncate contracts.

### Live endpoint samples

#### Blocks

`GET /api/v1/blocks?limit=2`

```json
{
  "page": 1,
  "limit": 2,
  "total": 8394,
  "items": [
    {
      "number": "8393",
      "hash": "403575932a0bd0988f7f9390b2f6cd8cfd390c4a2b299dc823f4ef5d6be84543",
      "parent_hash": "1ada093d9e3e00d31d0612565910e1368bff075ed2ab94496336a250e0946c64",
      "timestamp": "2026-04-24T15:04:13.000Z",
      "validator_address": "synv11mvlgy72uq7kuh200qnxv67zrqjugz267k46",
      "tx_count": 0
    },
    {
      "number": "8392",
      "hash": "1ada093d9e3e00d31d0612565910e1368bff075ed2ab94496336a250e0946c64",
      "parent_hash": "0df39ec80e761a1418b01aab53206ab0372d8ce5d56af16b9883099cdb301b39",
      "timestamp": "2026-04-24T15:04:10.000Z",
      "validator_address": "synv118u0v2gxn4zew5j886hwz32tkaujsvhykf49",
      "tx_count": 0
    }
  ]
}
```

#### Transactions

`GET /api/v1/txs?limit=2`

```json
{
  "page": 1,
  "limit": 2,
  "total": 0,
  "items": []
}
```

#### Validators

`GET /api/v1/validators?limit=2`

```json
{
  "page": 1,
  "limit": 2,
  "total": 0,
  "items": []
}
```

#### Contracts

`GET /api/v1/contracts?limit=3`

The live contracts endpoint returned 8 total contracts. The first three were:

```json
{
  "page": 1,
  "limit": 3,
  "total": 8,
  "items": [
    {
      "address": "synq1ldqask8hhrplxfsej9cmme7s8545z5lslvzm",
      "name": "Governance",
      "contract_type": "genesis_system",
      "bytecode_hash": "38168b2ac43d07bfa790d96455def19d9f6b12d1f40d4c760ca2791b4476fa43",
      "verified": true,
      "deployed_block": 0,
      "deployed_at": "2026-04-01T12:00:00.000Z"
    },
    {
      "address": "synq16c7tddneaqz27saffctwrdnyfm9s5mrsdmlq",
      "name": "Identity",
      "contract_type": "genesis_system",
      "bytecode_hash": "48c6e3fc0cbeabbab1fa1f221e38e8f8f6023db7417554ca11de6783cd162b9c",
      "verified": true,
      "deployed_block": 0,
      "deployed_at": "2026-04-01T12:00:00.000Z"
    },
    {
      "address": "synq1wxm7skspall8ruqjpyq9pnkfxjswlawkn4mt",
      "name": "RewardDistributor",
      "contract_type": "genesis_system",
      "bytecode_hash": "6b0c84278ff2cb7d13b70e175ed657fb16d8e29f0fae750009c7a34fbe30ce22",
      "verified": true,
      "deployed_block": 0,
      "deployed_at": "2026-04-01T12:00:00.000Z"
    }
  ]
}
```

### Frontend API types

Current frontend client expects:

Block:

- `number`
- `hash`
- `parent_hash`
- `timestamp`
- `validator_address`
- `tx_count`

Transaction:

- `hash`
- `block_number`
- `block_timestamp`
- `tx_index`
- `sender_address`
- `receiver_address`
- `amount_nwei`
- `fee_nwei`
- `nonce`
- `data`
- `status`
- `signature_algorithm`
- `timestamp_unix`

Network summary:

- `env`
- `chainId`
- `latestBlock`
- `avgBlockTimeSeconds`
- `totalTransactions`
- `activeValidators`
- `totalValidators`
- `averageSynergyScore`
- `peerCount`
- `indexedAt`
- `sourceRpc`
- `fallbackRpc`

Address detail:

- wallet/account identity
- balances
- staking info
- recent transactions
- transfer history

Validator:

- address
- name
- stake
- synergy score
- blocks produced
- uptime
- cluster
- last active
- status
- role hint

Contract:

- `address`
- `name`
- `contract_type`
- `bytecode_hash`
- `verified`
- `deployed_block`
- `deployed_at`
- detail may include bytecode and init params

## Current Database Schema

The current Synergy-native schema is in `explorer-app/backend/migrations/002_synergy_native_schema.sql` and `003_contracts.sql`.

### Blocks

```sql
blocks (
  number bigint primary key,
  hash text unique not null,
  parent_hash text not null,
  timestamp timestamptz not null,
  validator_address text not null,
  tx_count integer not null default 0,
  inserted_at timestamptz not null default now()
)
```

Indexes:

- timestamp descending
- validator address

### Transactions

```sql
transactions (
  hash text primary key,
  block_number bigint references blocks(number) on delete cascade,
  tx_index integer not null default 0,
  sender_address text,
  receiver_address text,
  amount_nwei numeric(78, 0),
  fee_nwei numeric(78, 0),
  nonce bigint,
  data text,
  status text,
  signature_algorithm text,
  timestamp_unix bigint,
  inserted_at timestamptz not null default now()
)
```

Indexes:

- block number descending
- sender address
- receiver address

### Validators

```sql
validators_current (
  address text primary key,
  name text,
  stake_amount numeric(78, 0),
  synergy_score numeric(18, 6),
  blocks_produced bigint,
  uptime_percentage numeric(8, 4),
  cluster_id text,
  last_active timestamptz,
  status text,
  role_hint text,
  updated_at timestamptz not null default now()
)
```

### Network snapshots

```sql
network_snapshots (
  id bigserial primary key,
  latest_block bigint not null,
  total_transactions bigint not null,
  active_validators integer not null,
  total_validators integer not null,
  average_synergy_score numeric(18, 6),
  peer_count integer,
  average_block_time_seconds numeric(12, 4),
  indexed_at timestamptz not null default now()
)
```

### Contracts

```sql
contracts (
  address text primary key,
  name text not null,
  contract_type text not null default 'genesis_system',
  bytecode_hash text not null,
  bytecode text,
  init_params jsonb,
  verified boolean not null default true,
  deployed_block bigint not null default 0,
  deployed_at timestamptz,
  inserted_at timestamptz not null default now()
)
```

Important limitation: current contract verification is only a boolean in the active schema. It does not model verifier identity, source repository, compiler version, ABI, audit status, official labels, community labels, risk score, or verification evidence.

## Current Indexer and Search Architecture

### Indexer

Path:

- `/Users/devpup/Desktop/Testnet-Beta/explorer-app/indexer/src/index.ts`

The current indexer:

- Polls the Synergy JSON-RPC endpoint.
- Default poll interval: 2000ms.
- Default batch size: 50.
- Uses max indexed block number as its practical checkpoint.
- Upserts blocks and transactions.
- Deletes and refreshes `validators_current`.
- Inserts network snapshots.

Current upstream JSON-RPC methods used by the indexer:

- `synergy_getBlockNumber`
- `synergy_getBlockRange(start, end)`
- `synergy_getTransactionsInBlock`
- `synergy_getValidatorActivity`
- `synergy_getNetworkStats`
- `synergy_getValidatorStats`
- `synergy_getNodeStatus`

Current indexer gaps for the redesign:

- No durable checkpoint table.
- No reorg retention.
- No branch model.
- No finality/confidence model.
- No DAG parent-set model.
- No relationship graph.
- No full-text search tables.
- No fuzzy search.
- No semantic/entity search.
- No time-travel historical state query model.
- No WebSocket/streaming pipeline.
- No analytics event ingestion.
- No explicit cache layer.

### Search

Current search is not an index-backed product capability.

Implemented today:

- Client-side route classifier.
- Detail pages fetch exact block/tx/address/validator/contract identifiers.

Not implemented today:

- `GET /search`
- fuzzy address/entity search
- partial hash search
- token/project search
- contract label search
- relationship search
- natural language search
- "Google the network" experience
- saved/replayable query URLs with backend filters
- time-travel state search

For the redesign, search must be treated as a backend product surface, not a frontend-only component.

## Current Synergy JSON-RPC Surface

The protocol repo exposes a broader JSON-RPC surface than Atlas currently indexes.

Important methods found in `/Users/devpup/Desktop/Testnet-Beta/synergy-testnet-beta/src/rpc/rpc_server.rs` include:

Blocks and transactions:

- `synergy_blockNumber`
- `synergy_getBlockNumber`
- `synergy_getBlockByNumber`
- `synergy_getBlockByHash`
- `synergy_getLatestBlock`
- `synergy_sendTransaction`
- `synergy_getTransactionPool`
- `synergy_getBlockRange`
- `synergy_getTransactionByHash`
- `synergy_getTransactionsInBlock`
- `synergy_getTransactionReceipt`
- `synergy_getTransactionCount`
- `synergy_getBlockTransactionCount`
- `synergy_getBlockReceipts`
- `synergy_getPendingTransactions`
- `synergy_getTransactionByBlockNumberAndIndex`
- `synergy_getTransactionByBlockHashAndIndex`

Network and validators:

- `synergy_nodeInfo`
- `synergy_getValidators`
- `synergy_getValidator`
- `synergy_getValidatorStats`
- `synergy_getNodeStatus`
- `synergy_getSyncStatus`
- `synergy_getBlockValidationStatus`
- `synergy_getValidatorActivity`
- `synergy_getSynergyScoreBreakdown`
- `synergy_getPeerInfo`
- validator rewards, performance, queue, exit, slashing, cluster, rewards, delegators, and claim methods

Wallet and token:

- `synergy_createWallet`
- `synergy_getWallet`
- `synergy_createWalletFromKeypair`
- `synergy_getAllWallets`
- `synergy_signTransaction`
- `synergy_sendTokens`
- `synergy_stakeTokens`
- `synergy_unstakeTokens`
- `synergy_getStakedBalance`
- `synergy_getStakingInfo`
- `synergy_getTokenBalance`
- `synergy_getTokens`
- `synergy_createToken`
- `synergy_mintTokens`
- `synergy_burnTokens`
- `synergy_transferTokens`
- `synergy_getAllBalances`
- `synergy_getTransferHistory`

SXCP relayer:

- `synergy_registerRelayer`
- `synergy_unregisterRelayer`
- `synergy_relayerHeartbeat`
- `synergy_getRelayerSet`
- `synergy_getRelayerHealth`
- `synergy_getSxcpStatus`
- `synergy_submitAttestation`
- `synergy_getEventAttestation`
- `synergy_getAttestations`
- `synergy_slashRelayer`
- `synergy_setSxcpHeartbeatTimeout`
- `synergy_resetSxcpState`

EVM-like compatibility:

- `synergy_getBalance`
- `synergy_gasPrice`
- `synergy_call`
- `synergy_estimateGas`
- `synergy_getLogs`
- `synergy_getCode`
- `synergy_getStorageAt`
- chain ID and fee-related methods

### Live core RPC samples

`synergy_nodeInfo` returned:

```json
{
  "chainId": 338639,
  "networkId": 338639,
  "consensus": "Proof of Synergy",
  "currentBlock": 8393,
  "name": "genesisrpc",
  "syncing": false,
  "version": "1.2.0",
  "timestamp": 1777044557
}
```

`synergy_getNetworkStats` returned:

```json
{
  "active_validators": 0,
  "block_height": 8393,
  "current_epoch": 0,
  "network_uptime": "99.9%",
  "tokens": 1,
  "total_staked": 0,
  "total_supply": "51000000000000000000",
  "total_transactions": 0
}
```

`synergy_getBlockRange [8392, 8393]` returned two linear blocks where `previous_hash` and `parent_hash` are the same concept.

`synergy_getValidatorActivity` returned:

```json
{
  "average_synergy_score": 0.0,
  "total_active": 0,
  "validators": []
}
```

## Current DAG Data Model

### Current reality

The current live Atlas/protocol model is linear for block indexing.

Core block shape in `/Users/devpup/Desktop/Testnet-Beta/synergy-testnet-beta/src/block.rs`:

- `block_index`
- `timestamp`
- `transactions`
- `previous_hash`
- `validator_id`
- `nonce`
- `hash`
- `transactions_root`
- `proposer_public_key`
- `block_signature`
- `block_signature_algorithm`

Core block header shape:

- `number`
- `timestamp`
- `parent_hash`
- `hash`
- `validator_id`
- `transactions_root`

`Block::header()` maps `parent_hash` from `previous_hash`.

The node block-generation path uses previous block height plus one and previous block hash.

The sync validation path validates a single expected parent in sequence.

The RPC explorer JSON emits both `previous_hash` and `parent_hash` as the same single-parent value.

### Consensus evidence that exists but is not Atlas-modeled

There are consensus structures such as:

- votes
- vote equivocation evidence
- quorum certificates
- block hash
- epoch
- round
- quorum booleans
- conflicting vote evidence

These are not currently persisted, indexed, or exposed by Atlas as a DAG model.

### Current frontend DAG assets

There is a disabled frontend DAG prototype in:

- `/Users/devpup/Desktop/Testnet-Beta/explorer-app/src/lib/dagStub.ts`

It includes design concepts such as:

- Alpha/Beta/Gamma/Delta lanes
- sealed/merge/tip nodes
- topology clusters
- relay corridors
- finality ring
- quorum cover

These are visual/product concepts only. They should not be treated as the canonical protocol data model.

### Redesign implication

Before a real DAG explorer is designed, the team must decide what "DAG" means for Synergy:

- block DAG
- transaction dependency DAG
- committee/quorum DAG
- confirmation/finality DAG
- execution-lane DAG
- another protocol-specific graph

Minimum DAG data contract needed:

- node id
- node type
- parent ids
- child ids or edge query
- edge type
- lane or shard/cluster identity, if applicable
- block/transaction mapping
- topological order
- timestamp/order metadata
- proposer/validator identity
- finality state
- quorum/confidence state
- conflict/equivocation state
- selected/finalized branch state
- merge depth or ancestor distance
- reorg/branch retention policy
- query APIs for ancestors, descendants, branch comparison, and local neighborhood expansion

Without this contract, the DAG visualization should remain disabled or presented as concept-only.

## SynQ Deploy and Verify Workflow Status

### Current Atlas status

Atlas UI references:

- Smart Contract Workbench
- SynQ Deploy Studio
- contract intelligence
- ABI/call data style workflows

Current Atlas backend only exposes read-only contract registry/list/detail endpoints. It does not implement:

- compile
- simulate
- deploy
- verify
- ABI generation
- wallet signing
- deployment logs
- failure handling
- source-code verification lifecycle

### Current SynQ language status

The separate `synq-language` materials indicate the SynQ platform is not yet a production deploy pipeline.

Important current facts:

- The documented current CLI surface is `compile`, `run`, and `verify`.
- The SynQ user manual explicitly says no deploy/estimate/trace command surface is implemented in that CLI yet.
- Project status materials put the SynQ platform around 40-45 percent complete toward production readiness at last audit.
- CLI workflow, SDK, release readiness, parser/semantics/codegen/deploy workflow, and first-class developer workflows remain active blockers.

The protocol repo also has simplified/demo SynQ support:

- compiler path emits placeholder-style compiled output
- interpreter returns simplified execution success
- contract manager stores WASM bytecode in memory and execution is simulation/stub-like

### Redesign implication

SynQ Deploy Studio can be an excellent P1/P2 product direction, but P0 should not present it as live unless the backend work lands first.

Required SynQ backend contract for real Atlas implementation:

- source upload/input
- compiler version selection
- deterministic compile output
- ABI/interface output
- simulation endpoint
- gas/fee/weight estimate endpoint
- wallet signing payload manifest
- deploy transaction creation
- deploy broadcast
- deploy receipt polling
- verification submission
- verification evidence
- source/ABI/bytecode hash matching
- logs/events/errors
- permission and abuse controls

## Ecosystem Navigation Rules

### Current Atlas links

Current Atlas links mainly point to:

- Explorer Home
- Terms/Privacy
- Synergy Portal at `https://synergy-network.io`
- Disabled future DAG/docs/governance links

### Sandbox router product family

The wallet sandbox router currently lists these product links:

- Wallet: `https://wallet.synergysandbox.xyz`
- Spark: `https://spark.synergysandbox.xyz`
- Keystone: `https://keystone.synergysandbox.xyz`
- Frontier: `https://frontier.synergysandbox.xyz`
- Relay: `https://relay.synergysandbox.xyz`
- Learn: `https://learn.synergysandbox.xyz`
- Atlas: `https://atlas.synergysandbox.xyz`
- Synergy Network: `https://synergy-network.io`

Live verification caveat:

- Wallet sandbox works.
- Atlas sandbox does not currently resolve.
- Apex sandbox did not successfully connect on HTTPS during this audit.

### Workspace product-code caveat

From local workspace inventory:

- Atlas/explorer has a real repo under `explorer-app`.
- Wallet has a real repo under `wallet-app`.
- Some ecosystem names are product concepts or links, not confirmed standalone codebases in this workspace.
- Relay exists as protocol primitives, but a standalone Relay app was not found in this workspace.
- Keystone and Learn were not found as standalone codebases in this workspace.

### Recommended navigation model

Design product navigation as intentional cross-product routing:

- Atlas should link out to products that are actually live.
- Disabled/unavailable products should not look operational.
- Product-family footer can show ecosystem scope, but primary app navigation should stay focused on explorer tasks.
- If `synergysandbox.xyz` remains the router, Atlas should use one verified canonical Atlas URL, not an unresolved placeholder.

## Brand and Design Assets

### Current Atlas/explorer assets found

Explorer assets:

- `/Users/devpup/Desktop/Testnet-Beta/explorer-app/src/assets/snrg.gif`
- `/Users/devpup/Desktop/Testnet-Beta/explorer-app/src/assets/snrg.png`
- favicon assets
- alert PNGs
- `/Users/devpup/Desktop/Testnet-Beta/explorer-app/Alert-Icons/alert-icons.psd`

Possible Atlas/router asset:

- `/Users/devpup/Desktop/Testnet-Beta/wallet-app/sandbox/atlas.png`

No Figma, Stitch editable output, or component-export source was found in this workspace during the audit. There is an `explorer-app.zip`, but it appears to be a snapshot/archive, not editable mockup output.

### Current color system

From `explorer-app/src/styles/palette.css`:

- Proof of Synergy lime: `#00ff66`
- SXCP cyan: `#00ced1`
- SynQ blue: `#0060ff`
- Aegis purple: `#7d00ff`
- electric accent: `#00ffd4`
- neon accent: `#ff00e5`
- gold: `#ffd700`
- dark backgrounds
- glassmorphism/glow effects

Fonts currently imported:

- Rajdhani
- IBM Plex Sans
- Sora

Body text currently uses Sora. Header brand uses Rajdhani. Search uses IBM Plex Sans.

### Design-system tension

The current explorer UI uses substantial glass, glow, animation, and large radii around 20px.

The broader `SynergyDesignSystem.md` emphasizes:

- clarity over decoration
- performance over excess
- systems over one-off design
- dark-first blockchain UI
- developer-first functionality
- Inter and JetBrains Mono recommendations
- 8px grid
- 4/8/12px radius scale
- accessibility and WCAG
- semantic buttons
- tables with sort/filter/pagination

For the redesign, decide whether Atlas should preserve the current neon/glass identity or move toward the more restrained design-system direction.

## Risk, Labels, Trust, and Legal Policy

### Current legal/telemetry copy

The current legal page says:

- schemas may evolve
- users should validate contract addresses and payloads before signing/broadcasting
- Atlas may use limited technical telemetry for reliability/security
- optional analytics cookies may be used for product quality/reliability
- automation is allowed only within published rate constraints
- excessive requests may be throttled or blocked
- blockchain data may be delayed, reorganized, or partial
- Atlas is not financial/legal/investment advice
- smart contract interactions carry execution risk
- pre-release features, including SynQ deploy tooling, may be experimental

Cookie consent currently stores a local setting under:

- `synergy-atlas-cookie-consent-v2`

Modes:

- `all`
- `essential`

### Current risk/label implementation

No production entity-label or risk-scoring schema was found in the current Atlas database.

Current contract schema only includes:

- `verified boolean`

It does not include:

- verifier identity
- official vs community label source
- label confidence
- malicious/suspicious/trusted taxonomy
- audit source
- risk score
- risk explanation
- appeal/update policy
- moderation workflow
- signature/attestation for labels

### Required policy decisions

Atlas needs rules for:

- who can label an entity
- whether labels are official, partner, automated, or community-submitted
- how labels are verified
- how malicious labels are assigned and appealed
- how risk scores are calculated and explained
- whether scores are deterministic, model-driven, heuristic, or manual
- how false positives are corrected
- what warnings appear before wallet/signing interactions
- how audit indicators are sourced
- how contract verification differs from audit/trust status

## Infrastructure, Analytics, and Deployment Constraints

### Current deployment model

Frontend:

- Static Vite build
- Served by nginx
- Hash routing
- Live frontend root referenced by scripts as `/var/www/explorer/dist`

Backend:

- Fastify API
- Runs on `127.0.0.1:3020`
- Behind nginx
- PostgreSQL database
- PM2 process name: `explorer-api`

Indexer:

- TypeScript process using `tsx`
- PM2 process name: `explorer-indexer`
- Polls public Synergy RPC
- Writes to PostgreSQL

Deployment scripts:

- `/Users/devpup/Desktop/Testnet-Beta/explorer-app/deploy-explorer.sh`
- `/Users/devpup/Desktop/Testnet-Beta/explorer-app/deploy-backend.sh`
- `/Users/devpup/Desktop/Testnet-Beta/explorer-app/ops/check-explorer-pipeline.sh`

No `.github` CI/CD workflows were found in `explorer-app` during this audit.

### Current observability

Found:

- pino backend logs
- PM2 process model
- operational check script for API/indexer/RPC lag
- legal copy mentioning technical telemetry and optional analytics cookies

Not found:

- named analytics provider
- error monitoring provider
- external log aggregation
- tracing
- frontend performance monitoring
- formal alert dashboards
- cache layer
- public rate-limit docs

### Operational notes from repo

Internal checks removed from the UI but noted operationally include:

- proposer rotation and block interval anomaly checks
- gas saturation spikes tied to contract deployments
- parent hash continuity for fork visibility
- block cadence SLA escalation
- future analytics endpoints that could bind current filters

## Current Launch Scope and Readiness

The workspace mainnet readiness checklist identifies Atlas as not yet mainnet-ready.

Key Atlas readiness needs:

- remove beta/testnet naming where inappropriate
- freeze production deployment model
- define standalone product vs subcomponent boundaries
- define environment matrix
- build canonical search endpoint
- harden address analytics
- harden validator analytics
- add list sorting/filtering/export/query persistence where needed
- standardize pagination/response schemas
- add loading/empty/error states consistently
- define the real DAG model if DAG is protocol truth
- add durable indexer checkpoints
- add idempotent ingestion
- add reorg handling, retry, and replay tools
- add metrics for lag, RPC errors, throughput, reorgs, and DB write latency
- add backfill controls
- add archive-node dependency checks
- add request validation and OpenAPI docs
- add rate limiting, cache, and timeout policies
- protect admin endpoints
- add structured error codes
- add API compatibility tests
- add API contract tests
- improve frontend responsive and accessibility coverage
- add copy/share/download raw JSON features where useful
- define performance budgets
- add backup/restore/reindex runbooks
- add dashboards and alerts
- add deploy automation and failure drills

Suggested scope framing:

- P0 should be a production-quality linear explorer and design-system shell over current APIs.
- P1 should add canonical search, richer entity pages, and durable indexer improvements.
- P2 should add real DAG, SynQ deploy studio, risk intelligence, streaming, and advanced relationship intelligence after data contracts exist.

## Priority Personas

No official persona ranking was found in the repo. Current implementation implies this rough order:

1. Internal Synergy operators and technical testers, because current pages emphasize live network health, blocks, contracts, validators, and operational legal copy.
2. Developers, because contracts, raw JSON, search routing, and SynQ workbench/deploy copy are present.
3. Validators, because validator routes and Synergy score/staking fields exist, though live validator data is currently empty.
4. Token/project researchers, because token route exists but is currently a shell.
5. Casual explorers, because the dashboard and blocks pages are usable, but many discovery and explanatory features are not production-complete.

The redesign team should explicitly rank:

- casual explorers
- token/project researchers
- validators
- developers
- internal operators

This ranking should drive P0 navigation and home-screen density.

## P0 Implementation Spec Direction

### P0 product goal

Ship a polished Atlas explorer that accurately represents current backend capabilities:

- network overview
- blocks
- transactions
- addresses
- validators
- contracts
- exact identifier search
- raw JSON where useful
- reliable empty/loading/error states
- no false promises around disabled DAG, tokens, wallet, SynQ deploy, or risk scoring

### P0 screens

Dashboard:

- network health KPIs
- latest block
- average block time
- peer count
- indexed transaction count
- validator count with explicit empty state if zero
- latest blocks table
- latest transactions panel with empty state
- contracts summary
- clear status chip for Testnet-Beta

Blocks list:

- sortable/paginated table
- block number
- hash
- parent hash
- timestamp
- validator
- tx count
- copy hash/number actions

Block detail:

- block header
- parent link
- transaction list or empty state
- validator link
- raw JSON

Transactions list:

- paginated table
- hash
- block
- sender
- receiver
- amount
- fee
- status
- timestamp
- strong empty state because live total is currently zero

Transaction detail:

- status
- block link
- from/to links
- amount/fee/nonce/data
- raw JSON

Address detail:

- balance
- staking info
- transfer history
- recent txs
- raw JSON
- empty states for missing sections

Validators:

- validator count
- validator table
- status
- role
- stake
- score
- uptime
- cluster
- empty state if API returns zero

Contracts:

- genesis/system contracts list
- verified boolean display, but not "audited" unless audit data exists
- bytecode hash
- deployed block
- deployed at
- contract detail with raw JSON

Search:

- exact identifier search first
- route classifier should stay transparent
- if a backend search endpoint is not available, avoid implying fuzzy search
- search result page should explain unsupported broad search states without using "stub" or "prototype" in public copy

Legal:

- keep operational cautions
- align cookies/analytics copy with actual tooling before launch

### P0 backend/API work

Minimum:

- define OpenAPI routes with request/response schemas
- expose backend docs at a stable API URL
- standardize pagination shape
- standardize error codes
- add request validation
- add response compatibility tests
- add health/readiness response that matches operational scripts
- decide CORS production origins
- protect admin endpoints consistently

### P0 indexer work

Minimum:

- durable checkpoint table
- idempotent block/tx ingestion
- reindex/backfill command with dry run
- lag metrics
- RPC error metrics
- DB write latency metrics
- operator runbook

## P1 Roadmap

P1 should turn Atlas from a page explorer into a useful research tool:

- canonical `/api/v1/search` endpoint
- partial hash search
- address/validator/contract lookup
- contract name search
- token registry search when token data is real
- richer address analytics
- validator analytics and history
- contract verification metadata beyond boolean
- saved filters/query params on list pages
- export CSV/JSON for tables
- entity labels schema
- official label source policy
- rate-limit and API usage docs
- frontend accessibility pass
- mobile/responsive polish
- API contract test suite
- monitoring dashboard

## P2 Roadmap

P2 should only be committed after backend data contracts are defined:

- real DAG explorer
- ancestor/descendant graph expansion
- branch/finality/conflict views
- WebSocket or event streaming
- relationship search
- time-travel state queries
- SynQ deploy/verify studio
- ABI/source explorer
- contract risk intelligence
- wallet/entity risk labels
- malicious/trusted label workflow
- project/token intelligence pages
- network-wide insights and anomaly detection

## Immediate Questions for ChatGPT Pro or the Synergy Team

1. Should Atlas P0 be a polished linear explorer, or should launch wait for real DAG support?
2. What is the canonical definition of DAG for Synergy: block DAG, transaction DAG, quorum DAG, finality DAG, execution-lane DAG, or another graph?
3. Which user persona is primary for first production release?
4. Should broad search be promised for launch, or should P0 stick to exact identifier search?
5. Is SynQ Deploy Studio a launch requirement or a later developer-platform milestone?
6. What is the official product URL for Atlas: `testnet-explorer.synergy-network.io`, `testnet-atlas-api.synergy-network.io`, `atlas.synergysandbox.xyz`, or another production domain?
7. Should Atlas use the current neon/glass brand direction or align more tightly with the restrained Synergy design-system guidance?
8. What are the rules for official labels, community labels, audit indicators, and risk scoring?
9. Which analytics/error monitoring/logging providers are approved?
10. What is the release deadline and team size?

## Most Useful Next Package

For the next Atlas redesign/spec pass, provide:

1. Current editable Atlas/Stitch/Figma mockups, if they exist.
2. Any source-of-truth Atlas logo SVG/transparent PNG and usage rules.
3. Any internal API docs, Postman collections, or OpenAPI drafts not currently in the repo.
4. The intended production DAG data model.
5. The intended launch persona ranking.
6. The target launch date and whether launch requires DAG, search, SynQ deploy, or risk scoring.
7. The approved ecosystem URL map for Portal, Wallet, Spark, Keystone, Relay, Forge, Learn, SynQ, Aegis, and Atlas.

## Suggested Prompt to Give ChatGPT Pro

Use the following prompt with this handoff:

```text
You are helping redesign Synergy Atlas, the Synergy Network explorer and intelligence interface. Use the attached handoff as the source of truth. Do not assume DAG, fuzzy search, SynQ deploy, or risk scoring are implemented unless the handoff says they are. Produce a P0 implementation spec for a production-quality Atlas redesign that preserves real current capabilities, clearly labels unavailable future capabilities, and defines API-by-page requirements. Then produce P1/P2 milestones for backend/indexer/search/DAG/SynQ/risk expansion.

Primary constraint: current Atlas is a React/Vite frontend over a Fastify/Postgres API and polling indexer. It currently supports linear blocks, transactions, addresses, validators, contracts, and network summary. DAG is disabled, search is client-side routing only, and SynQ deploy is not yet a real Atlas backend workflow.

Deliverables:
1. P0 screen-by-screen wireframe specification.
2. API-by-page data contract table.
3. Component inventory.
4. Navigation model.
5. Empty/loading/error state rules.
6. Backend/indexer gaps that block future P1/P2 features.
7. Milestone backlog with dependencies.
```
