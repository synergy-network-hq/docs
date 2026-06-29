**Part II — Synergy Network Architecture Overview**

**3. System-Level Architecture**

Synergy Network is architected as a **composed distributed system**, not a monolithic blockchain with auxiliary features. Its design reflects the observation that consensus, cryptography, interoperability, and execution impose fundamentally different correctness requirements and failure modes. Collapsing these concerns into a single layer introduces circular dependencies that undermine safety, liveness, and upgradeability.

To avoid these pathologies, Synergy separates responsibilities across **three architectural planes**, each governed by explicit invariants and bound together through cryptographic commitments rather than implicit trust.

This architecture is not an abstraction convenience; it is a structural requirement for achieving cooperative consensus, deterministic cross-chain verification, and post-quantum security simultaneously.

> **3.1 Architectural Decomposition and Rationale**
>
> The Synergy Network architecture is decomposed into:

1.  the **Consensus Plane**,

2.  the **Cryptographic Plane**, and

3.  the **Interoperability & Execution Plane**.

> Each plane is defined not only by what it does, but by **what it is forbidden from doing**.
>
> This decomposition exists to enforce four global invariants:

- Consensus liveness must not depend on external chains or application logic.

- Cryptographic authority must not be fragmented or duplicated.

- Cross-chain verification must not influence ordering or finality.

- Application execution must not introduce new trust assumptions.

> Any architecture that violates these invariants cannot simultaneously satisfy Synergy’s design goals.
>
> **3.2 Consensus Plane — Proof-of-Synergy as the Root of Authority**
>
> The **Consensus Plane** establishes the canonical ordering of state transitions and the sole source of finality within the Synergy Network. It is implemented through **Proof-of-Synergy (PoSy)**, a cooperative Byzantine Fault Tolerant protocol operating under partial synchrony assumptions.
>
> This plane is responsible for:

- defining the canonical ledger state,

- finalizing governance decisions,

- enforcing validator participation rules,

- sealing epoch boundaries and checkpoints.

> The Consensus Plane is intentionally **agnostic to application semantics and external blockchain state**. It neither interprets cross-chain events nor executes smart-contract logic. Its sole concern is agreement on *what happened* and *when it became final*.
>
> This isolation is critical. If consensus were allowed to depend on cross-chain verification or contract execution, then failures or delays in those subsystems could stall or destabilize the network. PoSy explicitly forbids such dependencies.
>
> Validator influence within the Consensus Plane is governed by **cooperative consensus weighting**, which incorporates stake commitment, historical contribution, reliability, and governance participation. This weighting affects *who participates* in consensus, but never *how correctness is determined*.
>
> Finality produced by this plane is deterministic and irreversible except through explicitly defined, governance-constrained emergency procedures.
>
> **3.3 Cryptographic Plane — Aegis as the Sole Source of Trust**
>
> The **Cryptographic Plane** is embodied by **Aegis**, the post-quantum cryptographic core of the Synergy Network. This plane provides a **single, protocol-level source of cryptographic authority** for all other planes.
>
> Its responsibilities include:

- identity binding for validators, relayers, users, and contracts,

- authentication of consensus messages and state transitions,

- threshold authorization for cross-chain attestations,

- distributed randomness and entropy generation,

- enforcement of cryptographic key lifecycles.

