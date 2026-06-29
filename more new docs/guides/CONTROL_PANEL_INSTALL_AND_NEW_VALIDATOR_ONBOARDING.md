# Synergy Node Control Panel Install And New Validator Onboarding

Version: 2026-06-04
Applies to: Synergy Node Control Panel `13.0.30` or newer
Audience: new validator operators on macOS, Linux, and Windows

This guide covers installing Synergy Node Control Panel, provisioning a new non-genesis validator, syncing it to the post-fork Testnet chain, bonding the required `50,000 SNRG`, and activating the validator through the Control Panel.

## Current Testnet Facts

| Item | Value |
| --- | --- |
| Chain ID | `1264` |
| Network ID | `synergy-testnet-v2` |
| Fork height | `204216` |
| Fork parent height | `204215` |
| Fork parent hash | `e209bd7554a06dfb052d5ff7ffd5664efc05e6cd1c5cadc9d139fa5bb9072816` |
| Consensus signing after fork | `FN-DSA` |
| Validator key algorithm | `FN-DSA-1024` |
| Parser mode | `fail_closed` |
| Public RPC | `https://testnet-core-rpc.synergy-network.io` |
| Public WS | `wss://testnet-core-ws.synergy-network.io` |
| Required validator stake | `50,000 SNRG` |
| Required validator stake in nWei | `50,000,000,000,000` |

A new validator must not use ML-DSA consensus keys, an unknown key type, or a genesis validator package. The Control Panel generates a fresh `synv1...` validator identity and writes explicit FN-DSA fork metadata into the workspace.

## What The Operator Does

The Control Panel is responsible for the repetitive work: writing configs, generating FN-DSA validator identity material, wiring bootstrap peers, registering with seeds, checking readiness, preparing safe actions, and recording local evidence.

The operator still needs to complete these decisions:

1. Install the app on the machine that will run the validator.
2. Open the validator P2P port, normally `5622/tcp`.
3. Provide a public IP address or DNS name that peers can reach.
4. Choose and protect an identity passphrase.
5. Fund the generated `synv1...` validator address with at least `50,000 SNRG`.
6. Click **Verify Onboarding** first to produce proof-only evidence.
7. Click **Run Safe Onboarding** when the proof is clean and the node should start, register, restore, or catch up.
8. Click **Stake Validator** and **Activate Validator** only when those explicit actions become eligible.

Do not manually edit genesis, validator registries, key algorithms, chain ID, network ID, fork metadata, quorum settings, peer allowlists, or validator key files.

## System Requirements

| Role | Minimum | Recommended |
| --- | --- | --- |
| Validator CPU | 4 cores | 4-8 cores |
| Validator memory | 8 GB RAM | 16 GB RAM |
| Validator disk | 100 GB SSD | 250 GB+ SSD |
| Network | Stable broadband | Static public IP or stable DNS |
| Clock | Automatic time sync enabled | NTP/Windows Time verified before activation |

The validator must be reachable from the public internet on its P2P port. Local RPC and metrics can stay private unless support explicitly asks otherwise.

## Download The App

Use the published Control Panel installer for the validator machine. Normal operators should not clone the source repository, install Node.js, install Rust, or build the app locally.

1. Open the official release page: `https://github.com/synergy-network-hq/synergy-node-control-panel-releases/releases`.
2. Open the latest release for Synergy Node Control Panel `13.0.30` or newer.
3. Download the artifact for the operating system that will run the validator:
   - macOS: `.dmg` for Apple Silicon or Intel, matching the machine.
   - Ubuntu/Debian Linux: `.deb` package. Use `.AppImage` only if that is the artifact published for the machine.
   - Windows: `.exe` installer.
4. Keep the downloaded installer until Settings confirms the installed version.
5. Do not install from screenshots, chat uploads, old working folders, or mirrors. If the downloaded filename or version does not match the release page, stop and download it again.

## Required Ports

