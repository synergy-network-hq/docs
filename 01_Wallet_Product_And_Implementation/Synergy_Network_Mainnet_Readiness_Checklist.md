# Synergy Network Mainnet Readiness Checklist

Reviewed against the workspace on 2026-03-25.

This document is a repo-grounded execution checklist for getting the current Synergy products to a mainnet-ready state. It is not a marketing roadmap. It is a delivery checklist anchored to what is actually present in `/Users/devpup/Desktop/Testnet-Beta` today.

## Current Repo Snapshot

| Product | Current state in workspace | Mainnet-readiness note |
|---|---|---|
| `Synergy-Testnet-Beta` | Active Rust protocol repo with configs, scripts, role binaries, security docs, bootstrap assets, and a checked-in `config/genesis.json` | Largest launch dependency; genesis, bootstrap topology, validator ceremony, endpoint consistency, and operational signoff are not yet frozen |
| `Node Control Panel App` | Active Electron/Vite app with Rust `control-service` nested in `synergy-testnet-beta/node-control-panel` | Substantial implementation exists, but packaging, role hardening, public operator experience, and release validation still need completion |
| `Synergy Atlas` | Active explorer repo in `explorer-app` with frontend, Fastify backend, Postgres schema, and indexer | Explorer baseline exists; DAG-aware data model and production hardening are still missing |
| `Synergy Wallet` | Active monorepo in `wallet-app` with web, desktop, mobile, extension, shared packages, and local helper service | Product exists, but production key-management architecture, testing depth, and release hardening are incomplete |
| `Synergy Relay` | No standalone top-level product repo found; protocol foundations exist in `synergy-testnet-beta/services/sxcp-relayer` and `synergy-testnet-beta/src/bin/synergy-relayer-node.rs` | Treat as not started as a full product even though relay daemon primitives exist |
| `Synergy Spark` | Active repo in `meme-launchpad` with frontend, backend, contracts, legal docs, and tests | Started, but not yet hardened for production money flows, moderation, or abuse handling |
| `Synergy Keystone` | No repo/folder found in the workspace | Greenfield product |
| `Synergy Learn` | No repo/folder found in the workspace | Greenfield product |

## Cross-Program Gates

### Program governance

- [ ] Appoint one accountable owner for each product plus one release manager for cross-product launch orchestration.
- [ ] Define the exact launch sequence: protocol first, infra second, wallet third, explorer fourth, governance fifth, application products last.
- [ ] Decide whether the immediate target is `public testnet`, `mainnet-beta`, or `mainnet`, and remove mixed terminology from docs, configs, scripts, and hostnames.
- [ ] Freeze a single naming convention for environments, chain labels, binary names, and public endpoints.
- [ ] Create a release train with entry criteria, exit criteria, rollback criteria, and a no-new-features freeze window.
- [ ] Define severity levels, incident commander rotation, and a launch-day escalation channel.

### Security and release engineering

- [ ] Produce a signed SBOM for every shipped desktop app, mobile app, browser extension, backend service, and Rust binary.
- [ ] Add reproducible build procedures and artifact signing for all production binaries and bundles.
- [ ] Centralize secrets management instead of relying on ad hoc `.env` files on operator machines.
- [ ] Add dependency auditing, license review, and provenance checks across Rust, Node, Electron, React Native, Solidity, and backend services.
- [ ] Commission independent security review for protocol, wallet, relayer, explorer backend, and smart contracts before mainnet value is enabled.
- [ ] Define a coordinated vulnerability disclosure policy and emergency patch process.

### Shared infrastructure

- [ ] Stand up production DNS, TLS, reverse proxies, WAF policy, load balancing, backups, and uptime checks for every public endpoint family.
- [ ] Define canonical RPC, WS, API, explorer, wallet API, indexer, SXCP, and governance endpoints for the target environment.
- [ ] Standardize structured logging, metrics, tracing, dashboards, alert routing, and pager ownership across all services.
- [ ] Create production/staging/dev environment parity rules and forbid prod-only manual config drift.
- [ ] Add backup and restore drills for RocksDB, Postgres, MongoDB, uploaded assets, and key metadata.
- [ ] Establish SLOs for protocol liveness, RPC latency, explorer freshness, wallet transaction success, and relayer settlement times.

### QA and support

- [ ] Build an end-to-end test matrix that covers protocol start, wallet creation, send/receive, explorer indexing, governance actions, and relay flows.
- [ ] Add long-running soak tests for chain stability, indexer lag, RPC reliability, and operator control-panel actions.
- [ ] Create a release candidate checklist that must be signed by engineering, infra, security, and support.
- [ ] Publish operator docs, user docs, API docs, runbooks, and rollback procedures before public release.
- [ ] Create a support intake flow, status page, and launch-day FAQ.

## 1. Synergy-Testnet-Beta