> No component of Synergy introduces independent cryptographic assumptions outside this plane. There are no alternative signature systems, external key authorities, or ad hoc cryptographic shortcuts elsewhere in the protocol.
>
> This unification is essential for two reasons.
>
> First, it ensures that the security model of the network is coherent. All correctness guarantees reduce to a single cryptographic foundation, rather than a patchwork of incompatible assumptions.
>
> Second, it enables **cryptographic agility**. Because cryptography is centralized at the plane level rather than embedded inconsistently across subsystems, the protocol can evolve its cryptographic foundations without architectural fracture.
>
> The Cryptographic Plane does not determine state, ordering, or application outcomes. It provides *evidence*, not *authority*. Authority remains with the Consensus Plane.
>
> **3.4 Interoperability & Execution Plane — Asynchronous, Deterministic, and Subordinate**
>
> The **Interoperability & Execution Plane** encompasses both the **Synergy Cross-Chain Protocol (SXCP)** and the **SynQ smart-contract execution environment**. This plane is responsible for interpreting finalized state and cryptographic evidence, never for producing finality itself.
>
> SXCP enables deterministic cross-chain attestation by validating external blockchain state through threshold post-quantum authorization. These attestations are **asynchronous** by design. They may lag consensus, arrive out of order, or fail entirely without affecting the safety or liveness of the base chain.
>
> SynQ executes application logic in response to finalized state and verified attestations. Smart contracts may consume cross-chain proofs, invoke cryptographic primitives, and produce application-level state transitions, but they cannot influence block ordering, validator selection, or finality decisions.
>
> This strict subordination ensures:

- cross-chain failures cannot halt the network,

- application bugs cannot corrupt consensus,

- external chain reorganization cannot retroactively affect finalized state.

> The Interoperability & Execution Plane is therefore powerful but contained. It extends the network’s reach without expanding its trust surface.
>
> **3.5 Composition Without Circular Dependency**
>
> The defining property of Synergy’s architecture is **composition without circular dependency**.

- The Consensus Plane finalizes state independently.

- The Cryptographic Plane authenticates and binds that state.

- The Interoperability & Execution Plane reacts to finalized, authenticated state.

> Information flows **downward**, never upward. No plane relies on outputs from a plane that depends on it. This eliminates feedback loops that have historically plagued blockchain designs, particularly in systems attempting deep interoperability.
>
> As a result, Synergy can support cooperative consensus, post-quantum security, and deterministic cross-chain operation simultaneously—without sacrificing correctness or upgradability.
>
> **3.6 Failure Containment and Systemic Resilience**
>
> By isolating responsibilities across planes, Synergy enforces strict failure containment:

- Consensus faults are bounded by Byzantine assumptions.

- Cryptographic faults are bounded by epoch-scoped key lifecycles.

- Interoperability faults are bounded by asynchronous isolation.

- Governance faults are bounded by procedural constraints.

> No single failure mode cascades into total system compromise.

This containment is not incidental; it is the primary architectural objective. The architectural plane separation described above is enforced not only at the network level, but across all protocol-adjacent components, including wallet implementations, routing mechanisms, and recovery systems. Any component that introduces identity authority, routing intent, or execution capability is required to respect the same plane boundaries to preserve systemic correctness.

> **3.7 Architectural Invariants**
>
> The Synergy Network architecture is governed by a set of **explicit architectural invariants**. These invariants define conditions that must always hold for the protocol to be considered correct. They are not implementation details; they are structural constraints that shape every subsystem and upgrade.
>
> Violation of any invariant constitutes an architectural failure, regardless of performance gains or feature completeness.

**Architectural Invariants Table**

| Invariant ID | Invariant Statement | Rationale | Enforced By |
|:--:|----|----|----|
| A-1 | Consensus finality must not depend on external blockchain state | Prevents cross-chain latency, reorgs, or failures from stalling the base chain | PoSy isolation, asynchronous SXCP |
| A-2 | No subsystem may introduce independent cryptographic authority | Prevents fragmented security assumptions and downgrade attacks | Aegis as sole cryptographic plane |
| A-3 | Cross-chain verification must never influence block ordering | Prevents external manipulation of consensus | One-way data flow, finalized-state consumption |
| A-4 | Application execution must not affect consensus safety or liveness | Prevents smart-contract bugs from becoming protocol failures | Execution plane subordination |
| A-5 | Validator influence must be multi-factor and behavior-sensitive | Prevents plutocracy and cartel entrenchment | Cooperative consensus weighting |
| A-6 | Cryptographic compromise must be temporally bounded | Limits blast radius of key exposure | Epoch-scoped key lifecycles |
| A-7 | Governance authority must be procedurally constrained | Prevents discretionary or ad hoc control | DAO rules, time-locks, auditability |
| A-8 | Failures must be containable within a single plane | Prevents cascading systemic collapse | Plane isolation and explicit interfaces |
| A-9 | Deterministic verification must be possible from public data | Enables independent audit and institutional trust | Deterministic attestations, public logs |
| A-10 | Architectural upgrades must not require trust in benevolent actors | Ensures long-term protocol neutrality | Formal processes, cryptographic enforcement |

