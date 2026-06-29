**Part IX — Threat Model, Security & Patent-Relevant Effects**

**31. Unified Threat Model (Network + Wallet)**

Synergy Network is designed under the assumption that **every layer of the system will be targeted**. This includes not only consensus and execution infrastructure, but also wallets, governance mechanisms, cross-chain coordination paths, and cryptographic lifecycle transitions.

Unlike many blockchain systems, Synergy does not treat these layers as independent security domains. Historical failures demonstrate that attackers exploit the seams between layers, not the layers themselves.

Accordingly, Synergy adopts a **unified threat model** in which the network, execution environment, interoperability protocols, governance mechanisms, and wallet are treated as a single adversarial surface with **explicitly separated authority domains**.

Security claims are made only within this unified model.

**31.1 Adversary Classes**

Synergy assumes the presence of multiple, overlapping adversary classes. These adversaries may cooperate, escalate, or shift tactics over time.

**Byzantine Infrastructure Participants**

Validators, relayers, governance participants, or other protocol actors may behave arbitrarily, including:

- equivocation,

- selective participation,

- censorship,

- collusion,

- protocol deviation.

Synergy assumes that a bounded fraction of such participants may be adversarial at any given time.

**Economic Adversaries**

Attackers may deploy substantial capital to:

- influence incentives,

- distort governance outcomes,

- manipulate staking distributions,

- fund prolonged denial-of-service campaigns.

Synergy explicitly rejects the assumption that capital concentration implies honesty or alignment.

**Network-Level Adversaries**

Adversaries may:

- delay or reorder messages,

- induce temporary network partitions,

- selectively censor traffic,

- observe all unencrypted communication.

Synergy assumes partial synchrony and designs liveness guarantees accordingly, without assuming reliable or fair message delivery.

**Endpoint and Wallet Adversaries**

User devices and wallet environments are treated as **high-risk endpoints**.

Adversaries may:

- compromise devices,

- exfiltrate key material,

- inject malicious applications,

- exploit user interfaces,

- attempt unauthorized recovery.

Synergy assumes wallet compromise is plausible and designs containment mechanisms so that compromise does not automatically imply loss of identity, authority, or assets.

**Software and Implementation Adversaries**

Attackers may exploit:

- implementation bugs,

- misconfigurations,

- undefined behavior,

- edge-case execution paths.

Synergy mitigates these risks through constrained execution semantics, formal verification, and explicit failure handling, but does not assume perfect implementations.

**Cryptographic Adversaries (Present and Future)**

Synergy assumes:

- present-day classical cryptographic attackers,

- future adversaries with access to large-scale quantum computation.

This includes adversaries capable of harvesting encrypted or signed data today for future decryption or forgery.

**31.2 Assumed Adversarial Capabilities**

Within this threat model, adversaries may:

- control a bounded fraction of validators or relayers,

- acquire large quantities of the native token,

- observe historical blockchain and wallet activity,

- compromise individual devices or operational environments,

- coordinate attacks across chains and domains,

- exploit timing, ordering, and recovery workflows,

- attempt downgrade or migration attacks during cryptographic transitions.

Synergy does **not** assume:

- trusted custodians,

- honest majorities by stake,

- secrecy of algorithms,

- permanent security of cryptographic primitives.

**31.3 Explicit Trust Boundaries**

Synergy defines explicit trust boundaries and refuses to blur them.

- **No trust is placed in wallets as signing oracles**

Wallets are constrained authority governors, not trusted executors.

- **No trust is placed in relayers as truth sources**

Relayers provide availability, not authority.

- **No trust is placed in governance as an override mechanism**

Governance is bounded and cryptographically enforced.

- **No trust is placed in bridges or custodial intermediaries**

Cross-chain interaction is attestation-based and non-custodial.

Where trust is required, it is:

- minimized,

- explicitly scoped,

- cryptographically enforced,

- auditable after the fact.

