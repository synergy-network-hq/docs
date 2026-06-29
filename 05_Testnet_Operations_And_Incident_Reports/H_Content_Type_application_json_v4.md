# Block Explorer Guide
## Synergy Network

---

## Overview

The Synergy Network Block Explorer provides comprehensive tools for viewing blockchain data, tracking transactions, monitoring validators, and analyzing network performance.

---

## Accessing the Explorer

**Web Interface:**
```
http://localhost:3000
```

**API Access:**
```bash
curl -X POST http://localhost:8545 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc": "2.0", "method": "synergy_getLatestBlock", "params": [], "id": 1}'
```

---

## Core Features

### Block Information

```json
// Latest block
{"jsonrpc":"2.0","method":"synergy_getLatestBlock","params":[],"id":1}

// Block by number
{"jsonrpc":"2.0","method":"synergy_getBlockByNumber","params":[12345],"id":1}

// Block range
{"jsonrpc":"2.0","method":"synergy_getBlockRange","params":[10000,10100],"id":1}
```

### Transaction Data

```json
// By hash
{"jsonrpc":"2.0","method":"synergy_getTransactionByHash","params":["0x..."],"id":1}

// Transactions in block
{"jsonrpc":"2.0","method":"synergy_getTransactionsInBlock","params":[12345],"id":1}
```

### Address Information

```json
// Balance
{"jsonrpc":"2.0","method":"synergy_getAllBalances","params":["sYn..."],"id":1}

// Transfer history
{"jsonrpc":"2.0","method":"synergy_getTransferHistory","params":["sYn...",100],"id":1}
```

### Validator Information

```json
// All validators
{"jsonrpc":"2.0","method":"synergy_getValidators","params":[],"id":1}

// Top validators
{"jsonrpc":"2.0","method":"synergy_getTopValidators","params":[10],"id":1}
```

### Network Statistics

```json
{"jsonrpc":"2.0","method":"synergy_getNetworkStats","params":[],"id":1}
```

---

## API Rate Limits

| Tier | Per Minute | Per Hour | Per Day |
|---|---|---|---|
| **Free** | 100 | 1,000 | 10,000 |
| **Pro** | 1,000 | 100,000 | Unlimited |

---

## Real-time Monitoring

Subscribe to real-time updates via WebSocket:

```javascript
const ws = new WebSocket('ws://localhost:8546');
ws.onmessage = function(event) {
    const data = JSON.parse(event.data);
    console.log('New block:', data);
};
```

Available subscriptions: New Blocks, New Transactions, Validator Updates, Token Transfers.

---

## Troubleshooting

- **"Block not found":** Verify block number exists and check network synchronization.
- **"Transaction not found":** Verify transaction hash or check the transaction pool.
- **"Rate limit exceeded":** Reduce request frequency, implement caching, or upgrade to Pro tier.

---

## Contact & Support

- **Email:** explorer@synergy-network.io
- **Support Hours:** 24/7
- **Response Time:** < 24 hours

---

*© 2025 Synergy Network. Confidential.*
