# API Reference
## Full Synergy JSON-RPC Specification

---

## Overview

The Synergy Network provides a comprehensive JSON-RPC API accessible via HTTP POST to port 8545.

**Base URL:** `http://localhost:8545`

---

## Request & Response Format

```json
// Request
{"jsonrpc":"2.0","method":"synergy_methodName","params":[...],"id":1}

// Success
{"jsonrpc":"2.0","result":{...},"id":1}

// Error
{"jsonrpc":"2.0","error":{"code":-32600,"message":"Invalid Request"},"id":1}
```

---

## Blockchain Queries

| Method | Description |
|---|---|
| `synergy_blockNumber` | Returns the latest block number |
| `synergy_getBlockByNumber` | Returns block info by block number |
| `synergy_getLatestBlock` | Returns the latest block information |
| `synergy_getBlockRange` | Returns blocks in a specified range |
| `synergy_getTransactionsInBlock` | Returns all transactions in a block |

---

## Transaction Methods

| Method | Description |
|---|---|
| `synergy_sendTransaction` | Submits a transaction to the network |
| `synergy_getTransactionPool` | Returns all pending transactions |
| `synergy_getTransactionByHash` | Returns transaction info by hash |

---

## Token Operations

| Method | Description |
|---|---|
| `synergy_createToken` | Creates a new token on the network |
| `synergy_mintTokens` | Mints new tokens to an address |
| `synergy_burnTokens` | Burns tokens from an address |
| `synergy_transferTokens` | Transfers tokens between addresses |
| `synergy_getTokenBalance` | Returns token balance for an address |
| `synergy_getAllBalances` | Returns all token balances for an address |
| `synergy_getTokens` | Returns all available tokens |
| `synergy_getTransferHistory` | Returns transfer history for an address |
| `synergy_getTokenStats` | Returns comprehensive token statistics |

---

## Staking Operations

| Method | Description |
|---|---|
| `synergy_stakeTokens` | Creates a staking transaction via wallet manager |
| `synergy_stakeTokensDirect` | Stakes tokens directly to a validator |
| `synergy_unstakeTokens` | Unstakes tokens from a validator |
| `synergy_getStakedBalance` | Returns staked balance for an address |
| `synergy_getStakingInfo` | Returns staking information for an address |

---

## Wallet Management

| Method | Description |
|---|---|
| `synergy_createWallet` | Creates a new wallet |
| `synergy_createWalletFromKeypair` | Creates a wallet from an existing keypair |
| `synergy_getWallet` | Returns wallet information |
| `synergy_getAllWallets` | Returns all wallets |
| `synergy_signTransaction` | Signs a transaction |

---

## Validator Management

| Method | Description |
|---|---|
| `synergy_registerValidator` | Registers a new validator |
| `synergy_approveValidator` | Approves a pending validator registration |
| `synergy_getValidators` | Returns all validators |
| `synergy_getValidator` | Returns validator information by address |
| `synergy_getTopValidators` | Returns top validators by Synergy Score |
| `synergy_getValidatorActivity` | Returns validator activity statistics |
| `synergy_getSynergyScoreBreakdown` | Returns detailed Synergy Score components |
| `synergy_slashValidator` | Slashes a validator |

---

## Network Information

| Method | Description |
|---|---|
| `synergy_nodeInfo` | Returns node information |
| `synergy_getNetworkStats` | Returns comprehensive network statistics |
| `synergy_getValidatorStats` | Returns comprehensive validator statistics |

---

## Error Codes

| Code | Meaning |
|---|---|
| `-32700` | Parse error |
| `-32600` | Invalid Request |
| `-32601` | Method not found |
| `-32602` | Invalid params |
| `-32603` | Internal error |

---

## Usage Examples

### Create a Token

```bash
curl -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"synergy_createToken","params":["MYTOKEN","My Token",18,1000000,"sYn..."],"id":1}'
```

### Transfer Tokens

```bash
curl -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"synergy_transferTokens","params":["sYn...","sYn...","SNRG",1000],"id":1}'
```

### Get Network Stats

```bash
curl -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"synergy_getNetworkStats","params":[],"id":1}'
```

---

*© 2025 Synergy Network. Confidential.*