**31.4 Failure as a First-Class Condition**

Synergy assumes that **failure will occur**.

Failures may include:

- validator outages,

- relayer unavailability,

- network partitions,

- wallet compromise,

- governance deadlock,

- cryptographic deprecation.

The system is designed so that failure:

- degrades toward non-execution,

- preserves identity continuity,

- prevents silent authority escalation,

- contains damage to the smallest possible scope.

This philosophy is summarized as:

*It is safer to do nothing than to do the wrong thing.*

**31.5 Unified Authority Perspective**

Within the unified threat model, authority is never implicit and never singular.

Authority in Synergy is:

- identity-anchored (UMA),

- cryptographically enforced (Aegis),

- plane-separated (network, wallet, execution),

- policy-gated,

- generation-scoped,

- auditable.

An adversary must simultaneously compromise **multiple independent authority domains** to produce catastrophic failure. Single-layer compromise is explicitly insufficient.

**32. Explicitly Mitigated vs. Out-of-Scope Threats**

Security claims in Synergy Network are made under explicit assumptions and bounded adversarial conditions. This section enumerates the threats that Synergy is architected to mitigate and distinguishes them from those that are intentionally left out of scope.

This distinction is not a weakness. It is a requirement for correctness, auditability, and legal defensibility.

**32.1 Explicitly Mitigated Threats**

Synergy Network is explicitly designed to mitigate the following classes of threats within the bounds defined in Section 31.

**32.1.1 Validator and Relayer Byzantine Behavior**

Synergy mitigates byzantine behavior among validators and relayers up to defined thresholds.

Mitigations include:

- Proof-of-Synergy cooperative consensus with bounded fault tolerance,

- Synergy Score–weighted participation to resist cartel formation,

- threshold cryptographic authorization for cross-chain attestations,

- cryptographic attribution and slashing for equivocation or omission.

No single validator or relayer, and no minority coalition, can unilaterally fabricate authority or induce incorrect execution.

**32.1.2 Custodial and Bridge-Based Failure Modes**

Synergy explicitly eliminates custodial bridge risk.

Mitigations include:

- non-custodial, attestation-based interoperability (SXCP),

- deterministic verification of finalized external state,

- absence of wrapped assets or pooled escrow contracts,

- local execution conditioned on verified facts only.

This removes entire classes of exploits that have historically resulted in catastrophic losses.

**32.1.3 Wallet Compromise and Endpoint Attacks**

Synergy mitigates the impact of wallet compromise without assuming endpoint security.

Mitigations include:

- plane separation within the wallet (identity, routing, execution),

- policy-gated authorization enforced pre-signature,

- generation-scoped authority with cryptographic deprecation,

- recovery workflows that restore authority without resurrecting secrets.

Compromise of a device or signing environment does not automatically imply loss of identity or irreversible asset theft.

**32.1.4 Governance Capture via Capital Concentration**

Synergy mitigates governance capture driven purely by economic power.

Mitigations include:

- Synergy Score–weighted governance rather than token-plutocratic voting,

- explicit authority domains and proposal typing,

- cryptographic enforcement of governance outcomes,

- time-bounded and auditable emergency powers.

Capital alone cannot override protocol invariants.

**32.1.5 Cross-Chain Replay, Reorganization, and Timing Attacks**

Synergy mitigates common cross-chain attack vectors.

Mitigations include:

- chain-specific finality enforcement in SXCP,

- deterministic attestation verification,

- replay protection through context binding,

- failure resolution toward non-execution under ambiguity.

Execution cannot be induced through timing manipulation or partial finality exploitation.

**32.1.6 Harvest-Now–Decrypt-Later Cryptographic Attacks**

Synergy mitigates long-horizon cryptographic risk.

Mitigations include:

- post-quantum cryptography embedded at the protocol level (Aegis),

- cryptographic agility with governance-controlled migration,

- hybrid verification during transition periods,