| Port | Direction | Purpose |
| --- | --- | --- |
| `5622/tcp` | Inbound | Validator P2P |
| `5640/tcp` | Localhost | JSON-RPC and gRPC |
| `5660/tcp` | Localhost | WebSocket |
| `5680/tcp` | Inbound or local | Discovery |
| `6030/tcp` | Localhost or private | Metrics |

The public bootstrap layer is:

| Service | DNS | Direct IP fallback |
| --- | --- | --- |
| Bootnode 1 | `bootnode1.synergy-network.io:5620` | `170.64.187.206:5620` |
| Bootnode 2 | `bootnode2.synergy-network.io:5620` | `146.190.210.121:5620` |
| Bootnode 3 | `bootnode3.synergy-network.io:5620` | `157.245.226.240:5620` |
| Seed 1 | `http://seed1.synergy-network.io:5621` | `http://170.64.187.206:5621` |
| Seed 2 | `http://seed2.synergy-network.io:5621` | `http://146.190.210.121:5621` |
| Seed 3 | `http://seed3.synergy-network.io:5621` | `http://157.245.226.240:5621` |

## Install On macOS

Dependencies:

- macOS 12 Monterey or newer; macOS 14 or newer is recommended.
- Administrator password for first launch, network prompts, and firewall changes.
- Apple Silicon or Intel installer that matches the machine.
- Automatic network time enabled.

Install:

1. Download the latest macOS `.dmg` from the Synergy Node Control Panel GitHub Releases page.
2. Open the `.dmg`.
3. Drag **Synergy Node Control Panel** into **Applications**.
4. Open the app from **Applications**.
5. If Gatekeeper prompts, choose **Open** only if the file came from the official release.
6. Allow incoming connections if macOS asks. A validator needs inbound P2P.
7. Open **Settings** and confirm the app version is `13.0.30` or newer.

macOS does not need Node.js, Rust, Git, or Xcode for normal installer usage.

Optional manual time check:

```bash
sudo systemsetup -setusingnetworktime on
sntp -sS time.apple.com
```

## Install On Ubuntu Or Debian Linux

Dependencies:

- Ubuntu 22.04 LTS, Ubuntu 24.04 LTS, Debian, or a compatible desktop distribution.
- A graphical desktop session or remote desktop session. The Control Panel is a desktop app.
- `sudo` rights for package installation, firewall changes, and time service repair.
- Automatic NTP enabled through `systemd-timesyncd`, `chrony`, or another NTP client.

Install common GUI/runtime dependencies first:

```bash
sudo apt-get update
sudo apt-get install -y \
  ca-certificates \
  curl \
  libgtk-3-0 \
  libwebkit2gtk-4.1-0 \
  libayatana-appindicator3-1 \
  librsvg2-2 \
  libssl3 \
  libfuse2 \
  ufw
```

Install the downloaded `.deb`:

```bash
sudo dpkg -i synergy-node-control-panel_*_amd64.deb
sudo apt-get install -f
```

Open the validator P2P port:

```bash
sudo ufw allow 5622/tcp
sudo ufw status
```

Enable automatic time sync:

```bash
sudo timedatectl set-ntp true
timedatectl status
```

Linux does not need Node.js, Rust, or Git for normal installer usage.

## Install On Windows

Dependencies:

- Windows 10 64-bit or Windows 11 64-bit.
- Administrator rights for installation and firewall changes.
- Windows Time service enabled.
- Microsoft Visual C++ Redistributable x64 only if Windows reports missing runtime DLLs.

Install:

1. Download the latest Windows `.exe` installer from the Synergy Node Control Panel GitHub Releases page.
2. Run the installer.
3. If SmartScreen appears, use **More info** then **Run anyway** only when the file came from the official release.
4. Start **Synergy Node Control Panel** from the Start menu.
5. Allow Windows Defender Firewall access when prompted.
6. Open **Settings** and confirm the app version is `13.0.30` or newer.

Open the validator P2P port from an Administrator PowerShell:

```powershell
New-NetFirewallRule `
  -DisplayName "Synergy Testnet Validator P2P 5622" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 5622 `
  -Action Allow
```