Repo reviewed: `synergy-testnet-beta/`

### Repo-grounded observations to resolve before anything called “mainnet ready”

- [ ] Reconcile the checked-in `config/genesis.json` with the generator at `scripts/testbeta/generate-testnet-beta-genesis.sh`.
- [ ] Resolve the `max_validators` mismatch: `config/genesis.json` is `15`, while `templates/*.toml`, `config/node_config.toml`, and Rust defaults point to `21`.
- [ ] Resolve the port mismatch: `config/genesis.json` uses `48650/58650` for RPC/WS while `config/network-config.toml` uses `48638/58638`.
- [ ] Resolve endpoint naming mismatch: `config/genesis.json` points to `testbeta-rpc` / `testbeta-ws`, while current network docs and config point to `testbeta-core-rpc` / `testbeta-core-ws`.
- [ ] Replace the checked-in genesis timestamp `2026-01-01T00:00:00Z` with a real ceremony-derived timestamp at final freeze.
- [ ] Resolve network identity mismatch: the checked-in genesis uses `synergy-testnet-beta`, while the generator emits `synergy-testbeta-closed-001`.
- [ ] Resolve bootnode mismatch: the checked-in genesis lists 3 bootnodes, while the generator currently derives only `machine-01` and `machine-02`.
- [ ] Decide whether `dynamic_validator_registration = true` is acceptable for mainnet, or whether validator admission must become governance-gated and ceremony-only.
- [ ] Align governance addresses and treasury addresses between the checked-in genesis and the generator script.
- [ ] Ensure the generator emits every required allocation type that the checked-in genesis expects, including fee-collector and stake-vault records if those are still protocol requirements.

### Genesis specification freeze

- [ ] Write a normative genesis spec that defines every field in `config/genesis.json`, its type, source of truth, and approval owner.
- [ ] Freeze the final `chain_id`, `network_id`, `network_name`, binary name, token symbol, decimals, and address prefixes.
- [ ] Freeze the final validator cap, minimum validator count, epoch length, block time, quorum threshold, slashing rules, and min-stake amount.
- [ ] Decide whether the target launch supports dynamic validator registration at genesis plus post-launch, or genesis-only validator admission until governance opens the validator set.
- [ ] Freeze the final system accounts: treasury, rewards pool, fee collector, burn address, stake vault, DAO address, foundation/devops wallet, faucet if any, and emergency council addresses.
- [ ] Produce a ledger worksheet that sums every allocation and independently proves `total_supply`.
- [ ] Freeze validator metadata rules: name format, identity string, website, security contact, validator class, commission rate, min self-delegation, and delegation caps.
- [ ] Decide whether placeholder `synergy.local` metadata is acceptable in any environment beyond rehearsal; if not, replace every placeholder with real production metadata.
- [ ] Define the exact genesis hash publication process and where the public will verify it.

### Genesis input data preparation

- [ ] Freeze the authoritative node inventory file and stop editing it once ceremony prep begins.
- [ ] Freeze the authoritative node-address export and verify every machine ID maps to the expected on-chain address.
- [ ] Regenerate validator keys and identity files from a controlled, documented process rather than relying on stale local artifacts.
- [ ] Verify every `public_key_file` referenced by genesis exists, is readable, and corresponds to the intended validator address.
- [ ] Verify every genesis validator address also appears in `genesis_allocations` with the correct self-stake amount.
- [ ] Verify every validator allocation amount equals the protocol’s min-stake requirement or the intended higher amount.
- [ ] Verify every special-purpose wallet address uses the correct prefix and passes checksum validation.
- [ ] Document whether any launch wallets are hot, warm, or cold, and who controls each one.
- [ ] Remove any accidental dependence on machine-local absolute paths or untracked key directories.

### Genesis ceremony design

- [ ] Define ceremony roles: coordinator, offline key custodians, independent verifier, release manager, scribe, observer, and final signer.
- [ ] Decide which materials must be generated offline and which may be generated on a connected staging machine.
- [ ] Require a clean commit SHA for the ceremony build and tag it before any artifact generation begins.
- [ ] Build the genesis generator and node binaries from the exact tagged commit on at least two independent machines.
- [ ] Run the genesis generator independently on at least two machines and compare byte-for-byte output.
- [ ] Compute and record SHA-256 hashes for `genesis.json`, validator inventory, address export, bootstrap bundles, and release manifests.
- [ ] Produce a signed ceremony manifest that records commit SHA, generator version, timestamp, participants, and resulting hashes.
- [ ] Record the exact UTC activation time and the allowed tolerance window.
- [ ] Define stop criteria for aborting the ceremony if hashes, counts, addresses, or supply totals do not match.
- [ ] Rehearse the ceremony at least once on a disposable pre-production network.

### Bootstrap topology and peer discovery