- wallet-level post-quantum authority enforcement.

Historical data capture does not yield future authority compromise.

**32.2 Intentionally Out-of-Scope Threats**

Synergy does not attempt to solve all security problems. The following threats are explicitly out of scope.

**32.2.1 Total Endpoint Compromise with Persistent Control**

If an adversary maintains uninterrupted, undetected control over all recovery guardians, all policy enforcement mechanisms, and all execution environments for an identity, Synergy cannot prevent authority misuse.

The system is designed to detect, limit, and recover from compromise—not to function under absolute endpoint domination.

**32.2.2 Global Network Suppression**

Synergy does not claim liveness under total or indefinite global network censorship.

Safety is preserved under such conditions; availability is not guaranteed.

**32.2.3 Cryptographic Breaks Beyond Assumed Bounds**

If all deployed cryptographic primitives—classical and post-quantum—are simultaneously broken beyond current threat models, Synergy cannot guarantee security.

The system assumes cryptographic hardness consistent with contemporary and near-future research.

**32.2.4 Social Engineering Beyond Policy Constraints**

Synergy mitigates technical enforcement failures, not human judgment failures.

If a user authorizes an action that complies with all enforced policy and identity constraints, the protocol treats that authorization as valid, regardless of social manipulation.

**32.2.5 Malicious Governance Supermajorities**

If governance thresholds are legitimately met under Synergy Score–weighted rules, and the resulting actions remain within authorized domains, the protocol will execute them—even if those actions are strategically harmful.

This is a fundamental property of decentralized governance, not a defect.

**32.3 Why Explicit Exclusions Matter**

Explicitly stating what is out of scope:

- prevents false security assumptions,

- strengthens auditor and regulator trust,

- improves incident response clarity,

- protects patent claims from overbreadth challenges,

- reinforces the integrity of mitigated guarantees.

Synergy prefers **narrow, enforceable claims** over vague assurances.

**33. Failure Containment Across Layers**

Synergy Network is not designed to be failure-free. It is designed to be **failure-bounded**.

Traditional blockchain systems tend to optimize for best-case execution and treat failure as an anomaly. Synergy instead treats failure as a **first-class condition** and designs every layer to ensure that when failure occurs, it is contained, attributable, and non-escalating.

Failure containment in Synergy is achieved through **explicit authority separation across layers**, reinforced by cryptographic verification, deterministic execution, and policy gating.

**33.1 Layered Authority Containment**

Each major subsystem in Synergy—consensus, cryptography, interoperability, execution, governance, and wallet—operates within a **bounded authority domain**.

Failure in one domain does not imply authority leakage into another.

Examples include:

- validator misbehavior does not grant execution authority,

- wallet compromise does not grant identity replacement,

- relayer failure does not grant state fabrication,

- governance deadlock does not enable unilateral override.

This layered containment ensures that attackers must compromise **multiple orthogonal systems** to achieve catastrophic impact.

**33.2 Containment Under Partial Compromise**

Synergy explicitly anticipates **partial compromise scenarios**, including:

- some validators behaving maliciously,

- some relayers withholding attestations,

- a wallet device being compromised,

- governance temporarily unable to reach consensus.

Under these conditions:

- execution resolves toward non-execution,

- authority revocation takes precedence over continuation,

- recovery workflows activate without resurrecting obsolete authority.

Partial compromise degrades availability, not correctness.

**33.3 Deterministic Failure Resolution**

When verification conditions cannot be satisfied deterministically—due to ambiguity, delay, or conflicting evidence—Synergy enforces a **single resolution path: halt**.

This principle applies across layers:

- SXCP halts cross-chain execution when finality is uncertain,

- SynQ halts execution when authorization is incomplete,

- wallets halt signing when policy or identity state is invalid,

- governance halts execution when thresholds are not met.

This deterministic resolution eliminates gray zones where attackers exploit timing, interpretation, or discretion.

**33.4 Recovery as Containment, Not Reset**

