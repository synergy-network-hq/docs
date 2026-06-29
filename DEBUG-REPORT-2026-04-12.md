# Synergy Testnet-Beta Debug Report

**Date:** 2026-04-12
**Scope:** Validator connectivity instability, asymmetric peer state, chain unable to produce blocks

---

## TL;DR

The chain cannot produce blocks because only 1–2 of the required 3 live validators are consistently visible. Four root causes were identified: (1) Validator 5 had a stale root-owned process stealing port 5622 — **FIXED**; (2) Validator 3 is completely down; (3) Validator 4 sends an empty `validator_address` so it's never counted as "live"; (4) a duplicate-peer-detection loop causes Val 1 ↔ Val 5 connections to cycle every ~2 minutes. Getting the chain running requires bringing at least one more validator online (Val 3 or Val 4).

---

## 1. Root Causes Identified

### 1.1 Validator 5 — Dual Process / Port Conflict ✅ FIXED

**Symptom:** Val 5 would report outbound connections to other validators, but those validators never saw Val 5 connect inbound. Asymmetric connection state.

**Root Cause:** Two `synergy-testbet` processes were running on 194.163.183.166:
- **PID 3207** (root, started Apr 10) — held port 5622 via TCP LISTEN
- **PID 28569** (justin, started later) — could not bind port 5622, so it could dial out but never accept inbound connections

Inbound connections from other validators hit root's stale process (PID 3207), which was non-functional. Justin's process could dial out and complete handshakes, but the remote side's return path went to the wrong process.

**Fix Applied:**
```bash
sudo kill 3207          # killed root's stale process
# restarted justin's process with correct env vars
```

After restart, Val 5 successfully bound port 5622 and began accepting inbound connections. Handshakes with Val 1 now complete bidirectionally.

### 1.2 Validator 3 (62.146.182.209) — Completely Down ❌ UNRESOLVED

**Symptom:** "Connection refused" from every other validator when dialing genesisval3.synergynode.xyz:5622. TCP connectivity tests from Val 1 and Val 5 both fail.

**Evidence:**
```
Testing connectivity to 62.146.182.209:5622... UNREACHABLE
```

No SSH access was provided for this machine. The node is either shut down, crashed, or has a firewall blocking port 5622.

**Action Required:** SSH into 62.146.182.209, check if the synergy process is running, restart if needed.

### 1.3 Validator 4 (71.182.137.118) — Empty Validator Address ❌ UNRESOLVED

**Symptom:** Val 4 establishes TCP connections and completes handshakes, but sends an **empty `validator_address`** in its status messages. The consensus code in `consensus_algorithm.rs` and `dual_quorum.rs` only counts peers with non-empty `validator_address` values that exist in the active validator registry. Val 4 is therefore invisible to consensus — it's "connected" at the P2P layer but not "live" for block production.

**Likely Cause:** Val 4's `node.toml` is missing or has an empty `[identity] address` field, OR its `SYNERGY_VALIDATOR_ADDRESS` environment variable wasn't set at launch.

**Action Required:** SSH into 71.182.137.118, check `node.toml` for `identity.address` and verify the launch environment includes `SYNERGY_VALIDATOR_ADDRESS`.

### 1.4 Duplicate Peer Detection Loop — Connection Cycling ⚠️ CODE FIX NEEDED

**Symptom:** Val 1 and Val 5 continuously connect and disconnect from each other in a ~2 minute cycle:
1. Val 1 dials Val 5 (outbound) — connection established
2. Val 5 simultaneously dials Val 1 (outbound) — second connection established
3. Val 1 detects duplicate peer, runs `resolve_duplicate_peer_session()`, decides to keep one and drop the other
4. Val 5 sees the dropped connection, doesn't know it was a deliberate dedup — treats it as a failure and redials on next bootstrap refresh (every 120 seconds)
5. Cycle repeats

**Root Cause in Code:** The duplicate resolution in `networking.rs` uses `DuplicateResolution::KeepExisting` vs `ReplaceExisting`, but the decision is made **unilaterally by one side**. The other side has no knowledge that a dedup occurred and interprets the disconnect as a connection failure, triggering a redial.

**Recommended Fix:** Make duplicate resolution **deterministic on both sides** by using a consistent tiebreaker (e.g., compare node IDs or validator addresses lexicographically — the node with the "lower" ID always keeps its outbound connection, the "higher" ID always drops its outbound). This way both sides independently reach the same conclusion about which connection to keep, and neither side redials unnecessarily.

**Location:** `src/p2p/networking.rs` — `resolve_duplicate_peer_session()` function and the duplicate detection logic in the connection handler.

---

## 2. Secondary Issues

### 2.1 Self-Penalization Cascade

Failed block proposals (caused by insufficient live validators) trigger the reputation/penalization system. Val 1's reputation dropped from 100 → 99 → 98.01, and the registry-active validator count dropped from 5 → 3. This means validators are being **deactivated by the protocol itself** due to failed proposals — making it even harder to reach quorum.

