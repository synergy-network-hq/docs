# Wallet Usage Guide
## Synergy Network API Reference

*Last Modified: September 2025*

---

## Overview

Wallets in the Synergy Network are essential for managing digital assets, signing transactions, and interacting with the blockchain.

---

## Wallet Types

| Type | Description |
|---|---|
| **Software Wallets** | Generated and stored locally; full control over private keys; requires secure backup |
| **Hardware Wallets** | Physical devices for key storage; enhanced security; recommended for large amounts |
| **Watch-Only Wallets** | Monitor balances without spending ability; useful for exchanges and services |

---

## Creating a Wallet

### Method 1: Generate New Wallet

```json
{"jsonrpc":"2.0","method":"synergy_createWallet","params":[],"id":1}
```

### Method 2: Import from Keypair

```json
{"jsonrpc":"2.0","method":"synergy_createWalletFromKeypair","params":["public_key_hex","private_key_hex"],"id":1}
```

---

## Address Format

- Bech32m addresses with "sYn" prefix
- Format: `sYn` + 38 character hash
- Example: `sYn1q2w3e4r5t6y7u8i9o0p1a2s3d4f5g6h7j8k`

---

## Wallet Operations

```json
// Get wallet info
{"jsonrpc":"2.0","method":"synergy_getWallet","params":["sYn..."],"id":1}

// List all wallets
{"jsonrpc":"2.0","method":"synergy_getAllWallets","params":[],"id":1}
```

---

## Transaction Signing

### Manual Signing

```json
{
  "jsonrpc": "2.0",
  "method": "synergy_signTransaction",
  "params": ["sYn...", {
    "sender": "sYn...", "receiver": "sYn...",
    "amount": 1000, "nonce": 5,
    "gas_price": 1000, "gas_limit": 21000
  }],
  "id": 1
}
```

### Automated Token Sending

```json
{"jsonrpc":"2.0","method":"synergy_sendTokens","params":["sYn...","sYn...","SNRG",1000],"id":1}
```

---

## Integration Examples

### JavaScript

```javascript
const axios = require('axios');

async function createWallet() {
  const response = await axios.post('http://localhost:8545', {
    jsonrpc: '2.0', method: 'synergy_createWallet', params: [], id: 1
  });
  return response.data.result;
}
```

### Python

```python
import requests

def create_wallet():
    payload = {"jsonrpc":"2.0","method":"synergy_createWallet","params":[],"id":1}
    return requests.post('http://localhost:8545', json=payload).json()['result']
```

---

## Troubleshooting

| Error | Resolution |
|---|---|
| "Wallet not found" | Verify address format and network connectivity |
| "Insufficient funds" | Check balances and consider gas fees |
| "Signature failed" | Verify private key and transaction format |
| "Network error" | Check node connectivity and RPC server status |

---

## Security Best Practices

- Never share private keys
- Use encrypted storage or hardware wallets
- Always verify recipient addresses and amounts before signing
- Wait for transaction confirmation on high-value transfers

---

*© 2025 Synergy Network. Confidential.*