Recovery in Synergy is not a rollback to a previous state. It is a **controlled forward transition**.

Recovery mechanisms:

- restore bounded authority,

- preserve historical attribution,

- maintain cryptographic evolution constraints,

- prevent resurrection of deprecated keys or permissions.

This ensures that recovery improves security posture rather than reverting the system to a more vulnerable configuration.

**33.5 Cross-Layer Failure Propagation Limits**

Synergy explicitly limits how failure signals propagate across layers.

For example:

- wallet recovery does not trigger protocol-level rollback,

- cross-chain attestation failure does not affect local consensus,

- governance suspension does not compromise execution determinism.

Each layer observes failures in other layers only through **explicit, verifiable signals**, not implicit assumptions.

**34. Patent-Relevant Technical Effects**

This section identifies the **technical effects** produced by Synergy Network’s architecture that arise from the interaction of its components, rather than from any single mechanism in isolation.

These effects are emergent, repeatable, and enforceable under the unified threat model defined in this document. They are not business methods or abstract goals; they are **concrete system-level behaviors**.

Descriptions are intentionally non-enabling and avoid claim-critical parameters.

**34.1 Identity Without Execution**

Synergy produces the technical effect of **identity persistence without execution authority**.

Through UMA identity anchoring, plane-separated wallets, and protocol-enforced authorization:

- identity remains stable across key rotation and recovery,

- possession of execution capability does not imply identity control,

- execution authority can be revoked or constrained without replacing identity.

This breaks the historical equivalence between keys and identity and enables long-lived, auditable authority structures.

**34.2 Recovery Without Key Resurrection**

Synergy produces the technical effect of **recovery without resurrecting cryptographic material**.

Recovery workflows:

- establish new authority epochs,

- cryptographically deprecate prior generations,

- preserve historical attribution and audit trails.

This eliminates a pervasive failure mode in which old backups silently regain power and enables secure recovery under post-quantum migration.

**34.3 Non-Custodial Cross-Chain Coordination**

Synergy produces the technical effect of **cross-chain coordination without custody or discretionary intermediaries**.

Through deterministic attestation (SXCP), threshold authorization, and local execution:

- assets never leave their native chains,

- execution is conditioned on verified external facts,

- relayers provide availability, not authority.

This removes entire attack classes associated with bridges, wrapped assets, and pooled escrow.

**34.4 Post-Quantum Security Without Compatibility Breakage**

Synergy produces the technical effect of **post-quantum cryptographic migration without identity fracture or execution incompatibility**.

By embedding cryptographic agility at the protocol and wallet levels:

- cryptographic primitives may evolve,

- identity semantics remain stable,

- contracts and governance remain valid,

- hybrid verification enables safe transition.

This effect directly addresses long-horizon cryptographic risk while preserving operational continuity.

---

## Section 35 — DAG Sublayer Threat Model

> **Applicability:** This section extends the Synergy Network Security Specification v1.3 to cover the Cluster-Local Certified DAG sublayer. All mitigations described here are mandatory for any testnet or mainnet deployment that enables `dag.enabled = true`. Cross-references to the base specification are noted inline.

The DAG sublayer operates exclusively within each validator cluster boundary. Its threat surface is distinct from the network-wide PoSy threat surface but intersects with it at the cluster-to-QC handoff point. Three DAG-specific attack categories are formally classified below.

---

### 35.1 DAG-1 — Spam, Replay, and Stale-Batch Injection

**Classification:** MEDIUM (escalates to HIGH under sustained load)
**Scope:** Cluster-local DAG ingestion pipeline

**Attack Description:**

A Byzantine or compromised cluster member submits DagBatch objects at excessive rate, submits previously-seen batch hashes (replay), or submits batches referencing stale or non-existent parent vertices. The goal is to exhaust cluster processing bandwidth, pollute the availability layer, or trigger incorrect causal ordering.