- [ ] Freeze the final number of bootnodes, seed services, validators, RPC gateways, archive nodes, observer nodes, and explorer/indexer nodes for launch.
- [ ] Verify the final topology against `testbeta/runtime/node-inventory.csv` and update inventory if reality changed.
- [ ] Decide whether bootnodes are pure `bootstrap_only` infrastructure or also consensus participants; document and enforce the answer.
- [ ] Verify all bootnode DNS names, management hosts, public IPs, and public P2P ports.
- [ ] Verify seed services expose `/healthz`, `/peers`, `/peer-list.json`, and `/peers/register` exactly as the control plane expects.
- [ ] Verify `_dnsaddr` bootstrap records if they are part of the final discovery design.
- [ ] Prove that a fresh node can join using hardcoded bootnodes only.
- [ ] Prove that a fresh node can join using DNS bootstrap only.
- [ ] Prove that a fresh node can join using seed-service fallback when DNS records are unavailable.
- [ ] Test bootstrap from at least three external networks and record time-to-first-peer and time-to-sync.

### Day-0 network start procedure

- [ ] Write the exact launch runbook from T-minus 24 hours to T-plus 2 hours.
- [ ] Start bootnodes first and verify stable peer acceptance before any validators come online.
- [ ] Start seed services second and verify peer-registration paths.
- [ ] Start the minimum genesis validator set and verify all are approved, staked, and visible to consensus.
- [ ] Confirm the active validator count meets the enforced `min_validators` threshold before expecting block production.
- [ ] Verify that block height advances on at least three independent nodes.
- [ ] Verify that all validator nodes report the same genesis hash, chain ID, and latest finalized height.
- [ ] Verify that no node exposes unsafe public RPC bindings directly to the internet.
- [ ] Bring up public RPC and WS proxies only after local node health is confirmed.
- [ ] Open explorer and wallet public surfaces only after chain liveness, RPC correctness, and indexing freshness are confirmed.

### Consensus and protocol correctness

- [ ] Add deterministic tests that verify genesis initialization, validator registration, and self-stake behavior from the final genesis file.
- [ ] Add tests that fail if validator count or staking rules diverge from the final genesis parameters.
- [ ] Prove the `min_validators` threshold behaves correctly under node loss and restart scenarios.
- [ ] Add tests for slashing triggers, downtime thresholds, and double-sign handling.
- [ ] Add deterministic tests for VRF proposer selection and epoch rotation logic.
- [ ] Validate reward distribution math against the intended tokenomics.
- [ ] Validate cluster assignment and synergy scoring behavior under the final validator topology.
- [ ] Add replay, malformed block, and cross-network genesis mismatch tests to sync and P2P logic.
- [ ] Run multi-node determinism tests from genesis with repeated resets and compare resulting states.
- [ ] Decide what parts of the protocol are still “testbeta bootstrap” behavior and must be removed before real economics.

### Storage, state, and recovery

- [ ] Document the RocksDB schema, pruning policy, and recovery expectations for validator, archive, and RPC roles.
- [ ] Prove that nodes can recover from an unclean shutdown without state divergence.
- [ ] Add snapshot and state-sync validation for archive and RPC roles.
- [ ] Test full replay from genesis to current head on clean machines.
- [ ] Define database migration policy for any on-disk format changes after launch.
- [ ] Create operator procedures for backup, restore, snapshot seeding, and disk-pressure handling.

### Security implementation program

- [ ] Turn `docs/security-implementation-program.md` and `docs/crypto-theft-solutions-implementation-matrix.md` into explicit release gates rather than advisory docs.
- [ ] Confirm which of the 14 attack categories are fully implemented at protocol, SynQ, and wallet layers and which remain open.
- [ ] Add CI gates for requirements marked fail-closed in the implementation program.
- [ ] Complete EIP-7702 protections if EVM compatibility remains in scope for launch.
- [ ] Review all consensus, RPC, and operator paths for secret leakage into logs.
- [ ] Review validator key lifecycle, rotation, backup, revocation, and emergency replacement procedures.
- [ ] Decide whether any genesis or bootstrap keys were ever handled on insecure developer machines and rotate them if so.

### Public endpoint and infrastructure alignment

- [ ] Freeze the canonical hostname family for the target environment and remove conflicting old hostnames from docs and configs.
- [ ] Regenerate TLS certificates only for the final hostname set.
- [ ] Verify reverse-proxy routing for core RPC, core WS, REST/API, explorer UI, explorer API, and any wallet or SXCP APIs.
- [ ] Add health, readiness, version, and build-info endpoints for every public-facing service.
- [ ] Ensure rate limiting, auth boundaries, and CORS rules are explicit rather than permissive defaults.
- [ ] Verify public status checks from outside the hosting provider network.

### Tooling, scripts, and operator docs