Resync Windows Time from an Administrator PowerShell if needed:

```powershell
w32tm /resync /force
```

Install the Microsoft Visual C++ Redistributable x64 only if Windows reports missing runtime DLLs.

## First Launch

1. Start the Control Panel.
2. Open **Settings**.
3. Confirm these values:
   - Chain ID `1264`.
   - Network ID `synergy-testnet-v2`.
   - Checkpoint fork `204216`.
   - Consensus algorithm `FN-DSA`.
   - Parser mode `fail_closed`.
4. Open the validator or onboarding screen.
5. Click **Resync Time** before provisioning or activation.

The **Resync Time** button runs the platform-specific time synchronization command and writes evidence under the local workspace logs:

- macOS: `sntp -sS time.apple.com`, `systemsetup -setusingnetworktime on`, and the current network time server check.
- Linux: `timedatectl set-ntp true`, then `chronyc -a makestep` or `ntpdate -u pool.ntp.org` when available.
- Windows: `w32tm /resync /force`.

If the button reports a warning, rerun the matching command with administrator privileges or fix the OS time service before activation.

## Autonomous Onboarding Flow

The Control Panel uses an autonomous, guarded flow. The app performs the mechanical steps, records evidence, and blocks unsafe actions. The operator supplies the public endpoint, identity passphrase, funding, and final approvals.

Use **Verify Onboarding** before any live action. It is a proof-only dry run: it does not start the runtime, resync the operating-system clock, restore snapshots, mutate chain state, submit stake, or submit activation. Use **Run Safe Onboarding** after reviewing the proof. Safe onboarding may resync time, start the runtime, register seeds, verify or restore trusted snapshots, and catch up, but staking and activation remain separate explicit buttons.

| Phase | Control Panel responsibility | Operator responsibility | Safe result |
| --- | --- | --- | --- |
| Verify | Run proof-only checks for role, address, fork metadata, FN-DSA key metadata, workspace, seed posture, stake state, and activation gates. | Review the generated evidence before allowing live actions. | Operator can see blockers without mutating runtime, chain state, stake, or activation. |
| Provision | Generate `synv1...` identity, FN-DSA key material, fork-aware configs, bootstrap peers, seed fallbacks, manifests, and logs. | Choose validator role, public IP/DNS, workspace, and passphrase. | Workspace exists with post-fork metadata and signing disabled until activation. |
| Snapshot restore | Discover trusted snapshot catalog data when available, verify snapshot metadata, and refuse unverified targets. | Use the Control Panel flow only; do not copy arbitrary chain data into the workspace. | Local chain data is canonical or the app tells you it must be reset/restored. |
| Catch-up | Stop the runtime, speed-sync the canonical chain, restart the runtime, and run preflight. | Click **Sync Catch Up** when lag is high and wait for the result. | Node is near public chain head or shows the next repair action. |
| Shadow | Keep consensus duties disabled while the node observes canonical blocks, QCs, peers, and would-have-voted behavior. | Keep the node online and do not force activation. | Shadow observation passes readiness gates. |
| Stake | Check funding, local signing key, runtime wallet import, and canonical chain state before staking. | Fund the validator `synv1...` address and click **Stake Validator** only when enabled. | `50,000 SNRG` is bonded for the validator identity. |
| Activation | Submit activation only after preflight, source-majority, full-shadow-epoch, and duty-gate evidence all pass. | Click **Activate Validator** only when the app reports the activation policy is clear, then keep the node online. | Validator enters the active set after transaction inclusion and epoch scheduling. |

The app should never require an operator to hand-edit `node.toml`, genesis files, validator registries, fork metadata, key algorithm fields, or local chain-state internals during normal onboarding.

## Provision A New Validator

1. Open the setup flow in the Control Panel.
2. Choose `validator`.
3. Enter a display name.
4. Enter the public IP address or DNS name for this machine.
5. Enter an identity passphrase with at least 8 characters.
6. Confirm the workspace path.
7. Click **Provision Node**.

The Control Panel generates:

- a new `synv1...` validator address;
- FN-DSA validator identity material;
- encrypted private key exports;
- `node.toml` with `chain_id=1264`, `network_id=synergy-testnet-v2`, and fork metadata;
- `peers.toml` with public relayers, bootnodes, seed services, and direct seed-IP fallbacks;
- funding and bootstrap manifests;
- local logs and evidence directories.

Do not reuse a genesis validator package. Do not replace the generated `synv1...` validator address with a `synw...` wallet address.

## Start And Sync

1. Open the validator node page.
2. Click **Verify Onboarding** and review the proof-only result.
3. Click **Run Safe Onboarding** when the proof shows only retryable start, sync, seed, or snapshot work remains.
4. Wait for the runtime process and listeners to become healthy.
5. Click **Sync Catch Up** if the app reports sync lag, or **Bootstrap / Reconnect** if the app reports peer discovery issues.
6. Wait until the validator is within `2` blocks of public chain head.

The node syncs while validator signing and activation remain disabled. It should not vote, propose, or aggregate QCs before activation.

Local sync check:

```bash
curl -s http://127.0.0.1:5640 \
  -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","id":1,"method":"synergy_getLatestBlock","params":[]}'
```

Public sync check:

```bash
curl -s https://testnet-core-rpc.synergy-network.io \
  -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","id":1,"method":"synergy_getLatestBlock","params":[]}'
```

## Snapshot Restore, Catch-Up, Shadow, And Activation States

Use the state names shown in the Control Panel to decide what to do next.

| State or step | Meaning | Operator action |
| --- | --- | --- |
| `REGISTERED` | Local validator identity exists. | Confirm the address begins with `synv1`. |
| `KEY_BOUND` | Generated FN-DSA identity material is present. | Keep the key files private; do not replace them. |
| `STAKE_REQUIRED` | The validator needs funding or bonded stake. | Fund the validator address with at least `50,000 SNRG`. |
| `STAKE_SUBMITTED` | Stake transaction was submitted. | Wait for inclusion, then refresh preflight. |
| `STAKE_CONFIRMED` | Bonded stake is visible. | Continue sync and preflight. |
| `SYNCING` | The node is behind public chain head. | Keep it running or click **Sync Catch Up**. |
| `SNAPSHOT_VERIFIED` | A trusted snapshot or snapshot catalog entry was verified. | Let the app continue restore/replay; do not swap in local chain data. |
| `REPLAYING` | The node is replaying canonical state after restore or catch-up. | Wait for replay to finish. |
| `SHADOW` | The node observes consensus without signing real votes. | Keep it online until shadow checks pass. |
| `READY` | Preflight is passing and the validator can proceed. | Stake or activate, depending on what remains. |
| `PENDING_ACTIVATION` | Activation transaction is submitted or scheduled. | Keep the node online until it appears active. |
| `ACTIVE` | Validator is participating in consensus. | Monitor Dashboard, Validator Lifecycle, Connections, Rewards, and Logs. |
| `QUARANTINED` | The app detected divergence or a safety issue and disabled consensus duties. | Use verified snapshot self-realignment and shadow observation before rejoin. |
| `FAILED_CLOSED` | A consensus-critical check failed. | Do not bypass. Fix the failed check and rerun preflight. |

The guarded self-realignment path may show lower-level recovery steps such as evidence preservation, snapshot discovery, snapshot downloading, snapshot verified, snapshot restored, speed syncing, caught up, shadow observing, shadow passed, and ready to rejoin. Continue the Control Panel flow through those states. Do not force a validator to `ACTIVE` while any fail-closed, quarantine, snapshot verification, replay, or shadow gate is unresolved.

## Register With Seed Servers

The Control Panel registers the validator with seed servers during provisioning and runtime repair. If preflight reports seed registration missing:

1. Click **Refresh Seeds** or **Re-register**.
2. Wait for the action to finish.
3. Run **Activation Preflight** again.

Seed registration publishes the validator dial target so other nodes can discover it. The generated configuration includes direct seed-IP fallbacks so onboarding can continue even while stale seed DNS records are being corrected.

