# Synergy Testnet-Beta Debug Report v2

**Date:** 2026-04-12 20:20 UTC  
**Status:** Chain NOT producing blocks. All 4 online validators stuck at "1 live."

---

## TL;DR

Despite 4 validators being online (Val 1, 2, 3, 5) with TCP connections established and handshakes completing, the chain cannot produce blocks because every validator only sees **1 live validator** (itself). The root cause is a **connection churn loop**: the P2P bootstrap cycle fires every 10 seconds when the validator mesh isn't complete, creating new outbound connections that trigger duplicate peer detection, which drops existing connections before status exchange can happen. Without status exchange, the mesh never "completes," so bootstrap stays in fast mode (10s), perpetuating the cycle. Two secondary cascading issues (aggressive penalization and restrictive eligibility thresholds) make recovery impossible once the initial window is missed.

---

## 1. Issues Found (Priority Order)

### 1.1 CRITICAL: Bootstrap Refresh Loop Prevents Connection Stability

**What happens:**
1. Validators start and the bootstrap thread resolves dial targets (all other validators + bootnodes)
2. Outbound TCP connections are created to all targets
3. Simultaneously, the OTHER validators also dial this validator (inbound connections)
4. The handshake handler detects duplicate peers (same identity, two connections) and resolves by keeping one, dropping the other
5. The dropped connection's peer entry is removed from the peer map
6. Before the surviving connection can complete a status exchange, the bootstrap refresh fires again (every 10 seconds in "fast" mode)
7. The new bootstrap cycle creates fresh outbound connections to the same targets
8. Duplicate detection fires again, dropping the connections that just formed
9. Repeat forever

**Evidence from Val 2 logs (20:16:23 UTC):**
```
Handshake received (Val 3) → Incoming peer (Val 3 inbound) → Peer disconnected (Val 3 outbound)
Incoming peer (Val 1 inbound) → Peer disconnected (Val 1 outbound)  
Peer disconnected (bootnode2) → Peer disconnected (bootnode3) → Peer disconnected (bootnode1)
```
All 5 peer disconnections happen within the same second. Then at 20:16:45 (22s later, next bootstrap + seed server timeout), more disconnections.

**Root cause in code:**
- `FAST_BOOTSTRAP_REFRESH_SECS = 10` in `src/p2p/networking.rs:35`
- `current_bootstrap_refresh_interval()` at line 919 returns fast interval when `status_ready_validator_participants < required_validators`
- `status_ready_validator_participants()` at line 875 requires `peer_has_remote_status()` which checks `status_received_at.is_some()` — peers must have completed a status exchange, not just a handshake
- Connections get churned before status exchange happens → `status_ready` count stays at 1 → bootstrap stays in fast mode → connections keep churning

**This is the #1 blocker. Nothing else matters until this is fixed.**

### 1.2 HIGH: Penalization Cascade Kills Validators Within Seconds

Even when connections briefly stabilize (we achieved blocks 1-3 in an earlier attempt), the penalization system rapidly deactivates validators.

**The math:**
- `MISSED_VOTE_UPTIME_PENALTY = 2.5` — each missed block reduces uptime by 2.5 points
- `is_eligible()` requires `uptime_percentage >= 95.0` — validators start at 100.0
- After **3 leader timeouts** for the same validator: uptime = 100 - 7.5 = 92.5 < 95.0 → **ineligible**
- `MISSED_VOTE_JAIL_THRESHOLD = 3` — after 3 missed votes, validator is also **jailed** (status changes from Active)
- Recovery rate is only +0.5 per successful block participation

**Cascade sequence:**
1. Block production starts, but leader rotation includes Val 4 (empty address) or a temporarily disconnected validator
2. Those leaders can't propose → 15-second timeout fires → `apply_leader_timeout_penalty()` → uptime -2.5
3. After 3 timeouts for any validator: ineligible/jailed → fewer active validators
4. Fewer active validators → more leaders are "offline" → more timeouts → more penalties
5. Within ~60 seconds, the live count drops from 4 to 1

**Location:** `src/consensus/consensus_algorithm.rs` lines 1689-1737, `src/validator.rs` lines 193-201, 215-222, 242-247

### 1.3 HIGH: Validator 4 — Empty Validator Address

Val 4 (71.182.137.118) establishes TCP connections and completes handshakes, but sends an **empty `validator_address`**. The consensus code only counts peers with non-empty `validator_address` values that exist in the active validator registry.

**Impact:** Val 4 consumes a slot in the leader rotation but can never propose blocks, triggering timeout penalties for every round it's selected as leader.

**Action required:** SSH into 71.182.137.118, check `node.toml` for `[identity] address` field and verify `SYNERGY_VALIDATOR_ADDRESS` environment variable.

### 1.4 MEDIUM: Duplicate Peer Resolution Asymmetry

