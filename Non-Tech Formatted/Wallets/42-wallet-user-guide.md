# Wallet User Guide
## Synergy Network

---

## 1. Overview

The Synergy Wallet is your secure interface for managing SYN tokens, staking, governance participation, SNS names, and UMA cross-chain identity.

**Key Features:** Create and import wallets, send and receive SYN tokens, stake tokens and manage rewards, register and manage SNS names, access transaction history, UMA-compatible cross-chain mapping.

---

## 2. Installing the Wallet

**GUI Version:**
- Download from: https://synergy-network.io/wallet
- Available for macOS, Windows, Linux
- Install via standard OS installer

**CLI Version:**
```bash
git clone https://github.com/synergy-network/wallet-cli.git
cd wallet-cli
cargo build --release
```

---

## 3. Creating a Wallet

**GUI:** Open the app → Click **Create Wallet** → Save 24-word seed phrase → Set PIN.

**CLI:**
```bash
synergy-cli wallet create
```

---

## 4. Importing a Wallet

**GUI:** Click **Import Wallet** → Paste 24-word seed phrase.

**CLI:**
```bash
synergy-cli wallet import --mnemonic "..."
```

---

## 5. Viewing Balance and History

**GUI:** Main dashboard displays SYN balance, staked amount, and recent transactions.

**CLI:**
```bash
synergy-cli wallet balance
synergy-cli wallet history
```

---

## 6. Sending and Receiving SYN

**GUI:** Go to **Send** tab → input recipient address and amount → confirm and send. Your address is under the **Receive** tab (QR code available).

**CLI:**
```bash
synergy-cli transaction send --to <address> --amount 10 --memo "Thanks!"
```

---

## 7. Staking and Rewards

**GUI:** Go to **Staking** section → Choose a validator → Delegate → View rewards under **Rewards** tab.

**CLI:**
```bash
synergy-cli stake delegate --amount 100 --validator <address>
synergy-cli stake rewards
```

---

## 8. SNS Name Registration

**GUI:** Go to **SNS** tab → Check name availability → Register `.syn` name.

**CLI:**
```bash
synergy-cli sns register --name myname.syn --years 2
```

---

## 9. UMA Mapping (Cross-Chain ID)

**GUI:** Go to **Settings → UMA Mapping** → Link ETH/BTC/SOL addresses.

**CLI:**
```bash
synergy-cli uma map --eth 0x... --btc bc1... --sol 3F...
```

---

## 10. Wallet Backup and Security

- Backup your encrypted wallet file
- Store your 24-word seed phrase offline
- Use biometric or 2FA protection if supported (GUI only)

---

## 11. Troubleshooting

- **Wallet not opening?** Reinstall or check dependencies.
- **Missing balance?** Ensure you're connected to the correct network.
- **Error in CLI?** Run with `--verbose` and check `~/.synergy/wallet.log`

---

## 12. Conclusion

The Synergy Wallet offers powerful tools for secure digital asset management and on-chain participation. Whether using the GUI or CLI, you have full control of your keys, tokens, and reputation in the Synergy ecosystem.

---

*© 2025 Synergy Network. Confidential.*