These invariants collectively ensure that Synergy Network remains coherent, auditable, and resilient as it evolves. They are referenced implicitly throughout subsequent sections and explicitly when evaluating protocol changes.

With the system architecture established, the next section introduces the **Universal Meta-Address (UMA)**—the identity abstraction that allows this architecture to present a coherent interface to users, contracts, and external chains without violating any of the invariants defined above.

**4. Universal Meta-Address (UMA)**

> **4.1 Identity as a Protocol Problem**
>
> In most blockchain systems, identity is treated as an incidental artifact of cryptography: an address is merely the hash or encoding of a public key, and meaning is inferred contextually by the chain on which it appears. This model suffices only in single-chain environments where identity, authority, and execution are co-located.
>
> Synergy Network violates this assumption by design.
>
> Synergy requires:

- cooperative consensus involving long-lived validator identities,

- deterministic cross-chain attestation spanning heterogeneous networks,

- post-quantum key lifecycles with enforced rotation,

- governance actions that persist across epochs and cryptographic transitions.

> In such a system, identity cannot be an emergent property of chain-specific address formats. It must be a **first-class protocol primitive**, explicitly modeled, temporally bound, and cryptographically governed.
>
> UMA exists to satisfy this requirement.
>
> **4.2 Definition and Scope of UMA**
>
> A **Universal Meta-Address (UMA)** is the canonical, chain-agnostic identity reference used throughout the Synergy Network protocol.
>
> UMA is not:

- a wallet address,

- a human-readable alias,

- a naming service,

- a decentralized identifier overlay.

> UMA **precedes** all of these concepts.
>
> It is the identity object against which:

- validator authority is measured,

- governance rights are assigned,

- cryptographic keys are rotated,

- cross-chain attestations are bound,

- smart-contract authorization is evaluated.

> Every identity-bearing actor within Synergy—validators, relayers, users, contracts, governance entities—possesses exactly one UMA at any given epoch.
>
> **4.3 UMA and Temporal Identity Consistency**
>
> A core requirement of Synergy Network is **temporal consistency of identity**.
>
> Without UMA, identity meaning drifts over time due to:

- key rotation,

- cryptographic upgrades,

- address format changes,

- cross-chain encoding differences.

> UMA decouples **identity continuity** from **cryptographic material**.
>
> The UMA remains stable across:

- epochs,

- key rotations,

- cryptographic transitions,

- external chain interactions.

> Cryptographic keys are **attributes of a UMA**, not the identity itself. When keys rotate, the UMA persists. When cryptographic schemes evolve, the UMA persists. When an actor interacts across chains, the UMA persists.
>
> This is not a convenience feature — it is required to prevent identity fragmentation and replay vulnerabilities across time.
>
> **4.4 UMA as the Anchor for the Cryptographic Plane**
>
> All UMA identities are anchored in the **post-quantum cryptographic core (Aegis)**.
>
> Aegis is responsible for:

- generating cryptographic material associated with a UMA,

- authenticating actions performed on behalf of a UMA,

- enforcing rotation, revocation, and lifecycle boundaries,

- binding cryptographic evidence to UMA references.

> Critically:

- No cryptographic key has standing in the protocol unless it is bound to a UMA.

- No UMA may assert authority using cryptographic material not managed by Aegis.

> This enforces a single source of cryptographic truth and prevents identity spoofing, downgrade attacks, and key-authority ambiguity.
>
> **4.5 UMA and Deterministic Cross-Chain Attestation**
>
> Deterministic cross-chain attestation is **impossible** without a chain-agnostic identity layer.
>
> External blockchains encode identity differently:

