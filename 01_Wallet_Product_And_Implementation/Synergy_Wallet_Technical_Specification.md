# Synergy Wallet Technical Specification
## Version 3.0 · February 2026

**CONFIDENTIAL — PATENT-SENSITIVE MATERIAL**
Not for public distribution prior to formal IP filing.

---

## Canonical Design Principles

### A — SXCP North Star (Bridgeless)

- SXCP never executes transactions; it only coordinates execution that happens elsewhere.
- Source-chain execution occurs natively; destination-chain execution occurs natively.
- SXCP is bridgeless: no intermediate custody domain (no pooled vaults, no mint/release wrappers, no synthetic representations).
- Threshold Attestation Mode attests to observed chain facts (finalization, inclusion, scope). It is not delegated execution.

### B — Seed-Theft Resistance (Non-Negotiable)

- Possession of a seed phrase (mnemonic) alone MUST be insufficient to spend or to complete wallet recovery.
- The mnemonic is treated as one factor in a multi-factor authority construction (Factor-Bound Root Authority, §7.2.3).
- The second factor (F2) must be high entropy, non-memorable, and not stored with the mnemonic.
- All execution keys are derived from `WRA = HKDF(MS ‖ F2)`, not from MS alone.

### C — PQC Alignment (Aegis Authority Boundary)

- Aegis governs all authoritative protocol roles: identity, routing governance, attestations, recovery receipts, and audit artifacts.
- The platform baseline for authoritative signing artifacts is ML-DSA (FIPS 204). FN-DSA (FIPS 206) is permitted only where explicitly specified by role or policy.
- External-chain execution remains canonical to each chain until that chain supports PQ verification.
- No subsystem may silently substitute non-Aegis signature primitives for Aegis-governed roles.

---

# 0. Preface and Document Scope

## 0.1 Purpose and Audience

This document is the canonical internal technical specification for the Synergy Wallet system. It supersedes all prior drafts, including V1 and V2 addenda. It serves simultaneously as an engineering reference, an architectural description for security and protocol analysis, and a disclosure substrate for provisional and non-provisional patent filings.

## 0.3 Terminology and Conventions

- **"Identity"**: Synergy-layer authority, not external-chain accounts.
- **"Execution"**: Exclusively native blockchain transaction signing.
- **"Routing"**: Address resolution and intent direction, not asset movement.
- **"Wallet"**: The composite system, not a single keypair.
- **"WRA"**: The Factor-Bound Root Authority secret from which all operational keys are derived.
- **"F2"**: The high-entropy second factor required alongside the mnemonic-derived master seed.
- **"Aegis"**: The Synergy post-quantum cryptographic governance subsystem.

---

# 1. Abstract

Synergy Wallet is a self-custodial, identity-centric, multi-chain wallet system designed to eliminate irreversible asset loss, collapse fragmented blockchain user experiences, and enable protocol-level cross-chain execution without custodial bridges, delegated signing, or wrapped assets.

A defining security property: possession of the BIP-39 mnemonic phrase alone is computationally insufficient to spend funds or complete wallet recovery. Execution authority requires both the master seed and an independent high-entropy second factor, bound together through the Factor-Bound Root Authority (WRA) construction.

---

# 2. Problem Statement and Design Motivation

## 2.1 Structural Failures of Existing Wallet Architectures

- **Single-Point Catastrophic Failure:** Loss or compromise of a seed phrase results in permanent loss of identity, authority, and assets.
- **Identity–Execution Conflation:** A single private key simultaneously represents user identity, governance authority, and transaction execution power.
- **Chain Fragmentation:** Users must manage incompatible address formats, signing curves, and derivation standards across every supported chain.
- **Unsafe Interoperability:** Cross-chain transfers rely on custodial bridges, wrapped assets, and opaque trust assumptions.
- **Cryptographic Obsolescence:** Most wallets cannot evolve toward post-quantum security without breaking compatibility.

---

# 3. Architectural Overview

## 3.1 The Three-Plane Model

