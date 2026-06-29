# SDK Documentation
## Synergy Network

---

## 1. Overview

The Synergy SDK enables developers to build applications on the Synergy Network using JavaScript, Python, and Rust libraries. These SDKs simplify interaction with smart contracts, wallets, transactions, and governance by abstracting JSON-RPC calls and encoding standards.

---

## 2. Available SDKs

| Language | Package Name | Repository |
|---|---|---|
| **JavaScript** | `@synergy/sdk-js` | github.com/synergy-network/sdk-js |
| **Python** | `synergy_sdk` | github.com/synergy-network/sdk-python |
| **Rust** | `synergy-sdk` (crate) | crates.io/crates/synergy-sdk |

---

## 3. Installation

```bash
# JavaScript
npm install @synergy/sdk-js

# Python
pip install synergy_sdk
```

```toml
# Rust (Cargo.toml)
[dependencies]
synergy-sdk = "1.0"
```

---

## 4. Core Features

| Function | Description |
|---|---|
| **Wallet Management** | Create/import PQC wallets, sign transactions |
| **Contract Interaction** | Call smart contracts, submit deploy scripts |
| **Staking Operations** | Delegate/undelegate, check rewards |
| **Governance Participation** | Submit proposals, cast votes |
| **SNS and UMA Integration** | Register `.syn` names, link UMA addresses |

---

## 5. Example Usage

### JavaScript: Sending SYN

```javascript
const { Wallet } = require("@synergy/sdk-js");
const wallet = Wallet.fromMnemonic("... seed words ...");
await wallet.send("sYnQ1xyz...", 100.0);
```

### Python: Querying Wallet Balance

```python
from synergy_sdk.wallet import Wallet
wallet = Wallet.from_mnemonic("... seed ...")
print(wallet.get_balance())
```

### Rust: Calling Smart Contract

```rust
let client = synergy_sdk::Client::new("https://rpc.synergy-network.io");
let tx = client.contract("0xabc...").method("getValue").call()?;
```

---

## 6. Network Configuration

| Network | Endpoint |
|---|---|
| **Mainnet** | https://rpc.synergy-network.io |
| **Testnet** | https://rpc.testnet.synergy-network.io |
| **WebSocket** | wss://ws.testnet.synergy-network.io |

Each SDK allows setting custom endpoints for private chains.

---

## 7. Error Handling & Logging

```javascript
// JavaScript
sdk.setLogLevel("debug")
```

```python
# Python
sdk.enable_logging(level="DEBUG")
```

---

## 8. Security Considerations

- Wallet keys are stored encrypted (never uploaded)
- PQC (Dilithium) signatures are used for all transactions
- SDKs validate inputs before RPC dispatch

---

## 9. Contributions & Support

- **Issues:** https://github.com/synergy-network/sdk-js/issues
- **Discord:** https://discord.synergy-network.io
- **Documentation:** https://docs.synergy-network.io/sdk

---

*© 2025 Synergy Network. Confidential.*