## Fund The Validator Address

Copy the generated validator address. It must start with `synv1`.

Request or send at least `50,000 SNRG` to that exact validator address. Do not fund a `synw...` wallet for validator staking.

Check liquid balance:

```bash
VALIDATOR_ADDRESS='synv1...'

curl -s https://testnet-core-rpc.synergy-network.io \
  -H 'Content-Type: application/json' \
  --data "{\"jsonrpc\":\"2.0\",\"id\":1,\"method\":\"synergy_getBalance\",\"params\":[\"$VALIDATOR_ADDRESS\"]}"
```

The RPC value is in nWei. `50,000 SNRG` is `50000000000000` nWei.

## Bond 50,000 SNRG

1. Run **Activation Preflight**.
2. Confirm the validator has enough liquid SNRG.
3. Click **Stake Validator**.
4. Wait for a transaction hash and final inclusion.
5. Run **Activation Preflight** again.
6. Confirm **Bonded stake** passes.

The Control Panel submits the stake transaction for the generated validator address. It does not stake from a separate regular wallet identity.

**Run Safe Onboarding** does not bond stake. The operator must click **Stake Validator** after funding and preflight confirm that staking is eligible.

Check bonded stake:

```bash
VALIDATOR_ADDRESS='synv1...'

curl -s https://testnet-core-rpc.synergy-network.io \
  -H 'Content-Type: application/json' \
  --data "{\"jsonrpc\":\"2.0\",\"id\":1,\"method\":\"synergy_getStakedBalance\",\"params\":[\"$VALIDATOR_ADDRESS\",\"SNRG\"]}"
```

The bonded balance must be at least `50000000000000` nWei.

## Activate The Validator

Run **Activation Preflight** and confirm all required checks pass:

- Validator role is correct.
- Validator address starts with `synv1` and is canonical.
- Chain ID is `1264`.
- Network ID is `synergy-testnet-v2`.
- Fork metadata matches height `204216` and the expected parent hash.
- Consensus algorithm is `FN-DSA`.
- Validator key algorithm is `FN-DSA-1024`.
- Parser mode is `fail_closed`.
- Workspace genesis and local chain state are canonical for chain `1264`.
- Local qRPC is responding.
- Runtime wallet has imported the validator identity and can sign.
- P2P listener is reachable.
- The node is within `2` blocks of public chain head.
- Relayer peers are visible.
- Seed registration is present.
- Validator wallet has funding.
- Bonded stake is at least `50,000 SNRG`.
- Source-majority proof uses at least `3` trusted non-public-RPC sources.
- A full shadow epoch passed with at least `1,000` observed blocks, `0` mismatches, and `0` missed observations.
- Duty-gate proof shows `can_vote`, `can_propose`, `can_aggregate_qc`, `can_count_toward_quorum`, `can_enter_proposer_schedule`, `can_serve_as_canonical_source`, and `shadow_signs_real_votes` are all false before activation.

When every preflight and activation-policy gate passes, click **Activate Validator**. The validator becomes eligible at the deterministic epoch boundary after the activation transaction finalizes.

Fail-closed behavior is intentional. If canonical chain, fork metadata, FN-DSA key role, parser mode, local signing key, runtime wallet, or chain-state checks fail, the app should block staking or activation instead of letting the validator sign on an unsafe state.

## Request Rejoin At The Epoch Boundary

After a quarantined or onboarding validator has passed the required shadow and activation/rejoin gates, click **Request Rejoin**. The button does not force consensus duties open immediately. It writes an armed rejoin request into the validator workspace for the local controller.

The controller must then:

- target the next epoch boundary, not the current mid-epoch height;
- execute `request-rejoin` only from the first block of that epoch through block `epoch_start + 9`;
- use the epoch-boundary block height/hash as the rejoin proof;
- roll forward to the following epoch if the first 10-block entry window is missed;
- keep consensus duties closed until the runtime confirms rejoin.