| Plane | Responsibility | What It Cannot Do |
|---|---|---|
| **Identity** | Define authority and ownership | Execute transactions or sign chain-native TXs |
| **Routing** | Resolve intent and destination (UMA) | Authorize execution or hold private keys |
| **Execution** | Move value on external chains natively | Govern identity, modify routing, or delegate authority |

## 3.2 Explicit Architectural Invariants

1. No single key can define identity and execute value transfer.
2. Identity authority cannot sign external-chain transactions.
3. Routing data cannot initiate or authorize execution.
4. Execution keys cannot modify identity or routing state.
5. Loss of any single cryptographic secret cannot permanently destroy future asset control.
6. Possession of a mnemonic phrase alone cannot authorize spending (WRA requirement).

## 3.4 Comparison to Conventional Wallet Models

| Dimension | Conventional Wallet | Synergy Wallet |
|---|---|---|
| Identity | = Private Key | Independent plane (SIK); cannot execute |
| Authority | = Private Key | Governed by Identity Plane; separated from execution |
| Execution | = Private Key | Derived from WRA; disposable instruments |
| Recovery | Restore the key | Restore authority via identity anchoring; no key resurrection |
| Cross-Chain | Per-chain silos / bridges | Protocol coordination via SXCP (bridgeless) |
| Custody | Self (fragile) or bridge | Explicitly none; all execution is chain-native |
| PQC | Future replacement | Parallel authority today via Aegis/ML-DSA |
| Seed Exposure | = Total loss | Insufficient alone; F2 required (WRA) |

---

# 4. Identity Plane — Authority Without Execution

## 4.3 Identity State Model

| State | Description |
|---|---|
| Active | Normal operation; full authority in effect |
| Restricted | Limited operations due to policy violation or recovery guard |
| Recovery-Pending | Recovery initiated; time-lock in effect; UMA updates may be frozen |
| Recovery-Confirmed | Quorum or time-lock satisfied; awaiting generation increment |
| Recovered | New execution generation activated; identity returns to Active |
| Decommissioned | Identity permanently retired; no further state transitions possible |

**Identity Plane Invariants:** Identity ≠ Execution · Authority ≠ Signing Power · Recovery ≠ Key Reconstruction · Names ≠ Addresses · Governance ≠ Custody

---

# 5. Routing Plane — Intent Without Authority

## 5.2 Universal Meta-Address (UMA)

A UMA is a cryptographically governed routing construct that maps a Synergy Identity to one or more chain-native public addresses: `(Synergy Identity, ChainID, Version) → Public Address`. UMA represents where value should be sent, not who can move it. UMA records are explicitly versioned, forward-only, and non-authoritative for execution.

---

# 6. Execution Plane — Authority Without Identity

## 6.3 Deterministic Chain-Native Key Derivation

| Chain Family | Curve / Scheme | Standard |
|---|---|---|
| Bitcoin & UTXO chains | secp256k1 | BIP-32 / BIP-44 |
| Ethereum & EVM chains | secp256k1 | BIP-44 |
| Solana | ed25519 | SLIP-0010 |
| Substrate-based | sr25519 | Native substrate |
| PQC domains | ML-DSA / FN-DSA | Aegis (FIPS 204 / FIPS 206) |

## 6.4 Execution Authorization Pipeline

1. User Intent Expression
2. Routing Resolution via UMA
3. Policy Evaluation (hard gate — abort if failed)
4. Execution Key Selection from WRA-derived domain
5. Native Transaction Construction
6. Local Signature Generation
7. Direct Broadcast to chain RPC

---

# 7. Root Secret and Cryptographic Isolation

## 7.2.3 Factor-Bound Root Authority (WRA)

```
WRA = HKDF( MS ‖ F2, info="synergy.wallet.wra.v1" )
```

MS alone is insufficient to derive any operational signing key.

**Deployment Tiers:**

| Tier | Name | Description |
|---|---|---|
| 1 | Compatibility | Standard single-key EOA derivation. Legacy only. |
| 2 | Factor-Bound Derivation | WRA = HKDF(MS ‖ F2). Seed alone cannot spend. Default tier. |
| 3 | Threshold Execution | No single derived key can unilaterally spend. |
| 4 | Chain-Enforced Accounts | On-chain multi-factor authorization. Strongest tier. |

