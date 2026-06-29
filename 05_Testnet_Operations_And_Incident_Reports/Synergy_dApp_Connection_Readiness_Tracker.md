# Synergy dApp Connection Readiness Tracker

**As of 2026-04-27 on `Testnet-Beta` (chain ID `0x52acf` / decimal `338639`).**

This file is now split into:

- **Section 0.1**: what is verified live and ready to use now.
- **Section 0.2**: what still must be completed before developer onboarding is truly ready.
- **Sections 1+**: detailed status inventories, not checklists.

Priority labels:

- **P0** — hard blocker for real dApp usability or safe public rollout.
- **P1** — should be completed before broad public developer rollout.
- **P2** — polish, ecosystem tooling, or follow-on operational hardening.

---

## 0. Current action checklists

### 0.1 Verified live and ready now

- [x] `synergy_simulateTransaction` is implemented and live on the public RPC gateway.
- [x] `synergy_subscribe` / `synergy_unsubscribe` are implemented and working over WebSocket.
- [x] `synergy_getAccountNonce` and `synergy_getAccountAuthNonce` are implemented and live.
- [x] `synergy_sendTransaction` accepts the spec-style envelope fields (`from`, `to`, `value`, `gasLimit`, `maxFee`, `chainId`, `signatureAlgorithm`) while preserving legacy compatibility.
- [x] `synergy_estimateGas` returns the structured response `{ gas, safeFee, maxFee, warnings[] }`.
- [x] `synergy_getChainId` returns hex (`0x52acf` on Testnet-Beta).
- [x] HTTP JSON-RPC enforces `Content-Type: application/json`.
- [x] JSON-RPC batch requests are accepted.
- [x] HTTP CORS and `OPTIONS` preflight handling exist in the node RPC surface.
- [x] Public HTTPS RPC is verified live at `https://testnet-core-rpc.synergy-network.io`.
- [x] Public WSS subscriptions are verified live at `wss://testnet-core-ws.synergy-network.io`.
- [x] Public RPC and WSS now enforce Security-Spec exposure tiers so authority-plane and non-public write methods are blocked on the unauthenticated public edge.

### 0.2 Remaining blockers and required follow-up

- [ ] Reduce sustained validator block times below `5s` by eliminating the remaining leader-proposal timeout path and then retuning consensus timeouts safely.
- [ ] Implement `synergy_resolveSynID` / `synergy_reverseResolveSynID`.
- [ ] Implement `synergy_revokeAllApprovals` / `synergy_getApprovals`.
- [x] Enforce exposure-profile / auth-tier gating per RPC method as required by the Security Specification.
- [ ] Finish wallet extension provider injection, connect/sign prompts, approval UX, and recipient-tamper enforcement.
- [ ] Publish the official SDK / address / contracts developer packages.
- [ ] Finish public developer onboarding artifacts so external dApps can connect without hand-built glue code.
- [x] Rebuild observer/Grafana dashboards so they use the live `synergy_*` series, remove duplicate and empty panels, explain each chart clearly, and add the missing fleet / validator / RPC / cluster metrics.

### 0.3 Production ingress note

- Raw custom ports on the RPC host (`5646` / `5666`) are still filtered upstream of the VPS.
- That is **not** blocking browser dApps today because the correct production ingress is the verified TLS surface on `443`:
  - `https://testnet-core-rpc.synergy-network.io`
  - `wss://testnet-core-ws.synergy-network.io`

---

## 1. Node RPC and WebSocket status inventory

