# API Reference
## JSON-RPC + WebSocket — Synergy Network

---

## 1. Overview

The Synergy Network exposes a comprehensive JSON-RPC and WebSocket API interface for developers to interact with the blockchain programmatically.

---

## 2. Endpoints

| Type | URL |
|---|---|
| **JSON-RPC** | https://rpc.synergy-network.io |
| **Testnet RPC** | https://rpc.testnet.synergy-network.io |
| **WebSocket** | wss://ws.synergy-network.io |
| **Testnet WS** | wss://ws.testnet.synergy-network.io |

---

## 3. JSON-RPC Request Format

```json
{
  "jsonrpc": "2.0",
  "method": "<method_name>",
  "params": [...],
  "id": 1
}
```

---

## 4. Common Methods

| Method | Description |
|---|---|
| `eth_blockNumber` | Returns the latest block number |
| `eth_getBlockByNumber` | Gets block details by number |
| `eth_getTransactionByHash` | Retrieves transaction by hash |
| `eth_getBalance` | Returns wallet balance |
| `eth_sendRawTransaction` | Broadcasts a signed transaction |
| `eth_call` | Executes a read-only contract method |
| `eth_estimateGas` | Estimates gas usage |

---

## 5. Example RPC Request

```bash
curl -X POST https://rpc.testnet.synergy-network.io \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

---

## 6. WebSocket Usage

```json
{
  "id": 1,
  "method": "eth_subscribe",
  "params": ["newHeads"]
}
```

Common subscriptions: `newHeads` (block headers), `logs` (contract events), `syncing` (sync progress).

---

## 7. Filtering Events

```json
{
  "method": "eth_subscribe",
  "params": ["logs", {
    "address": "0xContractAddress",
    "topics": ["0xTopicHash"]
  }]
}
```

---

## 8. Error Response

```json
{
  "jsonrpc": "2.0",
  "error": {"code": -32000, "message": "insufficient funds"},
  "id": 1
}
```

---

## 9. Developer Notes

- All methods are Ethereum-compatible (EVM subset)
- Batch requests supported
- PQC transaction formats handled at signature layer
- WebSocket connections limited to 100 subscriptions per session

---

*© 2025 Synergy Network. Confidential.*