**Attack Preconditions:**

- Attacker controls at least one cluster member (validator slot)
- DAG ingestion does not enforce per-sender rate limits or batch-hash deduplication

**Potential Impact:**

- Legitimate transaction batches delayed or dropped
- Availability threshold artificially depressed
- Denial-of-service at cluster level without triggering network-wide slashing

**Mitigations (mandatory per Security Spec v1.3, §DAG-1):**

1. **Batch deduplication index** — every node maintains a bloom filter plus exact-match index of seen batch hashes per epoch; duplicate submissions are silently dropped and logged
2. **Per-sender rate cap** — enforced via `dag.spam_filter_rate_limit_per_validator` (default: 100 batches/epoch); excess batches rejected with `DAG_RATE_EXCEEDED` error code
3. **Stale-parent rejection** — batches referencing parent hashes absent from the local DAG store and older than `dag.max_causal_depth` rounds are rejected
4. **ML-DSA-65 sender authentication** — every DagBatch carries a post-quantum signature from the submitting validator; unsigned or mis-signed batches are rejected before touching the DAG store
5. **Evidence chain** — sustained spam above threshold triggers `dag_replay_or_stale_spam` slashing evidence; evidence is broadcast cluster-internally and submitted to the PoSy evidence pool at the next QC boundary

**Slashing Condition:** `dag_replay_or_stale_spam` — triggered when a validator submits ≥ 3× `dag.spam_filter_rate_limit_per_validator` batches within a single epoch or submits any batch with a hash seen in the same or prior epoch.

**Configuration Parameters:**

```toml
[dag]
spam_filter_rate_limit_per_validator = 100   # batches per epoch
max_causal_depth                     = 20    # rounds; older parent refs rejected
```

---

### 35.2 DAG-2 — Data Withholding

**Classification:** HIGH
**Scope:** DagCertificate availability layer

**Attack Description:**

A Byzantine cluster member signs availability receipts (AvailabilityReceipt) for DagBatch objects it never actually stores or makes available to peer nodes. This allows the attacker to cause a DagCertificate to be issued (2f+1 receipts obtained) for a batch whose data is unavailable to honest nodes, breaking the availability-before-ordering guarantee and potentially allowing data to be suppressed from the final ordered output.

**Attack Preconditions:**

- Attacker controls f or more cluster members (at n=7, f=2 → attacker needs ≥ 2 nodes)
- No data-availability sampling or retrieval verification is applied before receipt acceptance

**Potential Impact:**

- Transactions included in an OrderingCut that honest nodes cannot reconstruct
- Post-QC block production failures as validators attempt to decode encrypted batches whose data is unavailable
- Silent data loss without immediate slashing evidence if not caught

**Mitigations (mandatory per Security Spec v1.3, §DAG-2):**

1. **Pull-before-ack protocol** — a node must successfully retrieve and hash-verify the full encrypted DagBatch content before issuing an AvailabilityReceipt; receipts issued without data retrieval are cryptographically indistinguishable but are caught by cross-validation
2. **Cross-node availability audit** — upon DagCertificate issuance, two randomly-selected honest nodes (selected via EntropyBeacon round) independently pull and verify the batch; failure generates `dag_data_withholding` evidence
3. **Threshold gating** — `dag.availability_threshold` requires 2f+1 receipts (default `5` at n=7); an attacker controlling f nodes cannot single-handedly certificate a withheld batch
4. **Erasure-coded batch storage** (Phase 3 target) — batches are erasure-coded across cluster members; any 2f+1 members can reconstruct the full batch, eliminating single-node withholding as an effective strategy
5. **Decryption-key release gating** — ML-KEM-768 decryption keys are not released until post-QC finalization; if a batch is unavailable at decryption time, the epoch's key release is halted and slashing evidence is submitted

**Slashing Condition:** `dag_data_withholding` — triggered when a node issued an AvailabilityReceipt for a batch it cannot serve on demand during the cross-node audit window (`dag.availability_audit_window`, default: 60 seconds after certificate issuance).