| Capability | Status | Priority | Notes |
|---|---|---|---|
| `synergy_simulateTransaction` | **Live** | P0 | Implemented and verified on the public RPC gateway. |
| `synergy_subscribe` / `synergy_unsubscribe` | **Live** | P0 | Implemented and verified over WSS. |
| `synergy_getAccountNonce` / `synergy_getAccountAuthNonce` | **Live** | P0 | Available for wallet and dApp nonce handling. |
| `synergy_sendTransaction` spec envelope compatibility | **Live** | P0 | Accepts current spec-style fields while preserving legacy compatibility. |
| `synergy_estimateGas` structured response | **Live** | P0 | Returns `gas`, `safeFee`, `maxFee`, and warnings. |
| `synergy_getChainId` hex response | **Live** | P0 | Returns `0x52acf`. |
| HTTP JSON-RPC batch requests | **Live** | P1 | Array payloads are accepted. |
| `Content-Type: application/json` enforcement | **Live** | P1 | Bad content types are rejected. |
| HTTP CORS / preflight handling | **Live** | P0 | Implemented in the node and verified alongside the public exposure-policy gate. |
| Public HTTPS RPC ingress | **Live** | P0 | Verified at `testnet-core-rpc.synergy-network.io`. |
| Public WSS ingress | **Live** | P0 | Verified at `testnet-core-ws.synergy-network.io`. |
| `synergy_resolveSynID` | **Missing** | P0 | Required so dApps do not have to hand-roll `synergy_call` into `Identity.sol`. |
| `synergy_reverseResolveSynID` | **Missing** | P1 | Needed for address-to-name display parity. |
| `synergy_revokeAllApprovals` | **Missing** | P0 | Required by the Security Specification approval model. |
| `synergy_getApprovals` | **Missing** | P0 | Required for wallet approval review and revocation UX. |
| `synergy_getAddressBook` | **Missing** | P1 | No native account address-book RPC surface yet. |
| `synergy_getTransactionsByAddress` | **Missing** | P1 | Explorer has indexed views; node RPC fallback still missing. |
| `synergy_newFilter` / `synergy_getFilterChanges` / `synergy_uninstallFilter` | **Missing** | P1 | Poll-based filter path still absent. |
| `synergy_getAuthorizationHistory` | **Missing** | P1 | Audit-history RPC still absent. |
| `synergy_getTransactionTraceHints` | **Missing** | P1 | Still needed for richer wallet preview UX. |
| AIVM RPC method family | **Missing** | P1 | Still commented out / disabled. |
| Operator-only methods (`synergy_mine`, `synergy_setAccountBalance`, `synergy_resetChainHead`) | **Missing** | P2 | Dev/operator surface still not shipped. |
| `synergy_call` full spec parity | **Partial** | P1 | Present, but still lacks the broader `stateOverride` / block-tag behavior called for in the spec. |
| `synergy_getLogs` full filter object parity | **Partial** | P1 | Present, but still not at full filter parity with the spec. |
| `synergy_gasPrice` / `synergy_maxFeePerGas` / `synergy_maxPriorityFeePerGas` | **Partial** | P1 | Present, but still need a real fee oracle rather than placeholder/static behavior. |
| `synergy_getFeeHistory` | **Partial** | P1 | Present, but still needs real block-history aggregation. |
| Exposure-profile / auth-tier gating | **Live** | P0 | Public HTTPS/WSS now allow read and canonical client methods while denying authority-plane and non-public write methods on the unauthenticated public edge. |
| Structured Synergy JSON-RPC error subcodes and `data` hints | **Partial** | P0 | Some RPC normalization landed, but canonical `-320xx` coverage still needs to be finished and audited. |

---

## 2. Network stabilization inventory

| Area | Status | Priority | Notes |
|---|---|---|---|
| Minute-scale dual-quorum stall | **Fixed** | P0 | False self-conflict path was removed and local leaders now skip already-used rounds. |
| Healthy-validator steady-state block production | **Live** | P0 | Fresh live samples on `2026-04-26` put the public chain at about `4.4s/block` over `13` blocks in about `57s`, with all five validators reporting `sync_status: synced`. |
| Sustained block time below `5s` | **Not yet stable** | P0 | Current live samples on `2026-04-27` are still around `10s/block` (`10412 -> 10414` over `20s` on the public RPC path), and Atlas is reporting `avgBlockTimeSeconds = 8.458333333333334`. This is not yet at the requested target. |
| Validator automatic catch-up in normal runtime | **Fixed** | P0 | The behind-tip validator path now invokes `SyncManager`, requests overlapping block windows for reconciliation, and can roll back to a common ancestor before replaying forward. |
| `GenVal-02` divergent-tip recovery | **Fixed** | P0 | The validator was recovered from a divergent block `9608` tip to the live chain on the new runtime, then restarted through the control-panel-managed workspace and rejoined consensus. |
| Runtime / bundle checksum alignment | **Live** | P0 | Validators `1-5`, `Node-RPC`, `Node-EXP`, the nested control-panel bundle, and the local monitor workspace are now aligned on `858c5aa89696f4e6fb3483edd024eec3cdad8f097d326470eeeb424a6fd2d37c`. |