When two validators simultaneously dial each other, duplicate detection resolves on one side (drop one connection) but the other side may not know a dedup occurred. The dropped side treats it as a failure and redials on next bootstrap refresh.

**Location:** `src/p2p/networking.rs` — `resolve_duplicate_connection()` at line 589

### 1.5 LOW: All Seed Servers Dead

`seed1/2/3.synergynode.xyz:5621` all return connection errors. Each failed request adds ~3-6 seconds of blocking time to the bootstrap cycle, further delaying peer discovery.

### 1.6 LOW: Zombie Processes

Multiple `[synergy-testbet] <defunct>` zombie processes on all servers from previous launches via the Node Control Panel. Cosmetic — doesn't affect functionality.

---

## 2. What Has Been Attempted

### Successfully Fixed
| # | Action | Result |
|---|--------|--------|
| 1 | Killed stale root process (PID 3207) on Val 5 that was stealing port 5622 | Port 5622 freed, Val 5 can accept inbound |
| 2 | Verified genesis.json matches across all validators | SHA256 identical — no genesis mismatch |
| 3 | Verified DNS resolution for all 5 validators | All resolve correctly |
| 4 | Identified correct SSH credentials for all accessible validators | Val 1, 2, 3, 5 accessible |
| 5 | Lowered `min_validators`, `status_ready_min_validators`, `validator_vote_threshold` from 3 → 2 on Val 1, 2, 3, 5 | Config updated and verified |
| 6 | Increased `leader_timeout_secs` from 15 → 120 on all 4 validators | Slows penalization rate (penalty every 120s instead of 15s) |
| 7 | Increased `status_ready_genesis_grace_secs` from 15 → 60 on all 4 validators | More time for mesh to form before consensus starts |
| 8 | Increased `block_timeout_secs` from 10 → 30 on all 4 validators | More time for block propagation |
| 9 | Set up Val 3 (62.146.182.209) — user installed Control Panel and started it | Val 3 now online and connecting |
| 10 | Deleted `validator_registry.json` on all 4 servers multiple times | Forces fresh registry rebuild from genesis |
| 11 | Performed multiple coordinated restarts of all 4 validators simultaneously | Each time: connections form briefly, then churn loop takes over |
| 12 | Verified clocks are NTP-synchronized across all servers | All within 6 seconds — time sync is NOT the issue |

### Attempted but Failed to Resolve
| # | Action | Why It Failed |
|---|--------|---------------|
| 1 | Coordinated restart with registry reset and lowered thresholds | Blocks 1-3 produced, then penalization cascade deactivated all validators within ~60 seconds |
| 2 | Multiple restarts with progressively tuned timeouts | Bootstrap churn loop prevents connections from stabilizing regardless of timeout values |
| 3 | Increased leader_timeout to 120s to slow cascade | Helps slow the cascade but doesn't fix the bootstrap churn root cause — validators still see "1 live" |

---

## 3. Steps Required to Fully Resolve

### Phase 1: Fix the Bootstrap Churn Loop (CODE CHANGE REQUIRED)

This is the primary blocker. Options (in order of preference):

**Option A: Add config option for bootstrap refresh interval**
Add a `bootstrap_refresh_secs` field to `[p2p]` config section. Set it to 60-120 seconds for testnet. This prevents the 10-second churn cycle.

**Location:** `src/p2p/networking.rs` lines 35-36, 919-927

**Option B: Make duplicate detection skip if handshake is in progress**
When an incoming connection is detected as duplicate, don't drop either connection if the newer one hasn't completed its handshake yet. Wait for handshake completion before deciding.

**Option C: Increase `FAST_BOOTSTRAP_REFRESH_SECS` from 10 to 60**
Quick constant change in `src/p2p/networking.rs:35`. Gives connections 60 seconds to stabilize before the next churn event.

**Option D: Skip re-dial to peers that are already connected**
In the bootstrap thread, before dialing a target, check if a connection to that peer identity already exists. If so, skip the dial. This prevents the duplicate detection from ever firing on established connections.

### Phase 2: Neuter the Penalization System (CODE CHANGE or CONFIG)

**Option A (Preferred): Add a config flag to disable penalization**
Add `penalization_enabled = false` to `[consensus]` config. Guard the `apply_proposer_penalty()` and `apply_leader_timeout_penalty()` calls behind this flag.

**Option B: Remove the `uptime >= 95%` eligibility check for testnet**
Change `is_eligible()` in `src/validator.rs:242` to not require uptime threshold, or lower it significantly (e.g., >= 10%).

**Option C: Increase the jail threshold**
Change `MISSED_VOTE_JAIL_THRESHOLD` from 3 to 50 in `src/validator.rs:11`. This gives validators 50 missed blocks before jailing.

**Option D: No-op the penalty functions**
Comment out the bodies of `apply_proposer_penalty()` and `apply_leader_timeout_penalty()` in `src/consensus/consensus_algorithm.rs` lines 1689 and 1707.

