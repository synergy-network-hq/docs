# Token System
## Synergy Network Technical Reference

---

## Overview

The Synergy Network features a comprehensive token system that supports multiple token types, staking, and advanced token operations. The system is designed to be flexible and extensible while maintaining security and performance.

---

## Token Structure

### Token Properties

| Property | Description |
|---|---|
| **Symbol** | Unique identifier (e.g., "SNRG", "USDS") |
| **Name** | Human-readable name |
| **Decimals** | Number of decimal places (9 is Synergy Network default) |
| **Total Supply** | Current circulating supply |
| **Max Supply** | Maximum possible supply (optional) |
| **Mintable** | Whether new tokens can be created |
| **Burnable** | Whether tokens can be destroyed |
| **Creator** | Address that created the token |

### Native Token: SNRG

| Property | Value |
|---|---|
| **Symbol** | SNRG |
| **Name** | SynergyCoin |
| **Decimals** | 9 |
| **Max Supply** | 12,000,000,000 SNRG |
| **Mintable** | No |
| **Burnable** | Yes |

---

## Token Operations

### Create Token
```json
{"jsonrpc":"2.0","method":"synergy_createToken","params":["MYTOKEN","My Token",18,1000000,"sYn..."],"id":1}
```

### Mint Tokens
```json
{"jsonrpc":"2.0","method":"synergy_mintTokens","params":["sYn...","SNRG",1000000],"id":1}
```

### Burn Tokens
```json
{"jsonrpc":"2.0","method":"synergy_burnTokens","params":["sYn...","SNRG",1000000],"id":1}
```

### Transfer Tokens
```json
{"jsonrpc":"2.0","method":"synergy_transferTokens","params":["sYn...","sYn...","SNRG",1000],"id":1}
```

---

## Balance Management

```json
// Single token
{"jsonrpc":"2.0","method":"synergy_getTokenBalance","params":["sYn...","SNRG"],"id":1}

// All tokens
{"jsonrpc":"2.0","method":"synergy_getAllBalances","params":["sYn..."],"id":1}
```

---

## Staking System

```json
// Stake
{"jsonrpc":"2.0","method":"synergy_stakeTokensDirect","params":["sYn...","sYn...","SNRG",1000000],"id":1}

// Unstake
{"jsonrpc":"2.0","method":"synergy_unstakeTokens","params":["sYn...","sYn...","SNRG",500000],"id":1}

// Staking info
{"jsonrpc":"2.0","method":"synergy_getStakingInfo","params":["sYn..."],"id":1}
```

---

## Security Features

- **Balance Validation:** All operations include validation to prevent insufficient funds and double-spending.
- **Access Control:** Only authorized addresses can mint; burn and transfer validate sender balance.
- **State Management:** Persistent storage, atomic operations, and rollback capabilities.

---

## Troubleshooting

| Error | Resolution |
|---|---|
| "Insufficient balance" | Ensure you have enough tokens for the operation |
| "Token not found" | Verify the token symbol is correct |
| "Unauthorized" | Check if you have permission for the operation |
| "Network error" | Verify node connectivity |

---

*© 2025 Synergy Network. Confidential.*