---

## 3. Wallet extension status inventory

| Capability | Status | Priority | Notes |
|---|---|---|---|
| `window.synergy` provider injection | **Missing** | P0 | dApps cannot reliably detect or connect to the wallet yet. |
| EIP-6963 provider announcement | **Missing** | P0 | Needed for modern multi-wallet discovery flows. |
| Connect-wallet approval flow | **Missing** | P0 | No production-ready connect prompt path yet. |
| Per-method sign / send / switch-network prompts | **Missing** | P0 | Still required. |
| Simulate-then-sign preview | **Missing** | P0 | Security Specification requires this before user confirmation. |
| Recipient-tamper detection | **Missing** | P0 | Security Specification requirement still open. |
| Scoped approvals storage and revoke UX | **Missing** | P0 | Depends on node approval RPC work too. |
| Type-0x04 / delegation rejection in wallet UX | **Missing** | P0 | Still must be enforced. |
| Origin allow-list / deny-list per account | **Missing** | P1 | Permission model not complete yet. |
| Chain-ID confirmation on first connect | **Missing** | P1 | Still needed for first-time trust confirmation. |
| Session tokens / reduced repeated prompts | **Missing** | P1 | Quality and usability requirement. |
| Watch-asset UX | **Missing** | P2 | Still follow-on wallet polish work. |

---

## 4. Developer package and contract inventory

| Deliverable | Status | Priority | Notes |
|---|---|---|---|
| `@synergy-network/address` | **Missing** | P0 | Browser-friendly address derivation and validation package still needed. |
| Published TypeScript types for all address variants | **Missing** | P0 | Still needed for dApp ergonomics. |
| `@synergy-network/contracts` ABI package | **Missing** | P0 | Canonical versioned ABI bundle still needs to be published. |
| Deployed-contract address registry by chain | **Missing** | P0 | Required for clean dApp integration. |
| Event-signature semantic map | **Missing** | P1 | Still needed for decoder / preview tooling. |
| `@synergy-network/sdk` | **Missing** | P0 | Official RPC/envelope/address helper package still absent. |
| `@synergy-network/wagmi-connector` | **Missing** | P1 | Still future compatibility work. |
| `@synergy-network/viem-transport` | **Missing** | P1 | Still future compatibility work. |
| Foundry / Hardhat plugins | **Missing** | P1 | Still needed for contract developer onboarding. |
| `synergy-cli` | **Missing** | P2 | Helpful, but not a prerequisite to complete the node/wallet surface. |

---

## 5. Explorer and public read-surface inventory

| Capability | Status | Priority | Notes |
|---|---|---|---|
| Public explorer REST read API | **Partial** | P0 | Backend exists, but developer-facing documentation and stable promotion still need to be finished. |
| Published Swagger / OpenAPI endpoint | **Partial** | P0 | Backend already serves docs; public developer guidance still needs to point to it cleanly. |
| Contract verification endpoint and UI | **Missing** | P1 | Still not complete. |
| Token-list endpoint | **Missing** | P1 | Needed by wallet watch-asset and dApp token selectors. |
| Explorer WebSocket fan-out | **Missing** | P2 | Polling exists; streaming read surface still future work. |

---

## 6. Infrastructure and operational readiness inventory