This keeps operators from having to press a button at the exact boundary while still preventing validators from joining consensus mid-epoch.

## Confirm Onboarding Success

After activation, confirm the validator appears in the validator set:

```bash
curl -s https://testnet-core-rpc.synergy-network.io \
  -H 'Content-Type: application/json' \
  --data '{"jsonrpc":"2.0","id":1,"method":"synergy_getValidators","params":[]}'
```

Confirm Atlas visibility:

```bash
curl -s https://testnet-atlas-api.synergy-network.io/api/v1/validators
```

A successful onboarding proof should include:

- setup completed in the Control Panel;
- generated validator address begins with `synv1`;
- FN-DSA fork metadata present in `node.toml`;
- local validator height within `2` blocks of public RPC;
- seed registration confirmed;
- funding receipt for at least `50,000 SNRG`;
- stake transaction hash and final receipt;
- activation transaction hash and final receipt;
- public RPC validator set includes the validator;
- Atlas sees the validator after indexing catches up.

## Troubleshooting

### Clock Skew

Symptoms include failed preflight, rejected messages, or repeated onboarding failures. Click **Resync Time** in the Control Panel, then rerun **Activation Preflight**. If it still fails, use the platform-specific manual time commands in this guide.

### Seed Registration Fails

Confirm direct seed services are reachable:

```bash
curl -s http://170.64.187.206:5621/healthz
curl -s http://146.190.210.121:5621/healthz
curl -s http://157.245.226.240:5621/healthz
```

If direct seed IPs respond but DNS seed names do not, keep using the current Control Panel build. It includes direct seed-IP fallbacks in generated configs.

### Validator Is Behind

Use **Sync Catch Up** or **Bootstrap / Reconnect**. Do not activate while behind. Activation requires the node to be within `2` blocks of chain head.

### Snapshot Restore Or Catch-Up Is Blocked

Do not manually copy a random database, snapshot, or old validator workspace into the new validator directory. Use **Sync Catch Up** first. If the Control Panel reports snapshot discovery, snapshot verification, or replay problems, preserve the evidence path shown in the app and fix the first failed readiness check. Activation must wait until the app reports canonical chain state, local RPC readiness, and sync gap within `2` blocks.

### Shadow State Does Not Finish

Shadow mode means the validator is observing consensus while real voting, proposing, QC aggregation, and quorum counting remain disabled. Keep the runtime online, check peer visibility, and rerun **Activation Preflight** after the required shadow period. Do not force activation from a shell.

### Funding Is Missing

Confirm the receiver address exactly matches the validator `synv1...` address. A regular wallet address does not satisfy validator staking.

### Stake Button Is Disabled

Run **Activation Preflight** and resolve the first failing check. Staking requires a validator workspace, local RPC, canonical chain/fork metadata, validator wallet import, and enough liquid SNRG.

### Activation Button Is Disabled

Activation requires sync, peer visibility, seed registration, funding, bonded stake, and post-fork FN-DSA metadata. Do not bypass this gate manually.

### Fail-Closed Or Quarantine Appears

Fail-closed or quarantine states are safety outcomes, not UI errors. The validator is prevented from signing until the cause is fixed. Common causes are wrong chain ID, wrong network ID, missing fork metadata, stale chain data, non-FN-DSA key material, parser mode not set to `fail_closed`, local RPC not ready, or wallet/key import failure. Use **Open logs**, **Open workspace**, **Sync Catch Up**, **Bootstrap / Reconnect**, and **Activation Preflight** from the app, then share the generated evidence file with support if the same check keeps failing.

## Developer Build From Source

Normal operators should use the published installer. Developers can build locally:

```bash
npm install
npm run build
cargo test --manifest-path control-service/Cargo.toml fork_metadata_matches_checkpointed_fndsa_fork
cargo test --manifest-path control-service/Cargo.toml setup_node_writes_non_genesis_validator_bootstrap_inputs
```

Tagged Control Panel releases trigger GitHub Actions installer builds. Do not tag a new release until the live onboarding proof succeeds.
