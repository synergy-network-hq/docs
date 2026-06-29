# Token Migration Guide
## Testnet → Mainnet

---

## 1. Introduction

This document provides a step-by-step guide for migrating Synergy Token (SYN) assets from the Synergy Testnet environment to the Mainnet. This process ensures secure and accurate mapping of user balances, validator stakes, and smart contract states during network upgrade phases.

---

## 2. Migration Phases

### Phase 1: Snapshot Finalization
- Final Testnet block height is announced.
- Snapshot of all token balances, staking states, SNS names, and UMA addresses is taken.
- The snapshot hash is published and verified by validators.

### Phase 2: Smart Contract Deployment on Mainnet
- Migration contract is deployed on Mainnet.
- Verifiable Merkle Root of snapshot data is embedded in the migration contract.
- Migration UI and CLI tools are made available to users.

### Phase 3: User-Initiated Migration
- Users access migration portal or CLI tool.
- Submit their Testnet address and receive Merkle proof.
- Sign a transaction proving ownership of the Testnet address.
- Claim Mainnet SYN tokens to a corresponding Mainnet wallet.

---

## 3. Required Tools

| Tool | Details |
|---|---|
| **Wallet** | MetaMask or Union Wallet (Mainnet enabled) |
| **Migration Portal** | https://migration.synergy-network.io |
| **CLI Tool** | `synergy-cli migrate` |
| **Merkle Proof Generator** | Bundled in CLI release |

---

## 4. Migration Steps

### Step 1: Verify Eligibility
Go to migration portal and enter your Testnet address. Confirm balance and claim status.

### Step 2: Prepare Mainnet Wallet
Import or create a Mainnet-compatible wallet and ensure it's connected to Synergy Mainnet endpoint.

### Step 3: Generate Proof of Ownership

```bash
synergy-cli generate-proof --address <testnet_address>
```

Sign the resulting message with your Testnet private key.

### Step 4: Submit Migration Transaction

```bash
synergy-cli migrate --proof <proof.json> --signature <sig> --to <mainnet_address>
```

### Step 5: Confirm Token Receipt
Verify SYN tokens in your Mainnet wallet and optionally track the transaction on Synergy Explorer.

---

## 5. Developer Notes

- Snapshot JSON and Merkle proof structures follow the Ethereum Merkle Airdrop standard.
- **Contract Address:** `0xSYNG-MIGRATE-MAINNET`
- **Source Code:** Audited and published in `synergy-network/token-migration`

---

## 6. Security Notes

- Never share your private key.
- Always verify the domain and SSL certificate when using the migration portal.
- Use hardware wallets when possible for signing.

---

## 7. Support

- **FAQ:** https://docs.synergy-network.io/migration/faq
- **Discord:** `#migration-help` channel
- **Email:** support@synergy-network.io

---

*© 2025 Synergy Network. Confidential.*