- different address formats,

- different signature semantics,

- different account models.

> SXCP does not reason about these identities directly. Instead, it reasons about **UMA-referenced identities** and validates external state *as evidence bound to a UMA*.
>
> This ensures:

- attestations remain invariant across chains,

- identity meaning does not change with encoding context,

- replay attacks using alternate address formats are structurally prevented.

> In this sense, UMA is not merely compatible with SXCP — **SXCP presupposes UMA**.
>
> Within the Synergy Wallet, UMA functions as the sole identity anchor for routing, authorization, and recovery decisions. Wallet components may consume UMA-referenced authority and produce intent or execution requests, but may not redefine, duplicate, or bypass UMA semantics. This ensures that wallet-level operations remain consistent with network-level identity guarantees.
>
> **4.6 UMA in Cooperative Consensus and Governance**
>
> Proof-of-Synergy relies on long-lived, behavior-sensitive identity.
>
> Validator influence is accumulated, decayed, and evaluated across epochs. Governance rights are exercised over time. Reputation and contribution histories must persist beyond cryptographic material.
>
> UMA provides the identity continuity required for:

- Synergy Score accumulation,

- validator accountability,

- governance participation and auditing,

- slashing and remediation processes.

> Without UMA, consensus weighting would reset under key rotation or cryptographic transition, enabling reputation laundering and governance abuse.
>
> **4.7 UMA in Smart-Contract Authorization**
>
> Within the SynQ execution environment, authorization is evaluated against UMA identities, not raw addresses.
>
> This allows contracts to:

- reason about identity consistently across chains,

- remain invariant under cryptographic upgrades,

- avoid embedding chain-specific address logic,

- verify cross-chain actions without translation ambiguity.

> UMA thus enables **contract-level determinism** in a multi-chain environment.
>
> **4.8 UMA Failure Modes and Containment**
>
> UMA is designed so that identity-related failures are **non-catastrophic**.

- Key compromise does not destroy identity.

- Identity revocation does not rewrite history.

- Resolution failures do not affect consensus finality.

- Governance abuse is procedurally constrained.

> UMA participates in authority, but does not define finality. Consensus safety remains isolated.
>
> **4.9 UMA Architectural Invariants**
>
> The Universal Meta-Address (UMA) subsystem is governed by the following architectural invariants. These invariants are **co-equal** with the system-wide architectural invariants and constrain all present and future implementations of UMA-related logic.
>
> Violation of any UMA invariant constitutes a **protocol-level failure**, regardless of system liveness or partial correctness elsewhere.

**  **

**Universal Meta-Address (UMA) Invariants Table**

| Invariant ID | Invariant Statement | Rationale | Enforced By |
|:--:|----|----|----|
| U-1 | Every authority-bearing actor must be associated with exactly one UMA per epoch | Prevents identity duplication, aliasing attacks, and authority ambiguity | Protocol-level identity registry |
| U-2 | UMA identity must persist independently of cryptographic key material | Prevents identity loss during key rotation or cryptographic migration | Aegis key lifecycle binding |
| U-3 | No cryptographic key or signature has protocol standing unless bound to a UMA | Eliminates unmanaged cryptographic authority | Aegis-only authentication paths |
| U-4 | UMA must be the sole identity reference used by consensus, governance, and execution logic | Prevents identity drift across subsystems | Canonical identity resolution |
| U-5 | Cross-chain attestations must reference UMA identities rather than chain-native addresses | Prevents replay, substitution, and encoding-context attacks | SXCP attestation format rules |
| U-6 | UMA resolution must be deterministic and reproducible from public data | Enables independent verification and auditability | Deterministic resolution functions |
| U-7 | UMA resolution must not depend on custodial services, registrars, or mutable naming systems | Eliminates hidden trust assumptions | Protocol-mediated resolution |
| U-8 | UMA resolution must not influence consensus ordering or finality | Prevents identity-layer feedback into consensus | One-way consumption by upper planes |
| U-9 | UMA semantics must be invariant across chains and execution contexts | Prevents context-dependent identity meaning | Chain-agnostic identity model |
| U-10 | UMA must support temporal continuity across epochs | Enables long-lived reputation, governance, and accountability | Epoch-aware identity binding |
| U-11 | UMA-associated authority must be revocable without destroying identity history | Enables remediation without historical erasure | Governance-controlled revocation |
| U-12 | UMA compromise must be containable without cascading protocol failure | Limits blast radius of identity-related attacks | Plane isolation and key rotation |
| U-13 | UMA must not encode application-specific or contract-specific semantics | Prevents tight coupling between identity and execution logic | Separation of identity and execution |
| U-14 | UMA resolution failures must not halt consensus or cross-chain verification | Preserves liveness under partial failures | Asynchronous resolution handling |
| U-15 | UMA must remain valid under cryptographic algorithm transitions | Ensures survivability across PQC evolution | Aegis cryptographic agility |
| U-16 | UMA creation, modification, and revocation must be fully auditable | Enables forensic analysis and governance accountability | Immutable public logs |
| U-17 | UMA authority must not be transferable without protocol-governed authorization | Prevents shadow delegation and authority laundering | Governance-enforced control paths |
| U-18 | UMA must not allow silent reassignment or rebinding | Prevents stealth identity takeover | Explicit state transition rules |