- [ ] Review every script under `scripts/` and `scripts/testbeta/` for hardcoded ports, stale hostnames, stale environment names, and destructive behavior.
- [ ] Consolidate duplicate start/stop/reset scripts so operators have one authoritative flow.
- [ ] Verify `generate-node-keys.sh`, `render-configs.sh`, `generate-monitor-hosts-env.sh`, and `generate-testnet-beta-genesis.sh` are consistent with each other.
- [ ] Turn manual runbooks into automated validation commands where possible.
- [ ] Publish a single operator launch runbook that supersedes fragmented older docs.
- [ ] Add a post-launch maintenance runbook for validator replacement, node expansion, and emergency bootstrap replacement.

## 2. Node Control Panel App

Repo reviewed: `synergy-testnet-beta/node-control-panel/`

### Product boundary and role support

- [ ] Freeze the list of node roles the app will support in production and mark every other role internal-only or unsupported.
- [ ] Remove remaining generic-node assumptions wherever specialized role binaries are required.
- [ ] Verify every `NodeType` maps to the correct compiled binary on Windows, macOS, and Linux.
- [ ] Verify every role writes correct `identity.role`, `role.compiled_profile`, and bootstrap settings into generated configs.
- [ ] Verify every role starts only its intended local services and never inherits validator-only powers accidentally.
- [ ] Verify service-only roles cannot acquire treasury, governance, or emergency authorities through misconfiguration.
- [ ] Freeze the on-disk config schema and implement versioned migrations for future app updates.

### Desktop runtime and local agent

- [ ] Verify the bundled local agent installs correctly on clean Windows, macOS, and Linux machines.
- [ ] Verify the agent survives reboot, app restart, and app upgrade.
- [ ] Verify the app cleanly falls back from local agent to SSH only when explicitly designed to do so.
- [ ] Minimize privileged operations and document exactly when elevation is required.
- [ ] Add integrity checking for all bundled helper binaries and agent payloads before execution.
- [ ] Confirm local logs, crash reports, and diagnostics never contain private keys, seed phrases, or node secret material.

### Provisioning and lifecycle operations

- [ ] Test setup, install, bootstrap, start, stop, restart, status, logs, reset, and uninstall for every supported role.
- [ ] Add recovery handling for interrupted downloads, partial installs, corrupted configs, and missing inventory files.
- [ ] Verify that network resets and reindex actions are gated, explained, and reversible where possible.
- [ ] Ensure that every destructive action has a preflight summary and confirmation screen.
- [ ] Verify the app can recover from stale PIDs, stale sockets, and zombie child processes.
- [ ] Verify multi-node views remain accurate when some nodes are offline, lagging, or unreachable.

### Build, packaging, and updates

- [ ] Produce signed release artifacts for macOS, Windows, and Linux from CI rather than local-only builds.
- [ ] Verify `electron-builder` outputs are reproducible and include all required sidecars and docs.
- [ ] Complete code signing, notarization, and Windows signing before public operator distribution.
- [ ] Publish a production artifact manifest with checksums for the desktop app and role binaries.
- [ ] Verify updater metadata generation and distribution end-to-end.
- [ ] Verify in-app updates never overwrite active node state unsafely.
- [ ] Test upgrade from at least one previous public version on all supported OSes.

### UX, observability, and supportability

- [ ] Ensure the dashboard exposes role, binary name, version, peer count, sync height, latest error, and last successful contact.
- [ ] Ensure operator-facing language is unambiguous and does not expose internal-only concepts without explanation.
- [ ] Add explicit UI labels for unsupported, experimental, and production-ready roles.
- [ ] Add one-click diagnostic bundle generation that redacts sensitive material.
- [ ] Add clear user-facing remediation text for bootstrap failures, DNS failures, checksum failures, and role mismatches.
- [ ] Add support playbooks for the top 20 operator failure scenarios.

### Security and supply chain

- [ ] Verify checksum and signature validation before any downloaded binary executes.
- [ ] Verify update and download URLs enforce TLS and expected hostnames.
- [ ] Review Electron security posture: IPC exposure, preload boundaries, remote content loading, update channel trust, and file-system access.
- [ ] Review Rust `control-service` RPC and command surfaces for command injection or unsafe path handling.
- [ ] Define an emergency disable path for a compromised or bad release.

### QA

- [ ] Build an automated smoke suite for clean-install provisioning on all supported desktop OSes.
- [ ] Add role-mapping regression tests for every supported node type.
- [ ] Add end-to-end tests that provision a node, join the network, and surface healthy status in the UI.
- [ ] Add recovery tests for failed downloads, invalid manifests, missing DNS, and corrupted config files.
- [ ] Add performance tests for large inventories and multi-node monitoring sessions.
- [ ] Run external-operator beta tests before any production operator rollout.

## 3. Synergy Atlas

Repo reviewed: `explorer-app/`

### Current-state alignment

