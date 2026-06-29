# Staking Guide
## Synergy Network

---

## 1. Overview

Staking SYN tokens helps secure the Synergy Network, support validator operations, and earn rewards. Both delegators and validator operators can stake to participate in consensus and governance.

---

## 2. Staking Basics

| Role | Description |
|---|---|
| **Stakers** | Token holders who lock SYN to support network validators |
| **Validators** | Nodes that propose/validate blocks and require self-bonded SYN |
| **Delegators** | Users who stake SYN with a validator of their choice |

---

## 3. Benefits of Staking

**For Stakers:** Earn regular rewards, support network decentralization, participate in governance, no technical node requirements.

**For the Network:** Economic security against attacks, distributes influence across participants, encourages active validator participation.

---

## 4. Rewards Overview

- Distributed every epoch (~24 hours)
- Based on validator uptime, Synergy Points, amount staked, and reward tier
- Rewards auto-compound (optional via wallet toggle)

| Rate Type | Value |
|---|---|
| **Base Rate** | 5–15% APY (variable) |
| **Performance Bonus** | Up to 50% additional |
| **Minimum Stake (SNRG)** | 1,000 tokens |
| **Unstaking Cooldown** | 7-day cooldown period |

---

## 5. Staking via GUI Wallet

1. Open the Synergy Wallet Tool.
2. Go to the **Staking** tab.
3. Browse validator list.
4. Choose a validator and click **Delegate**.
5. Enter amount and confirm.
6. View staked SYN and projected APY in dashboard.

---

## 6. Staking via CLI

```bash
# Delegate stake
synergy-cli stake delegate --amount 1000 --validator <validator_address>

# Check rewards
synergy-cli stake rewards

# Claim rewards
synergy-cli stake claim

# View history
synergy-cli stake history
```

---

## 7. Staking via JSON-RPC

```json
// Stake tokens
{
  "jsonrpc": "2.0",
  "method": "synergy_stakeTokensDirect",
  "params": ["sYn...", "sYn...", "SNRG", 1000000],
  "id": 1
}

// Get staking info
{
  "jsonrpc": "2.0",
  "method": "synergy_getStakingInfo",
  "params": ["sYn..."],
  "id": 1
}

// Unstake tokens
{
  "jsonrpc": "2.0",
  "method": "synergy_unstakeTokens",
  "params": ["sYn...", "sYn...", "SNRG", 500000],
  "id": 1
}
```

---

## 8. Becoming a Validator

Requirements:

- Minimum SYN bond (set by governance, e.g., 25,000 SYN)
- High Synergy Score
- Node uptime and correct setup

```bash
synergy-cli validator register --name "MyNode" --bond 25000
```

---

## 9. Unstaking and Lock Periods

- Unstaking requires a **7-day cooldown period**
- Tokens remain visible but locked during cooldown
- Early slashing can occur if validator misbehaves during cooldown

---

## 10. Risks and Safeguards

- **Slashing Risks:** Delegated SYN is subject to slashing if validator acts maliciously.
- **Market Risks:** Token volatility and opportunity cost of locked tokens.
- **Technical Risks:** Validator hardware/software failures or smart contract bugs.

Always research validator history and Synergy Score before delegating. Use multiple validators to diversify risk.

---

## 11. Best Practices

- Start with small amounts to learn the process
- Stake across multiple validators to diversify risk
- Monitor validator performance regularly
- Re-stake rewards to compound returns
- Participate in governance

---

## 12. Conclusion

Staking is a critical function of the Synergy Network's collaborative consensus. It provides security, enables reward distribution, and grants governance power. By participating in staking, you actively contribute to the growth, decentralization, and resilience of the Synergy ecosystem.

---

*This staking guide provides general information and should not be considered financial advice.*

*© 2025 Synergy Network. Confidential.*
