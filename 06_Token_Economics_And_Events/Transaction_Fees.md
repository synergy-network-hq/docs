# Transaction Fees
## Synergy Network

---

## 1. Overview

Transaction fees in the Synergy Network serve as an economic mechanism to prevent spam, prioritize processing, and incentivize validators. The fee system is dynamic, balancing affordability and sustainability through congestion-aware pricing and partial token burns.

---

## 2. Fee Structure Overview

- **Base Fee:** Minimum required for any transaction (0.0001 SYN)
- **Dynamic Multiplier:** Adjusts based on network congestion
- **Burn Percentage:** % of base fee permanently removed from supply
- **Validator Tip:** Optional incentive paid directly to block proposers

---

## 3. Transaction Types & Fees

| Transaction Type | Estimated Fee (SYN) | Notes |
|---|---|---|
| Token Transfer | 0.0002–0.001 | Most common type |
| Smart Contract Execution | 0.001–0.01 | Scales with complexity |
| SNS Registration | 0.5–5.0 | Based on name length |
| UMA Mapping | 0.1–0.3 | Once per external address |
| Governance Proposal | 1.0–10.0 | Refundable if proposal passes |

---

## 4. Fee Calculation Logic

- Modeled after EIP-1559 with adaptive block-level base fee
- 50% of base fee is burned
- Tip is set by sender to prioritize transaction in mempool

Example:
```
Base Fee: 0.0004 SYN
Tip:      0.0002 SYN
Total:    0.0006 SYN (0.0002 burned, 0.0002 to validator, 0.0002 to treasury)
```

---

## 5. Setting Fees

**CLI:**
```bash
synergy-cli transaction send --to <addr> --amount 50 --gas-tip 0.0003
```

**GUI:** Click gear icon on send screen → Choose Standard, High Priority, or Custom → Preview estimated fee.

---

## 6. Gas Estimation Tools

```bash
synergy-cli gas estimate --type contract --complexity high
```

- Web interface: https://gas.synergy-network.io
- Explorer live feed shows recent block fee ranges

---

## 7. Fee Redistribution

| Portion | Destination |
|---|---|
| **50% Burned** | Reduces total SYN supply |
| **25% Validators** | Rewarded for inclusion work |
| **25% Treasury** | Used for DAO development |

---

## 8. Monitoring Fee Usage

```bash
synergy-cli wallet fees-stats
```

- Per-wallet spending analytics in GUI wallet
- Global fee burn tracker in explorer dashboard

---

## 9. Future Fee Model Enhancements

- Predictive fee UI using historical gas data
- Synergy Score-based discounts for contributors
- Fee batching: Bundle multiple low-value transactions into one

---

## 10. Conclusion

Synergy Network's transaction fee model promotes long-term token utility and economic stability through dynamic pricing, partial burning, and transparent redistribution.

---

*© 2025 Synergy Network. Confidential.*