- [ ] Rename or re-scope any remaining legacy beta naming in code, env examples, PM2 configs, and nginx configs if Atlas is intended to serve testnet/mainnet.
- [ ] Freeze the production deployment model for Atlas: UI, backend API, indexer, database, cache, reverse proxy, and archive/RPC dependencies.
- [ ] Define whether Atlas is a standalone product repo or a subcomponent owned by protocol/infrastructure.
- [ ] Publish a single environment matrix for testnet-beta, testnet, mainnet-beta, and mainnet hostnames and ports.

### Core explorer completeness

- [ ] Finish address analytics beyond basic activity history.
- [ ] Finish validator analytics: rewards, uptime, participation, committee changes, missed blocks, and stake history.
- [ ] Add a canonical search endpoint that classifies hash, block height, address, validator, and transaction in one pass.
- [ ] Add sorting, filtering, export, and query parameter persistence to list pages.
- [ ] Add reliable loading, empty, error, and partial-data states across all views.
- [ ] Add API pagination consistency and response schema guarantees.

### DAG-specific Atlas work

- [ ] Decide the exact DAG model Atlas must represent: block DAG, transaction dependency DAG, committee DAG, or confirmation DAG.
- [ ] Extend the indexer schema to store parent-set relationships rather than assuming strictly linear previous-block history.
- [ ] Add DAG metadata fields to block records: parent hashes, branch ID, merge depth, finality state, confidence, and topological order.
- [ ] Add APIs to query ancestors, descendants, competing branches, and finalized branch selection.
- [ ] Add a DAG visualization component for block lineage and branch convergence.
- [ ] Add transaction-page support for dependency graphs if transaction DAG semantics are part of protocol behavior.
- [ ] Add finality indicators that distinguish “seen”, “branch candidate”, “accepted”, “checkpointed”, and “finalized”.
- [ ] Add reorg/branch event retention and UI surfacing instead of silently collapsing history.
- [ ] Define how DAG data affects validator analytics, fork accountability, and explorer search results.
- [ ] Add replay tests proving Atlas renders the same DAG result from a clean reindex.

### Indexer hardening

- [ ] Add durable checkpointing with explicit processed/finalized block markers.
- [ ] Add idempotent ingestion guarantees for blocks, transactions, validator snapshots, and address activity.
- [ ] Add reorg handling, retry strategy, and bounded replay logic.
- [ ] Add rate limiting and parallelization controls for RPC backfills.
- [ ] Add metrics for lag, RPC error rate, ingest throughput, reorg count, and database write latency.
- [ ] Add backfill controls for genesis-to-head, bounded ranges, and partial rebuilds.
- [ ] Add archive-node dependency checks so Atlas can fail loudly instead of indexing partial state.

### Backend hardening

- [ ] Add request validation and OpenAPI coverage for every public route.
- [ ] Add rate limiting, cache policy, and timeout policy per endpoint family.
- [ ] Add auth or admin protections for any reset/reindex endpoints.
- [ ] Add structured error codes rather than generic 500-class responses.
- [ ] Add compatibility tests against the protocol RPC surface that Atlas depends on.
- [ ] Add contract tests so explorer API output remains stable for wallets and external integrators.

### Frontend quality

- [ ] Add responsive passes for desktop, tablet, and mobile layouts.
- [ ] Add accessibility review for table navigation, contrast, focus order, and graph visualization.
- [ ] Add search-result routing that never lands users on ambiguous or empty detail pages.
- [ ] Add copy/share/download patterns for hashes, addresses, and raw JSON.
- [ ] Add performance budgets for initial load, route changes, and graph-heavy pages.

### Operations and reliability

- [ ] Add backup, restore, and reindex runbooks for Postgres and Atlas services.
- [ ] Define data retention policy for historical blocks, validator snapshots, and analytics aggregations.
- [ ] Add dashboards and alerts for API uptime, indexer lag, failed migrations, and stale explorer freshness.
- [ ] Add production deployment automation for nginx, PM2 or service manager, env injection, and health checks.
- [ ] Run failure drills for RPC outage, Postgres outage, partial reindex, and disk-full scenarios.

## 4. Synergy Wallet

Repo reviewed: `wallet-app/`

### Product architecture decisions

- [ ] Decide the authoritative wallet architecture for production across web, desktop, mobile, and extension.
- [ ] Eliminate or formalize the current web-wallet dependency on a local helper API plus locally compiled binaries from another repo.
- [ ] Decide whether transaction signing for web must remain local-only in browser/WASM, move to native helpers, or be limited to desktop/mobile/extension.
- [ ] Define which chains are supported at launch versus merely scaffolded in the codebase.
- [ ] Freeze the policy-engine model for approvals, spending limits, timelocks, delegated actions, and recovery.

### Key management and cryptography