**Configuration Parameters:**

```toml
[dag]
availability_threshold      = 5     # 2f+1 at n=7; adjust for other cluster sizes
availability_audit_window   = 60    # seconds; cross-node pull window post-certificate
```

---

### 35.3 DAG-3 — Equivocation (Double-Vertex Attack)

**Classification:** CRITICAL
**Scope:** DAG vertex construction and certificate issuance

**Attack Description:**

A Byzantine cluster member broadcasts two or more distinct DagVertex objects for the same `(author, round)` pair — identical round number, same author, different content (different parent hashes, different batch references, or different signatures). This forks the local DAG, causing honest nodes to build on incompatible branches and undermining deterministic ordering. If the fork is not detected before an OrderingCut, conflicting transactions may appear in the ordered output.

This is the DAG-layer analogue of double-voting in BFT consensus, and is treated with equivalent severity.

**Attack Preconditions:**

- Attacker controls at least one cluster member
- No per-(author, round) uniqueness check is enforced before vertex acceptance

**Potential Impact:**

- Two incompatible DAG forks within a cluster
- OrderingCut produced from a forked DAG, leading to non-deterministic block assembly
- Potential for double-spend at the pre-ordering layer if equivocated vertices contain conflicting encrypted transactions

**Mitigations (mandatory per Security Spec v1.3, §DAG-3):**

1. **Per-(author, round) lock** — each node maintains a write-once map: once a vertex is accepted for `(author, round)`, any subsequent vertex for the same pair is rejected and triggers immediate evidence generation; no second message accepted regardless of content
2. **Evidence broadcast** — the detecting node broadcasts a `DagConflictEvidence` object containing both vertex hashes, both ML-DSA-65 signatures, and the detecting node's own identity; all cluster members receiving the evidence halt processing of both conflicting vertices
3. **Automatic slashing submission** — `dag_equivocation_same_round` evidence is submitted to the PoSy evidence pool at the next QC boundary; slashing is unconditional and does not require an additional vote
4. **Propagation blacklist** — the equivocating node's subsequent messages are quarantined for the remainder of the epoch; the cluster continues with n−1 effective participants
5. **Cross-cluster notification** — at the next validator-set epoch boundary, the slashing record is broadcast network-wide to update validator reputation scores

**Slashing Condition:** `dag_equivocation_same_round` — triggered immediately upon detection of two distinct valid DagVertex objects from the same author for the same round, regardless of which arrived first. This slash is unconditional; no quorum vote is required.

**Configuration Parameters:**

```toml
[dag]
# Equivocation detection is always active when dag.enabled = true
# No configurable parameters; detection and slashing are protocol-mandatory
```

---

### 35.4 Supporting DAG Threat Categories

The following two threat categories are addressed in the DAG integration documents but summarized here for completeness. Full details appear in *Synergy DAG Validator Cluster Integration — Part 2, Section 14 (Threat Matrix).*

**DAG-4 — Hidden Sub-DAG / Censorship Pattern (HIGH)**

A cluster majority withholds specific transaction batches from the DAG to selectively censor transactions. Mitigation: `dag_censorship_pattern` slashing triggered when a validator's batch inclusion rate falls below `dag.min_batch_inclusion_rate` for two consecutive epochs. Enforced by cluster-local inclusion monitoring and cross-cluster audit at epoch boundaries.

**DAG-5 — MEV Leakage via Ordering Manipulation (HIGH)**

A cluster member with foreknowledge of encrypted batch contents (or colluding with an encryptor) manipulates causal references to influence transaction ordering for profit. Mitigation: encrypted batch invariant (all batches ML-KEM-768 encrypted; decryption keys released only post-QC finalization by PoSy epoch key ceremony); ordering deterministic from on-wire artifacts; `dag_certificate_fraud` slashing for provable ordering manipulation.