> **4.10 UMA as a Structural Necessity**
>
> UMA is not an optional abstraction layered atop Synergy Network. It is the **identity substrate that makes the rest of the protocol coherent**.
>
> Without UMA:

- cooperative consensus degenerates under key churn,

- deterministic cross-chain attestation collapses into ambiguity,

- post-quantum migration becomes identity-destructive,

- governance loses temporal legitimacy.

> UMA is therefore not a feature. It is a **precondition**.

With architecture, invariants, and identity fully specified, the document now turns to **Proof-of-Synergy (PoSy)**—the mechanism that converts these constraints into deterministic, cooperative consensus.

**5. Identity, Routing, and Execution Separation**

Synergy Network is designed around a strict separation of identity, routing, and execution. This separation is not an implementation preference or layering convenience; it is a system-level invariant required to preserve security, correctness, and long-term evolvability across both the network and wallet domains.

In many blockchain systems, these concerns are conflated. Wallets sign and execute. Contracts embed identity semantics. Routing mechanisms implicitly acquire authority. Over time, such coupling becomes a source of systemic fragility, enabling privilege escalation, identity leakage, and recovery failures.

Synergy explicitly rejects this model.

Instead, Synergy enforces a decomposition in which identity, routing, and execution are orthogonal planes, each with well-defined authority and explicit prohibitions. These planes exist consistently across the base protocol, interoperability mechanisms, smart-contract execution, and wallet architecture.

> **5.1 Identity Plane (Authority Without Execution)**
>
> The Identity Plane is the sole locus of authority within the Synergy system. It defines who an actor is, what cryptographic authority they possess, and how that authority persists over time.
>
> Identity in Synergy is represented exclusively through Universal Meta-Addresses (UMA) and is anchored in the Aegis Post-Quantum Cryptographic Core. The Identity Plane is responsible for:

- binding cryptographic keys to long-lived identities,

  - enforcing key lifecycle rules (generation, rotation, revocation),

  - supporting threshold and distributed authorization,

  - preserving temporal continuity of authority across epochs and upgrades.

> Critically, the Identity Plane does not execute actions. It produces cryptographic evidence of authorization, but it does not interpret intent, route messages, or mutate state.
>
> This prohibition is essential. If identity were allowed to execute or directly effect state transitions, compromise or misuse of identity authority would immediately escalate into systemic failure. By design, identity in Synergy is authoritative but inert.
>
> **5.2 Routing Plane (Intent Without Authority)**
>
> The Routing Plane is responsible for expressing and propagating intent through the system. Intent includes requests to transfer value, invoke contracts, attest to external state, or initiate recovery workflows.
>
> Routing mechanisms in Synergy include, but are not limited to:

- cross-chain intent propagation (SXCP),

- same-chain external transfer coordination (SCETP),

- wallet-level routing decisions,

- transaction and message dissemination.

> The Routing Plane may reference identities (via UMA) to indicate on whose behalf an action is requested, but it never possesses authority to execute those actions independently.
>
> Routing components:

- cannot sign authoritative state transitions,

- cannot mutate identity bindings,

- cannot bypass execution rules,

- cannot reinterpret cryptographic authorization.

> This separation ensures that even a fully compromised routing layer cannot unilaterally move assets, alter governance state, or impersonate identities. Routing may fail, delay, or reorder intent, but it cannot decide outcomes.
>
> **5.3 Execution Plane (Authority Without Identity)**
>
> The Execution Plane is responsible for applying authorized actions to system state. This includes:

- base-layer state transitions finalized by consensus,

- smart-contract execution within SynQ,

- execution of same-chain transfers coordinated via SCETP,

- application of cross-chain outcomes validated through SXCP attestations.

> The Execution Plane consumes authorization artifacts produced by the Identity Plane and intent descriptors propagated by the Routing Plane. It verifies that both are valid and consistent, then applies deterministic rules to update state.
>
> Importantly, the Execution Plane does not define identity. It does not create UMA, rotate keys, or assign authority. It only verifies that valid authority has been presented.
>
> By denying execution components any ability to define or mutate identity, Synergy prevents a large class of escalation attacks in which execution bugs or malicious contracts gain identity-level privileges.
>
> **5.4 Plane Alignment Between Network and Wallet**
>
> The separation of identity, routing, and execution is enforced symmetrically across the Synergy Network and the Synergy Wallet.
>
> At the network level:

- PoSy consensus consumes identity-bound authority but does not define identity.

- SXCP propagates cross-chain intent but cannot execute state changes.

- SynQ executes authorized logic but does not own identity semantics.

> At the wallet level:

- the wallet’s Identity Plane manages root authority and recovery state,

- the wallet’s Routing Plane constructs and dispatches intent,

- the wallet’s Execution Plane signs and submits transactions only after policy and authorization checks.

> This mirroring is intentional. The wallet is not an external convenience layer; it is a protocol-adjacent enforcement boundary that must obey the same architectural rules as the network itself.
>
> Because both domains share the same plane model, properties such as recovery without key resurrection, non-custodial routing, and post-quantum migration can be implemented consistently without introducing special cases or hidden trust assumptions.
>
> **5.5 Cross-Plane Invariants**
>
> The correctness of the Synergy system depends on several cross-plane invariants that must always hold:

- Identity may authorize actions but never execute them.

- Routing may express intent but never grant authority.

- Execution may apply authorized actions but never define identity.

- No plane may silently assume the responsibilities of another.

- Failures in one plane must not compromise the correctness of others.

> These invariants apply globally: to the base protocol, to interoperability mechanisms, to smart-contract execution, and to wallet implementations.
>
> Any design or implementation that violates these invariants—by collapsing planes, introducing implicit authority, or allowing unchecked execution—constitutes a protocol-level defect.
>
> **Architectural Consequences**
>
> By enforcing identity, routing, and execution separation:

- authority becomes durable without being dangerous,

- routing becomes flexible without being trusted,

- execution becomes powerful without being privileged,

- recovery becomes possible without recreating secrets,

- interoperability becomes non-custodial by construction.

> Most importantly, the system remains evolvable. Cryptographic primitives can change, routing mechanisms can expand, and execution environments can grow more expressive without destabilizing identity or governance foundations.
>
> With the architectural separation of identity, routing, and execution established, the document can safely proceed to detailed protocol mechanisms and wallet architecture without risk of conflation or contradiction.
>
> This separation underpins:

- Proof-of-Synergy consensus,

- the Aegis cryptographic core,

- SXCP and SCETP interoperability,

- SynQ execution semantics,

- and the Synergy Wallet’s recovery and policy model.

---

## 3.7 The Cluster-Local DAG Sublayer