- [ ] Complete a production threat model for FN-DSA, ML-KEM, seed phrase handling, envelope encryption, device secret usage, and recovery workflows.
- [ ] Verify that no production path derives PQC signing keys from mnemonic when the security model forbids it.
- [ ] Define cross-platform secure storage rules: browser, Tauri desktop, iOS, Android, and browser extension.
- [ ] Use OS-native keystores or secure enclave features wherever available and document fallback behavior.
- [ ] Define key rotation, account export, account import, and secure wipe procedures.
- [ ] Define recovery flow ownership for lost-device, forgotten-passcode, and compromised-device cases.
- [ ] Verify the wallet never logs or crashes with plaintext secrets.
- [ ] Complete side-channel and memory-residency review for the PQC packages and any WASM/native bridges.

### Platform completeness

- [ ] Bring the web app to a production-grade state with full tests, release build validation, and CORS-safe backend/API integration.
- [ ] Bring the desktop app from scaffolded Tauri shell to a fully functional signed release.
- [ ] Bring the mobile app from React Native starter + current features to a production distribution pipeline for iOS and Android.
- [ ] Bring the browser extension to a production-grade extension with permissions review, signing, and store packaging.
- [ ] Define a platform-parity matrix so users know which features exist on which platform.

### Core wallet functionality

- [ ] Verify wallet creation, import, backup, recovery, send, receive, history, address book, and network switching across all supported platforms.
- [ ] Add transaction simulation, fee estimation, nonce handling, replace/cancel behavior, and pending-state reconciliation.
- [ ] Define token discovery, token registry, hidden-token handling, and scam-token filtering.
- [ ] Add multisig support only if the protocol and UX are genuinely ready; otherwise defer it explicitly.
- [ ] Define SXCP and cross-chain UX only if the relay product is launch-ready; otherwise keep it hidden.
- [ ] Add account-lock, session-lock, idle timeout, and biometric unlock policy where supported.

### App security and trust UX

- [ ] Build clear signing surfaces that show exact destination, asset, amount, fee, network, scope, delegation risk, and contract warnings.
- [ ] Add anti-phishing protections for address poisoning, clipboard hijacking, malformed approvals, and blind signing.
- [ ] Implement allowlist, warning, and blocklist infrastructure for known bad addresses and malicious contracts.
- [ ] Add transaction manifest rendering for every risky action, not just basic transfers.
- [ ] Add explicit handling for EIP-7702, permit-style approvals, multicall, delegatecall, and proxy interactions if EVM support stays in scope.
- [ ] Review browser extension content-script boundaries, permissions, messaging, and site-connection approval model.

### Testing and quality

- [ ] Replace placeholder “tests to be implemented” scripts in shared packages with real unit and integration tests.
- [ ] Add vault lifecycle tests for create, lock, unlock, backup, restore, rotate, and wipe.
- [ ] Add transaction signing fixture tests for all supported networks and asset types.
- [ ] Add mobile device tests for backgrounding, biometric changes, push-notification handling, and keychain failures.
- [ ] Add desktop tests for updater, filesystem permissions, and native-helper availability.
- [ ] Add browser extension tests for install, connect, permission prompts, and hostile webpage interactions.
- [ ] Add end-to-end tests that send real transactions on a staging network and verify Explorer visibility.

### Release and operations

- [ ] Create a release pipeline for web deploys, desktop installers, mobile app archives, and extension bundles.
- [ ] Add code signing, notarization, App Store/TestFlight, Play Store, and browser store packaging requirements.
- [ ] Add crash reporting and telemetry with strict secret redaction.
- [ ] Publish a wallet security whitepaper, supported-network matrix, and user recovery guide.
- [ ] Define a support policy for lost access, compromised device, phishing report, and emergency wallet hotfixes.

## 5. Synergy Relay

Current workspace state: no standalone product repo found; relay primitives exist in `synergy-testnet-beta/services/sxcp-relayer/` and protocol role binaries.

### Product and repo bootstrap

- [ ] Create a dedicated `synergy-relay` product repo or formally decide to keep Relay embedded in the protocol repo.
- [ ] Define the Relay product boundary: operator daemon only, user-facing DEX, intent aggregator, routing API, and settlement UI.
- [ ] Define the chains supported at launch and which are deferred.
- [ ] Define the business model: protocol-owned relayers, permissioned partner relayers, or open relayer market.
- [ ] Define the user-facing brand, endpoint family, environment layout, and ownership model.

### Protocol and settlement model

- [ ] Freeze the SXCP intent format, attestation format, quorum rules, and timeout semantics.
- [ ] Define how relayer quorum, signature thresholds, slashing, and dispute resolution work on mainnet.
- [ ] Define how intent cancellation, expiry, replay protection, and duplicate settlement are handled.
- [ ] Define finality assumptions per connected chain and how Relay waits for enough confirmations.
- [ ] Define how failed destination-chain execution is reported and recovered.
- [ ] Define exact on-chain contracts required on each external chain and audit them.