---

### 35.5 Encrypted Batch Invariant (Security Spec v1.3 §4.1-DAG)

The encrypted batch invariant is the primary MEV and front-running defense within the DAG sublayer. It is a **protocol-level hard constraint**, not a configuration option.

**Invariant Definition:**

> All transaction data carried in DagBatch objects MUST be encrypted under ML-KEM-768 at the time of batch creation. The encryption key for each batch is committed on-chain as part of the batch header. Decryption keys are released by the epoch's PoSy QC finalization ceremony and MUST NOT be made available to any cluster member prior to QC finalization of the epoch in which the batch is ordered.

**Enforcement Mechanism:**

- `dag.require_encrypted_batches = true` is a hard default; setting it to `false` is a testnet-only override and MUST be blocked in mainnet configuration
- Any DagBatch submitted with a plaintext payload field is rejected with `DAG_UNENCRYPTED_BATCH` error
- The encryption key commitment is included in the batch header hash; a batch whose header does not include a valid KEM commitment is treated as malformed
- Post-QC key release is part of the PoSy epoch transition protocol; keys released early constitute `dag_certificate_fraud` evidence

**Cross-Reference:** Security Specification v1.3, §4.1 (Encrypted Mempool), §7.3 (ML-KEM-768 Usage), §12.2 (Epoch Key Ceremony)

---

### 35.6 DAG Slashing Summary

| Slashing Type | Severity | Trigger Condition | Evidence Required |
|---|---|---|---|
| `dag_equivocation_same_round` | CRITICAL | Two valid vertices, same (author, round) | Both vertex hashes + signatures |
| `dag_certificate_fraud` | CRITICAL | Certificate issued without 2f+1 valid receipts, or ordering precedes certificate | Certificate + receipt audit |
| `dag_data_withholding` | HIGH | AvailabilityReceipt issued; data unavailable during audit window | Audit failure log + receipt |
| `dag_censorship_pattern` | HIGH | Batch inclusion rate below minimum for 2 consecutive epochs | Inclusion rate telemetry |
| `dag_replay_or_stale_spam` | MEDIUM | Duplicate batch hash or stale parent reference submitted | Batch hash index + timestamps |

All DAG slashing evidence is submitted to the PoSy evidence pool and processed identically to existing validator slashing evidence. DAG slashing does not bypass the existing evidence adjudication process.

---

### 35.7 Security Specification v1.3 Alignment

The DAG threat model above is fully derived from and aligned with Synergy Network Security Specification v1.3. The following table maps each DAG threat to the relevant specification sections.

| DAG Threat | Spec Section | Requirement |
|---|---|---|
| DAG-1 Spam/Replay | §3.4 (Rate Limiting), §8.1 (Evidence Chain) | Mandatory rate caps + deduplication |
| DAG-2 Data Withholding | §4.2 (Availability Guarantees), §8.3 (Cross-Node Audit) | Pull-before-ack + audit window |
| DAG-3 Equivocation | §5.1 (Byzantine Fault Tolerance), §8.2 (Slashing) | Immediate detection + unconditional slash |
| DAG-4 Censorship | §6.1 (Inclusion Fairness), §8.2 (Slashing) | Inclusion rate monitoring |
| DAG-5 MEV Leakage | §4.1 (Encrypted Mempool), §7.3 (ML-KEM-768) | Encrypted batch invariant (hard constraint) |
| Encrypted Batch Invariant | §4.1, §7.3, §12.2 | Protocol-mandatory; no mainnet override |

**Note:** The CRQC (Cryptographically Relevant Quantum Computer) threat classification from Security Spec v1.3 applies in full to all DAG cryptographic operations. ML-DSA-65/87 for vertex signatures, ML-KEM-768 for batch encryption, and Blake3 for all DAG hashing are non-negotiable; classical signature schemes (Ed25519, ECDSA) are explicitly prohibited in DAG object construction.