> *This section was added to reflect the DAG Validator Cluster Architecture introduced in the Testnet-Beta phase. The normative implementation reference is the Synergy DAG Validator Cluster Integration specification (Parts 1–3).*

### 3.7.1 Role of the DAG in the Consensus Plane

Synergy's Consensus Plane — as defined in Section 3.2 — establishes canonical ordering and deterministic finality through PoSy dual-quorum QC certification. As of the Testnet-Beta phase, the Consensus Plane includes an additional sublayer: a **cluster-local, availability-certified Directed Acyclic Graph (DAG)** that improves transaction dissemination, availability attestation, and deterministic pre-ordering **beneath** the unchanged PoSy QC finality path.

The DAG sublayer is a **data plane and pre-ordering plane**. It does not produce, influence, or shortcut finality. PoSy dual-quorum QC remains the sole canonical settlement artifact. This distinction is non-negotiable and is enforced at the protocol level.

### 3.7.2 Scope and Boundary

The cluster-local DAG operates within each PoSy validator cluster. The cluster is the natural DAG boundary because it is already the BFT boundary for PoSy consensus. **Raw DAG vertex gossip does not cross cluster boundaries.** Cross-cluster communication is limited to compact `ClusterCheckpoints` — cryptographically signed summaries of DAG progress — which are the only DAG artifacts that bridge relay validators consume.

This scope decision eliminates network-wide gossip overhead and prevents the DAG from becoming a cross-cluster MEV or censorship surface.

### 3.7.3 Five Architectural Invariants

The cluster-local DAG is governed by five non-negotiable architectural invariants:

1. **Finality remains QC-based.** The DAG is a data plane only; PoSy QC is the only finality mechanism.
2. **DAG scope is cluster-local.** Only `ClusterCheckpoints` cross cluster boundaries — not raw vertices.
3. **Availability before ordering.** Only DAG vertices certified by ≥ 2f+1 cluster members may influence block ordering.
4. **Encrypted batch invariant.** All transaction data in the DAG is encrypted (`DagBatch` objects); plaintext is never exposed before QC finalization.
5. **Deterministic ordering from on-wire artifacts.** `OrderingCut` construction is a pure function of the certified DAG state — no local randomness or wall-clock time influences which transactions are ordered.

### 3.7.4 Target Data Flow

The target data flow with the DAG sublayer active is:

```
Client Tx → Batch Builder (encrypt → DagBatch)
          → DAG Vertex Builder (DagVertex, parent refs)
          → Cluster Gossip (cluster-scoped P2P only)
          → Availability Receipt Collection (≥ 2f+1 receipts)
          → DagCertificate Formation
          → Certified Frontier Selection (deterministic)
          → Deterministic OrderingCut
          → Block Assembly (with DAG roots in block header)
          → Dual-Quorum Vote Collection [UNCHANGED]
          → QC Finalization → Chain Commit
          → Batch Decryption Key Release (post-QC only)
```

Cross-cluster: `ClusterCheckpoint` only (no raw vertex gossip).

### 3.7.5 Relationship to Existing Architecture Planes

The DAG sublayer does not introduce a new architectural plane. It extends the Consensus Plane's data layer:

- **Consensus Plane:** DAG dissemination and pre-ordering operate inside the cluster; QC finality is unchanged; DAG roots are committed in block headers as informational commitments.
- **Cryptographic Plane (Aegis):** All DAG signatures use ML-DSA-65/87; batch encryption uses ML-KEM-768; all hash functions use Blake3. No DAG-specific cryptographic assumptions are introduced outside Aegis.
- **Interoperability & Execution Plane:** `ClusterCheckpoints` can serve as verified DAG state references for SXCP cross-chain attestations. Batch decryption occurs post-QC and feeds into SynQ execution. The DAG is transparent to SynQ contract semantics.

The composition-without-circular-dependency property of Synergy's architecture is preserved: the DAG operates within the Consensus Plane and does not create dependencies upward toward the Cryptographic or Interoperability planes.