### Phase 3: Fix Validator 4

SSH into 71.182.137.118, verify `node.toml` has `[identity] address` set to its validator address, and verify the launch environment includes `SYNERGY_VALIDATOR_ADDRESS`.

### Phase 4: Fix Duplicate Peer Detection (Long-term)

Make resolution **deterministic on both sides** using a consistent tiebreaker (lexicographic comparison of node IDs). Both sides independently reach the same conclusion about which connection to keep. Neither side treats the other's dedup as a failure.

**Location:** `src/p2p/networking.rs` — `resolve_duplicate_connection()` and the connection handler around line 2130.

### Phase 5: Infrastructure

1. Restart or replace dead seed servers (seed1/2/3.synergynode.xyz:5621)
2. Fix zombie process reaping in Node Control Panel
3. Set up Validator 2 proper SSH access (current `rob` user at .207 is wrong)

---

## 4. Current Server State

| Validator | Address | Host | Status | Config (min_val) | Notes |
|-----------|---------|------|--------|-------------------|-------|
| Val 1 | `synv114cvu472rkdgpmzvkj70zk9tu8cqqlu4x9ra` | 62.146.182.207 | Running, 1 live | 2 | PID 47398 |
| Val 2 | `synv11wrj74dnkc802jfl4e7j7jd2azj2zk2eqvgu` | 62.146.182.208 | Running, 1 live | 2 | PID 51048 |
| Val 3 | `synv11v2r4gnp5py3ae5ft6646lxpqphdv58k8tyu` | 62.146.182.209 | Running, 1 live | 2 | PID 35463 |
| Val 4 | `synv119gw09...` | 71.182.137.118 | Empty address | Unknown | Not accessible |
| Val 5 | `synv11mvlgy72uq7kuh200qnxv67zrqjugz267k46` | 194.163.183.166 | Running, 1 live | 2 | PID 449694 |

All servers: NTP-synced, Europe/Berlin timezone, clocks within 6 seconds of each other.

---

## 5. Config Changes Applied (Current State)

```toml
[consensus]
min_validators = 2              # was 3
status_ready_min_validators = 2 # was 3
validator_vote_threshold = 2    # was 3
leader_timeout_secs = 120       # was 15
block_timeout_secs = 30         # was 10
status_ready_genesis_grace_secs = 60  # was 15
# Unchanged:
allow_genesis_status_bypass = true
block_time_secs = 2
status_ready_gate_enabled = true
max_validators = 5
validator_cluster_size = 5
```

---

## 6. Recommendations

1. **Fastest path to blocks:** Rebuild the binary with `FAST_BOOTSTRAP_REFRESH_SECS = 120` and either remove the uptime eligibility check or set `MISSED_VOTE_JAIL_THRESHOLD = 100`. Redeploy to all 4 servers.

2. **If rebuilding is not immediately possible:** Add a `persistent_peers` mechanism that bypasses bootstrap for known validator addresses (similar to Tendermint's persistent_peers). This would prevent bootstrap-triggered connection churn for known validators.

3. **Long-term:** The P2P layer needs a "connection stability" period where established connections are protected from being disrupted by bootstrap refresh cycles. Consider a "recently connected" grace period of 60+ seconds where duplicate detection prefers keeping existing connections unconditionally.

---

## 7. Files Reference

| File | Key Lines | Role |
|------|-----------|------|
| `src/p2p/networking.rs` | 35-36 | `FAST_BOOTSTRAP_REFRESH_SECS = 10`, `NORMAL_BOOTSTRAP_REFRESH_SECS = 120` |
| `src/p2p/networking.rs` | 919-927 | `current_bootstrap_refresh_interval()` — decides fast vs normal |
| `src/p2p/networking.rs` | 875-897 | `status_ready_validator_participants()` — counts peers with status exchange |
| `src/p2p/networking.rs` | 230-231 | `peer_has_remote_status()` — requires `status_received_at` |
| `src/p2p/networking.rs` | 589 | `resolve_duplicate_connection()` |
| `src/p2p/networking.rs` | 2130-2300 | Handshake handler with duplicate detection |
| `src/consensus/consensus_algorithm.rs` | 469-500 | Live validator check and "Insufficient live" message |
| `src/consensus/consensus_algorithm.rs` | 1689-1701 | `apply_proposer_penalty()` |
| `src/consensus/consensus_algorithm.rs` | 1707-1737 | `apply_leader_timeout_penalty()` |
| `src/validator.rs` | 11-18 | Penalization constants (jail=3, slash=6, uptime penalty=2.5) |
| `src/validator.rs` | 193-201 | `record_missed_vote()` — applies penalties |
| `src/validator.rs` | 242-247 | `is_eligible()` — requires uptime >= 95, synergy >= 50 |