| Area | Status | Priority | Notes |
|---|---|---|---|
| Public TLS RPC ingress | **Live** | P0 | Working on `443`. The public edge was returning `502` on `2026-04-27` because `Node-RPC` itself had stopped; the managed `Node-RPC` runtime was restarted on the host and service was restored. |
| Public TLS WSS ingress | **Live** | P0 | Working on `443`. |
| Raw custom-port public exposure | **Filtered upstream** | P1 | Not needed for browser use today, but still worth documenting correctly. |
| Method exposure-profile enforcement | **Live** | P0 | Public HTTP and WSS now enforce the Security-Spec method tiers; local loopback still retains the node-local administrative surface. |
| Observer/Grafana query accuracy | **Live** | P1 | The active dashboards now query the live `synergy_*` series, the stale metric references only remain in timestamped backup copies, and the observer includes a dedicated public-edge / bootstrap dashboard. |
| Observer scrape coverage | **Live** | P1 | Prometheus now has `0` down targets. Validators, relayers, observer, RPC, and explorer are covered by live metrics scrapes; bootnodes and seeds are covered by observer-side public TCP probes on their real bootstrap ports. |
| Bootnode host-level CPU / memory / disk telemetry | **Partial** | P2 | The observer does not yet have raw `node_exporter` host metrics for bootnodes because their exporter ports are not reachable on the current public path. Service reachability is still covered through blackbox TCP probes. |
| Public testnet faucet | **Missing** | P0 | External developers still need a supported funding path. |
| `synergy-network.io/chains.json` | **Missing** | P1 | Still helpful for ecosystem autodiscovery. |
| Public status page | **Missing** | P1 | Still operational follow-on work. |
| Multi-region RPC balancing | **Missing** | P1 | Not a current blocker for initial testnet use, but still future infra work. |

---

## 7. Documentation inventory

| Deliverable | Status | Priority | Notes |
|---|---|---|---|
| RPC/API reference updated for current ports and current methods | **Partial** | P0 | The doc set still needs a pass so it matches the live public TLS ingress on `443`, the node-local backend ports (`5646` / `5666` on `Node-RPC`), and the current method surface including `synergy_getAccountAuthNonce`, `synergy_reverseResolveSynID`, and `synergy_resetChainHead`. |
| dApp connection manual aligned to live state | **Partial** | P0 | Companion documentation still needs a final accuracy pass after the remaining live fixes land. |
| Error-code lookup table | **Missing** | P0 | Needed once final JSON-RPC subcodes are settled. |
| SynID registration walkthrough | **Missing** | P1 | Still not complete. |
| Address-format cheatsheet | **Missing** | P1 | Still not complete. |
| Post-quantum signer selection guide | **Missing** | P1 | Still not complete. |

---

## 8. Practical rollout view

### Good to go now

- Browser dApps can reach the network through the verified HTTPS/WSS endpoints on `443`.
- Public read flows and WebSocket subscriptions are usable through the current RPC gateway.
- Authority-plane and non-public write RPC methods are no longer exposed on the public HTTPS/WSS ingress.
- The validator fleet, public RPC node, explorer/indexer node, control-panel bundle, and local monitor workspace are aligned to the same runtime checksum `858c5aa89696f4e6fb3483edd024eec3cdad8f097d326470eeeb424a6fd2d37c`.
- The control-panel-managed validator fleet is currently running and reporting `sync_status: synced` after the `858c...` rollout.
- The chain is moving and public RPC is responding again after the `Node-RPC` restart on `2026-04-27`.
- Grafana now has clean observer data for the validator fleet, the shared public service host, and the public bootstrap / seed surfaces.

### Not good to go yet

- Wallet-to-dApp connection UX is not yet complete.
- Approval-management RPC and SynID convenience RPC are still missing.
- The sustained block-time target is not yet back under `5s`; fresh public sampling on `2026-04-27` is still around `10s/block`.
- Official developer packages and onboarding flow are still incomplete.
- Bootnode host-level resource telemetry is still not visible from the observer; only their public service reachability is covered today.

---

*End of tracker.*