**Recommendation:** Either disable self-penalization during testnet bootstrap, or add a guard that skips penalization when the failure reason is "insufficient validators" (since that's a network-wide issue, not proposer misbehavior).

**Location:** `src/consensus/consensus_algorithm.rs` — search for "reputation reduced" and the `penalize_proposer` logic.

### 2.2 All Seed Servers Dead

`seed1.synergynode.xyz:5621`, `seed2.synergynode.xyz:5621`, `seed3.synergynode.xyz:5621` all return connection errors. Seed servers are used for initial peer discovery. While not critical (validators have `additional_dial_targets` and `persistent_peers` configured directly), dead seed servers slow down bootstrap and prevent any new nodes from discovering peers.

### 2.3 Validator 2 Location Mismatch

The SSH credentials provided (`rob@62.146.182.207`) land on the **same machine as Validator 1** (justin@62.146.182.207). The `rob` user account has an empty home directory with no synergy workspace. DNS shows `genesisval2.synergynode.xyz` resolves to **62.146.182.208** (different IP). Val 2 appears to be running at .208 based on TCP connection state observed from other validators.

**Action Required:** Provide correct SSH credentials for 62.146.182.208.

### 2.4 Zombie Processes

Multiple defunct `[synergy-testbet]` zombie processes exist on both machines. These are children of the Electron Node Control Panel that weren't reaped. Cosmetic issue — doesn't affect functionality, but indicates the control panel's process management needs a `waitpid()` call.

---

## 3. What Was Fixed

| Action | Result |
|---|---|
| Killed stale root process (PID 3207) on Val 5 | Port 5622 freed |
| Restarted Val 5 with correct env vars | Binds port 5622, accepts inbound connections |
| Restarted Val 1 for clean state | Fresh connections, handshakes completing |
| Verified genesis.json matches across Val 1 & Val 5 | SHA256 `b1d9dff8...` identical — no genesis mismatch disconnects |
| Verified DNS resolution for all 5 validators | All resolve correctly |

---

## 4. Current Chain State

```
Val 1: "⏳ Insufficient live validators: 1 live, 3 registry-active, 3 required"
Val 5: "⏳ Insufficient live validators: 2 live, 5 registry-active, 3 required"
```

The chain needs **3 live validators** to produce blocks. Currently only Val 1 and Val 5 are online and can see each other (intermittently, due to the duplicate peer loop). One more validator must come online to reach quorum.

---

## 5. Action Plan — Priority Order

### Immediate (get chain producing blocks)

1. **Bring Validator 3 online** — SSH into 62.146.182.209, check process, restart if needed. This is the fastest path to quorum since it just needs to be turned on.

2. **Fix Validator 4's empty address** — SSH into 71.182.137.118, verify `node.toml` has `identity.address` set and `SYNERGY_VALIDATOR_ADDRESS` env var is present at launch.

3. **Clarify Validator 2 access** — Provide SSH credentials for 62.146.182.208 (the actual Val 2 host, not .207).

### Short-term (stabilize connections)

4. **Fix duplicate peer detection** — Make resolution deterministic on both sides using a consistent tiebreaker (lexicographic comparison of node/validator IDs). Location: `src/p2p/networking.rs`.

5. **Address self-penalization cascade** — Add a guard to skip proposer penalization when failure reason is insufficient validators. Location: `src/consensus/consensus_algorithm.rs`.

### Medium-term (infrastructure)

6. **Restart or replace seed servers** — seed1/2/3.synergynode.xyz:5621 are all dead.

7. **Fix zombie process reaping** in the Node Control Panel's process management.

---

## 6. Files Reference

| File | Role |
|---|---|
| `src/p2p/networking.rs` | P2P connections, handshake, duplicate detection, peer state |
| `src/consensus/consensus_algorithm.rs` | Block production loop, live validator counting, penalization |
| `src/consensus/dual_quorum.rs` | Quorum checks, vote tracking |
| `node-control-panel/docs/developer/testnet-beta-validator-update-workflow.md` | Config change workflow, installer builds |
| `docs/config-guide.md` | Full config reference |

---

## 7. Validator Quick Reference

| Validator | Address | Host | Port | Status |
|---|---|---|---|---|
| Val 1 | `synv114cvu472rkdgpmzvkj70zk9tu8cqqlu4x9ra` | 62.146.182.207 | 5622 | ✅ Running |
| Val 2 | `synv11h3yrx...` | 62.146.182.208 | 5622 | ⚠️ No SSH access |
| Val 3 | `synv11g5gg2...` | 62.146.182.209 | 5622 | ❌ Down |
| Val 4 | `synv119gw09...` | 71.182.137.118 | 5622 | ⚠️ Empty address |
| Val 5 | `synv11mvlgy72uq7kuh200qnxv67zrqjugz267k46` | 194.163.183.166 | 5622 | ✅ Running |
