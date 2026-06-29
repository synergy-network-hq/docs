# Deployment Tools
## CLI + GUI — Synergy Network

---

## 1. Overview

Deploying smart contracts and managing network components on the Synergy Network can be done using a feature-rich CLI and an intuitive GUI interface.

---

## 2. Synergy CLI Overview

```bash
git clone https://github.com/synergy-network/cli.git
cd cli && cargo build --release
./target/release/synergy-cli --help
```

Key Functions: wallet creation/import/recovery, contract deployment and method calls, validator setup and monitoring, governance voting and proposal creation.

---

## 3. CLI: Deploying a Contract

```bash
# Deploy
synergy-cli contract deploy --wasm ./my_contract.wasm --wallet mywallet.syn

# Call method
synergy-cli contract call --contract <address> --function get_hello --args []

# Upload metadata
synergy-cli contract metadata upload --contract <addr> --abi ./abi.json
```

---

## 4. CLI: Validator Registration

```bash
synergy-cli validator register \
  --name "ClusterAlpha" \
  --commission 3 --bond 25000 --website "https://validator.alpha"
```

---

## 5. CLI: Governance Participation

```bash
# Submit proposal
synergy-cli proposal submit --type funding --title "Wallet Grants" --amount 100000 --description "Fund wallet development."

# Vote
synergy-cli proposal vote --id 52 --vote yes
```

---

## 6. Synergy GUI Wallet Tool

Download: https://synergy-network.io/wallet

Features: create/import/manage wallets, deploy and interact with contracts, register SNS names and link UMA, stake SYN and track rewards, vote on proposals with real-time impact previews.

---

## 7. Deploy via GUI

1. Navigate to **Contracts → Deploy Contract**
2. Upload `.wasm` file
3. Name and tag the contract
4. Confirm deployment and pay gas in SYN
5. Interact with methods in the live UI after deployment

---

## 8. Developer Convenience Tools

| Tool | Purpose |
|---|---|
| `deploy.js` (Hardhat) | Automated JS deployment scripts |
| `scripts/cli-tools` | Bash helpers for testnet operators |
| Snapshot Restore | Resync testnet node after restart |
| Gas Estimator | CLI and explorer estimation tools |

---

## 9. Deployment Best Practices

- Always simulate with `synergy-cli contract simulate` before deploying
- Upload ABI to explorer for discoverability
- Use SNS to register contract name (e.g., `voting.syn`)
- Monitor transaction on Explorer after deploy

---

*© 2025 Synergy Network. Confidential.*
