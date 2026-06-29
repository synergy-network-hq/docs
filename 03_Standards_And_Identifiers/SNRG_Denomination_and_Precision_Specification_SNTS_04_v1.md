# SNRG Denomination & Precision Specification (SNTS-04)

**Version:** v1.2  
**Classification:** Public Technical Standard  
**Date:** December 2025  

## Companion Documents

- SNTS-01: Synergy Address Formatting Specification
- SNTS-03: AIVM Execution Determinism Standard
- SNTS-05: Synergy Cross-Chain Protocol (SXCP) Specification

---

## 1. Purpose and Scope

This specification defines the official denomination hierarchy, precision model, and cross-chain decimal conversion rules for the Synergy Coin (SNRG), the native asset of the Synergy Network.

This standard governs:

- Synergy Network L1
- AIVM smart contracts
- All wallets, explorers, and SDKs
- All SXCP multi-chain integrations
- Off-chain systems such as exchanges and price oracles

This version adopts the Wei-class naming system and establishes nano-wei (nWei) as the smallest indivisible monetary unit.

---

## 2. Monetary Parameters

| Parameter | Value |
|-----------|-------|
| **Native Asset** | SNRG (Synergy Coin) |
| **Base Denomination** | SNRG |
| **Precision** | 9 decimal places |
| **Smallest Unit** | nWei (nano-wei) |
| **Max Total Supply** | 12,000,000,000 SNRG (12 billion SNRG) |
| **Max Total Supply (nWei)** | 12,000,000,000,000,000,000 nWei (12 × 10¹⁸ nWei) |

**All protocol-level amounts must be stored as integer nWei values.**  
**Floating-point arithmetic is strictly prohibited in consensus systems.**

---

## 3. Denomination Hierarchy

Synergy Network uses a clean and deterministic Wei-class hierarchy:

| Unit Name | Symbol | Factor | nWei Equivalent | Formal Decimal |
|-----------|--------|--------|-----------------|----------------|
| **Synergy Coin** | SNRG | 1 | 1,000,000,000 | 10⁹ |
| **milli-wei** | mWei | 0.001 | 1,000,000 | 10⁶ |
| **micro-wei** | µWei | 0.000001 | 1,000 | 10³ |
| **nano-wei** | nWei | 0.000000001 | 1 | 10⁰ |

### Pronunciation

"Wei" is pronounced "way" (named after Wei Dai).

- mWei: milli-way
- µWei: micro-way
- nWei: nano-way

---

## 4. Accounting Rules

### 4.1 Ledger Storage

All balances, transfers, state values, and AIVM contract fields must store monetary amounts as **raw nWei**.

**Example:**

```
35.125 SNRG → 35.125 × 1e9 = 35,125,000,000 nWei
```

### 4.2 Smart Contract Arithmetic (AIVM)

- Integer nWei arithmetic only
- Deterministic division with explicit rounding
- No floating-point operations
- All user-facing values must convert from integer nWei

### 4.3 Wallet & Explorer Display

- **Default:** SNRG
- **Advanced Modes:** mWei, µWei, nWei
- **Gas fees:** Always shown in nWei (with optional SNRG equivalent)

---

## 5. Gas Model

- **Gas price unit:** nWei
- **Gas limit:** integer

### Fee formula:

```
fee_nWei = gasLimit × gasPrice_nWei
fee_SNRG = fee_nWei ÷ 1e9
```

### Example:

- **Gas Limit:** 25,000
- **Gas Price:** 40 nWei
- **Fee:** 25,000 × 40 = 1,000,000 nWei = 0.001 SNRG

---

## 6. Cross-Chain Precision Mapping

**Synergy Network is bridgeless.**  
**SXCP does not wrap, mint, or burn assets.**

However, decimal conversion is still required for:

- Cross-chain UX
- Wallets and explorers
- Multi-chain pricing displays
- CEX integrations
- Oracles
- Off-chain accounting systems

This section defines the official SNRG decimal mappings to the three dominant decimal standards:

- 18-decimal chains (Ethereum/EVM ecosystems)
- 9-decimal chains (Solana/SVM ecosystems)
- 8-decimal chains (Bitcoin family)

### 6.1 General Conversion Framework

Let:

- D_source = decimals of source chain
- D_target = decimals of target chain
- amount_source = integer amount in source base units
- amount_target = integer amount in target base units

**General formula:**

```
amount_target = amount_source × 10^(D_target – D_source)
```

If the exponent is negative, flooring must be used for determinism.

---

## 7. Conversion Rules by Chain Type

### 7.1 SNRG (9 decimals) → EVM Chains (18 decimals)

**Decimal difference:**

```
18 − 9 = 9
```

#### 7.1.1 SNRG → EVM (9 → 18)

```
evm_amount = snrg_amount × 10⁹
```

**Example:**

```
2.5 SNRG = 2.5 × 1e9 = 2,500,000,000 nWei
→ 2,500,000,000 × 1e9 = 2,500,000,000,000,000,000 EVM units
```

#### 7.1.2 EVM → SNRG (18 → 9)

```
snrg_amount = evm_amount ÷ 10⁹
```

**Remainders are discarded (floored).**

### 7.2 SNRG (9 decimals) → 9-Decimal Chains (e.g., Solana)

**Decimal difference:**

```
9 − 9 = 0
```

#### 7.2.1 Conversion Rule

```
target_amount = source_amount
```

No decimal scaling required.

**This equivalence applies only to value representation; SXCP does not wrap SNRG into Solana assets.**

### 7.3 SNRG (9 decimals) → Bitcoin Family (8 decimals)

**Decimal difference:**

```
8 − 9 = –1
```

#### 7.3.1 SNRG → Bitcoin (9 → 8)

```
btc_amount = snrg_amount ÷ 10
```

#### 7.3.2 Bitcoin → SNRG (8 → 9)

```
snrg_amount = btc_amount × 10
```

---

## 8. SDK Constants

### Rust

```rust
pub const SNRG_DECIMALS: u32 = 9;

pub const NWEI_PER_SNRG: u128 = 1_000_000_000;
pub const MWEI_PER_SNRG: u128 = 1_000_000;
pub const UWEI_PER_SNRG: u128 = 1_000;

pub const EVM_DECIMALS: u32 = 18;
pub const BTC_DECIMALS: u32 = 8;
```

### JavaScript

```javascript
const SNRG_DECIMALS = 9n;
const NWEI_PER_SNRG = 1_000_000_000n;

const toEvm = amount => amount * 10n ** 9n;
const fromEvm = amount => amount / 10n ** 9n;

const toBitcoin = amount => amount / 10n;
const fromBitcoin = amount => amount * 10n;
```

---

## 9. Compliance Requirements

To be Synergy-compliant, all implementations must:

- Store monetary values as integer nWei.
- Use nWei for all gas and fee calculations.
- Apply deterministic flooring when reducing precision.
- Display SNRG by default in wallets and explorers.
- Support mWei, µWei, and nWei in advanced UI modes.
- Use the conversion rules in Section 7 for all external decimal mappings.
- Use no floating-point arithmetic inside AIVM or node logic.

---

## 10. Revision History

| Version | Date | Changes |
|---------|------|---------|
| v1.0 | October 2025 | Initial specification |
| v1.1 | November 2025 | Added cross-chain mapping section |
| v1.2 | December 2025 | Refined SDK constants and compliance rules |

---

**END OF SPECIFICATION**