### Existing daemon hardening

- [ ] Review the current `services/sxcp-relayer` daemon and decide what can be reused versus rewritten.
- [ ] Replace SQLite-only local durability assumptions if HA or cluster failover is required.
- [ ] Add deterministic checkpointing, replay protection, and idempotent settlement guarantees.
- [ ] Add Prometheus metrics, structured logs, alerting, and dashboard coverage.
- [ ] Add signer-isolation strategy so PQC private keys are never left as flat files in ad hoc locations.
- [ ] Add quorum peer authentication, peer allowlisting, and anti-spam protections.

### DEX and routing product

- [ ] Decide whether Synergy Relay is only a relayer service or also the multi-chain DEX frontend/backend.
- [ ] Define swap routing, quote generation, fee model, slippage model, liquidity source integration, and price-oracle dependencies.
- [ ] Decide whether liquidity is protocol-native AMM, third-party DEX aggregation, RFQ, or hybrid.
- [ ] Build a quote engine and route simulator that exposes exact execution risk.
- [ ] Add approval management, allowance revocation, and cross-chain risk disclosures in the user UI.
- [ ] Add settlement receipts and full intent audit trails visible to users and operators.

### Security, compliance, and operations

- [ ] Threat-model bridge, relayer, quorum, and settlement abuse paths before launch.
- [ ] Commission independent audit for SXCP contracts, relayer daemon, and quote/settlement logic.
- [ ] Define chain outage, RPC degradation, and relayer quorum loss handling.
- [ ] Define KYC/AML and jurisdiction posture if Relay becomes a public DEX product.
- [ ] Add runbooks for stuck intents, partial fills, gas spikes, malicious counterparties, and signer compromise.
- [ ] Launch Relay only after Explorer and Wallet can surface relay intents and statuses correctly.

## 6. Synergy Spark

Repo reviewed: `meme-launchpad/`

### Product definition and scope control

- [ ] Decide whether Spark launches first as a token launchpad, a social platform, an NFT marketplace, or a smaller subset.
- [ ] Remove or explicitly defer under-construction sections that cannot be supported at launch.
- [ ] Define which features are core revenue features versus experimental community features.
- [ ] Define whether Spark is Synergy-native only or multi-chain at launch.

### Smart contracts and token-launch safety

- [ ] Audit all launch, factory, staking, presale, and marketplace contracts before production deployment.
- [ ] Freeze the contract upgrade model and ownership controls.
- [ ] Define anti-rug protections, treasury controls, fee routing, and launch defaults.
- [ ] Define whether created tokens are standard templates only or allow user-supplied custom logic.
- [ ] Add on-chain and off-chain checks for supply caps, mint authority, transfer-tax disclosures, and ownership renounce flows.
- [ ] Add launch risk disclosures and standardized token metadata requirements.

### Backend and data layer

- [ ] Verify the backend API actually implements the routes documented in `backend/README.md`.
- [ ] Freeze the production database choice, schema migration path, and backup/restore plan.
- [ ] Add auth hardening, wallet-signature auth replay protection, session controls, and abuse throttling.
- [ ] Add moderation tooling, audit logs, and operator admin surfaces before public social features launch.
- [ ] Define file upload storage, malware scanning, size limits, retention, and takedown handling.
- [ ] Add production observability and error budgeting for chat, streams, token launches, and AI endpoints.

### Social, chat, and livestream safety

- [ ] Define moderation policy, reporting tools, ban workflow, and content takedown SLAs.
- [ ] Add spam prevention, bot defense, and abuse scoring for chat and social features.
- [ ] Define livestream moderation, clip retention, and illegal-content handling before going live.
- [ ] Add user-blocking, muting, reputation, and community-safety controls.
- [ ] Decide whether direct messaging should be public at launch or deferred.

### AI assistant

- [ ] Define what Jarvis is allowed to generate and what it must never do.
- [ ] Add prompt-safety and moderation for launch guidance, NFT prompts, and market-analysis features.
- [ ] Add clear disclosure that AI guidance is not legal, financial, or investment advice.
- [ ] Add cost controls, rate limits, and observability for AI usage.

### UX, analytics, and launch operations

- [ ] Add production analytics for launch conversions, failed token deployments, suspicious activity, and user retention.
- [ ] Add creator onboarding, buyer onboarding, and launch checklists.
- [ ] Add legal acceptance records, consent tracking, and jurisdictional gating if required.
- [ ] Add full staging environment with seeded data and rehearsed launch events.
- [ ] Add incident playbooks for failed token launches, broken presales, and exploited contracts.

## 7. Synergy Keystone

Current workspace state: no repo/folder found. Treat as not started.

### Product definition