**Root Secret Invariants:** Determinism without key reuse · Isolation without fragmentation · Recoverability without resurrection · Migration without re-custody · Mnemonic alone is insufficient for execution authority

---

# 9. SXCP — Synergy Cross-Chain Protocol

## 9.7 Non-Custodial Guarantees

| Property | SCETP | SXCP |
|---|---|---|
| Asset custody | Never | Never |
| Local signing | Always (WRA-derived keys) | Always (native to each chain) |
| Wrapped assets | Never | Never |
| Intermediate custody domain | N/A | Never (bridgeless) |
| Delegated execution | Never | Never |

---

# 10. Recovery Without Seed Reconstruction

## 10.3 Separation of Recoverable and Irrecoverable State

**Irrecoverable (by design):** Execution private keys, assets controlled by those keys, historical signing authority. This preserves blockchain immutability.

**Recoverable (by architecture):** Synergy identity authority (SIK), SNS name ownership, UMA routing mappings, policy configuration, future execution derivation (new WRA epoch).

---

# 12. Post-Quantum Architecture (Aegis)

## 12.2 Aegis Algorithm Suite and Wallet Roles

| Algorithm | Standard | Status | Role in Wallet |
|---|---|---|---|
| ML-DSA | FIPS 204 | Active (baseline) | Identity authority, SNS governance, UMA routing governance, SXCP attestations, audit artifacts |
| ML-KEM | FIPS 203 | Active | Encrypted wallet state, multi-device sync, guardian channels, recovery coordination |
| FN-DSA | FIPS 206 | Active (conditional) | High-frequency attestations; NOT for high-stakes authoritative actions |
| HQC-KEM | NIST selected | Active (backup KEM) | Backup KEM; code-based, independent of lattice assumptions |
| SLH-DSA | FIPS 205 | Reserve | Not active: 8–50 KB signature sizes |
| CMCE-KEM | NIST alternate | Reserve | Not active: 261 KB–1 MB public keys |

---

# 13. Threat Model and Security Analysis

## 13.5 Failure Containment Model

| Failure | Blast Radius |
|---|---|
| Mnemonic exposure | None (WRA requires F2; Tier 2+ unaffected) |
| Execution key loss | Single chain, single generation; identity/routing intact |
| F2 loss | Recovery path via guardian-threshold F2 re-establishment |
| Routing error | Execution aborts safely before signing |
| Identity compromise | Governance-gated recovery; execution keys separately derived |
| Relayer failure | Protocol timeout or reversion; compensation pool available |

---

# 14. Patent-Relevant Technical Effects

1. **Seed-Theft-Resistant Execution Authority (WRA Construction):** WRA = HKDF(MS ‖ F2) — two-factor derivation requirement enforced at the cryptographic root prior to any domain expansion.
2. **Seed Phrase Regeneration Without Historical Key Resurrection:** Identity-anchored recovery without reconstructing historical execution keys.
3. **Persistent Identity Independent of Execution Keys:** Survives loss of execution keys, device replacement, and cryptographic algorithm rotation.
4. **Three-Plane Separation with Cryptographic Non-Overlap:** No key material from one plane can derive keys for, authorize actions in, or modify authority in another.
5. **Revocable, Identity-Governed Routing Fabric (UMA):** Versioned routing construct supporting rotation without asset migration; non-authoritative for execution.
6. **Bridgeless Cross-Chain Coordination (SXCP):** Cross-chain coordination without any intermediate custody domain, wrapped asset, pooled vault, or delegated signing.
7. **PQC Parallel Authority Without Execution Disruption:** ML-DSA authority coexisting alongside classical execution without chain upgrades.

---

# 15. Canonical Invariants

If an implementation violates any of these, it is not a Synergy Wallet:

- Mnemonic alone does not authorize spending or recovery
- Identity authority never executes value
- Execution authority never governs identity
- Routing determines intent, not permission
- Recovery restores future control, not history
- Coordination never becomes custody
- Cryptographic migration never breaks compatibility
- ML-DSA is the authoritative signing baseline for protocol-governed roles

---

*© 2025–2026 Synergy Network. CONFIDENTIAL — Patent-Sensitive.*