- [ ] Create the repo, architecture doc, and initial product spec before implementation starts.
- [ ] Define whether Keystone is only a governance UI or a full governance stack including proposal indexing, treasury ops, and delegate management.
- [ ] Define target users: token holders, delegates, councils, treasury operators, and protocol admins.
- [ ] Define the governance model: token voting, delegated voting, council review, multi-house governance, or hybrid.

### On-chain governance integration

- [ ] Freeze the proposal lifecycle: draft, submission, deposit, review, voting, timelock, execution, cancellation, emergency halt.
- [ ] Define the canonical on-chain governance contracts/modules and their upgrade path.
- [ ] Define quorum, threshold, veto, abstain, delegation, and emergency powers.
- [ ] Define how treasury actions are proposed, approved, executed, and audited.
- [ ] Define how governance events are indexed and exposed to Explorer and Wallet.

### Application architecture

- [ ] Choose frontend stack, backend/indexer needs, database, and hosting model.
- [ ] Build a proposal indexer and event ingestion service if governance state is not trivial to fetch live.
- [ ] Build a secure transaction-builder flow for proposal creation, voting, delegation, and execution.
- [ ] Add role-based operator surfaces for treasury, security council, and governance admin functions.
- [ ] Add notifications, reminders, and proposal-state subscriptions.

### Governance UX and trust

- [ ] Add human-readable proposal rendering, diff views, and linked on-chain actions.
- [ ] Add wallet signing previews that show exact governance actions.
- [ ] Add delegate discovery, delegate profiles, and voting history.
- [ ] Add treasury dashboards with inflows, outflows, budget categories, and execution status.
- [ ] Add archive/history views for passed, failed, queued, executed, and canceled proposals.

### Security and operations

- [ ] Threat-model governance takeover, malicious proposals, treasury-drain proposals, and delegate abuse.
- [ ] Add simulation and static checks for executable proposal payloads before users sign them.
- [ ] Commission independent audit for governance execution paths.
- [ ] Define emergency pause, veto, and recovery paths.
- [ ] Add governance incident response and public communication templates.

## 8. Synergy Learn

Current workspace state: no repo/folder found. Treat as not started.

### Product definition

- [ ] Create the repo, architecture doc, and MVP feature set.
- [ ] Define whether Learn is only a knowledge base, only an LMS, or a combined product.
- [ ] Define target audiences: network users, validators, developers, smart-contract authors, and new coders.
- [ ] Define what content is free, gated, certification-based, or community-contributed.

### Knowledge base foundation

- [ ] Decide CMS/docs platform, content source format, versioning model, and publishing workflow.
- [ ] Build docs IA for protocol, wallet, explorer, governance, relay, launchpad, and developer education.
- [ ] Add search, tagging, version selectors, and cross-linking to product docs.
- [ ] Define content ownership and review workflow per product team.
- [ ] Add docs analytics for search failures, abandoned pages, and outdated content.

### LMS foundation

- [ ] Decide whether courses require accounts, progress sync, certificates, quizzes, and cohort features.
- [ ] Build curriculum model: lessons, modules, quizzes, labs, assignments, and completion states.
- [ ] Define coding-lab infrastructure if coding lessons are in scope.
- [ ] Add authoring tools and editorial review workflow for technical lessons.
- [ ] Add learner progress dashboards and admin reporting.

### Developer education

- [ ] Build beginner-to-advanced tracks for wallet usage, validator operations, protocol development, and SynQ development.
- [ ] Define code examples, sandbox environments, and repo-backed exercises.
- [ ] Add automated validation for code-lab answers if interactive coding is offered.
- [ ] Align learning content with the actual current product behavior so docs do not teach dead paths.

### Accessibility, quality, and operations

- [ ] Build for accessibility from the start: captions, transcripts, keyboard support, contrast, and screen-reader semantics.
- [ ] Add moderation and abuse controls if community comments, submissions, or forums are included.
- [ ] Add backup, export, and portability for course content and learner records.
- [ ] Define uptime expectations, content rollback, and emergency notice mechanisms.
- [ ] Create a docs/content freshness program so outdated material is flagged automatically.

## Suggested Execution Order

1. Freeze `Synergy-Testnet-Beta` genesis, topology, ceremony, and environment naming.
2. Finish `Node Control Panel App` role hardening, packaging, and operator validation.
3. Harden `Synergy Atlas` enough to index and display the live network, including DAG support if DAG is a protocol truth.
4. Re-architecture and harden `Synergy Wallet` for production key management and release pipelines.
5. Decide whether `Synergy Relay` is a launch product or a post-mainnet product; do not ship a half-owned bridge/DEX.
6. Harden `Synergy Spark` only after wallet, explorer, and relay dependencies are real.
7. Build `Synergy Keystone` after governance contracts and treasury policy are frozen.
8. Build `Synergy Learn` after product surfaces and docs sources are stable enough to teach accurately.
