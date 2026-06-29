**Appendix A — Formal Definitions, Domains, and Invariants**

This appendix establishes the **formal conceptual framework** used throughout the Synergy Network whitepaper. Definitions in this appendix are normative. Where ambiguity exists between informal usage and this appendix, **this appendix governs interpretation**.

**A.1 Identity (UMA)**

An **identity** in Synergy Network is a persistent, protocol-recognized abstract entity defined independently of any specific cryptographic key material, execution environment, or physical device.

Formally, an identity is characterized by:

- a stable identifier (UMA),

- a history of authority states,

- a set of current authorization constraints,

- a cryptographically verifiable lifecycle.

Identity **does not imply authority**. Identity is a reference frame under which authority may be granted, revoked, rotated, or constrained.

Key properties:

- **Continuity**: identity persists across key rotation, cryptographic migration, device loss, and recovery.

- **Attribution**: all protocol-relevant actions are attributable to an identity, not merely to a key.

- **Non-resurrection**: identity continuity does not imply resurrection of deprecated authority.

**A.2 Authority**

**Authority** is the protocol-recognized capability to authorize actions under explicitly defined constraints.

Authority is always:

- scoped,

- conditional,

- time- and generation-bounded,

- cryptographically enforced.

Authority is **never implicit** and **never global**.

Authority domains include, but are not limited to:

- execution authorization,

- governance participation,

- cross-chain attestation,

- recovery coordination.

Authority may be revoked without destroying identity and may evolve independently across domains.

**A.3 Execution**

**Execution** refers to deterministic state transitions occurring within a defined execution environment (e.g., SynQ/SVM).

Execution is:

- local to a chain or execution context,

- conditioned on verified authority,

- deterministic with respect to inputs and verified external facts,

- atomic (commit or abort).

Execution never propagates authority. Authority is always evaluated *before* execution.

**A.4 Planes and Plane Separation**

A **plane** is a domain of responsibility with explicitly bounded authority.

Synergy defines three planes:

- **Identity Plane**: identity state and authority eligibility.

- **Routing Plane**: intent formation and contextual dispatch.

- **Execution Plane**: cryptographic authorization and action.

Plane separation is an invariant:

No plane may unilaterally perform the responsibilities of another.

**A.5 Attestation**

An **attestation** is a cryptographically verifiable statement asserting that a specific condition about external state holds under explicit finality assumptions.

Attestations:

- assert facts, not commands,

- are deterministic,

- are replay-protected by context binding,

- do not imply execution.

**A.6 Failure Containment**

**Failure containment** is the property that failures degrade availability or liveness without escalating authority, corrupting state, or invalidating auditability.

Failure containment is a primary correctness goal.

**  **

**Appendix B — Security Proof Sketches (Structured, Non-Formal)**

This appendix provides **structured proof sketches** for key safety and correctness properties. These sketches are not full formal proofs but are written to demonstrate that claims in the main text are *derivable*, not aspirational.

**B.1 Safety of Proof-of-Synergy Consensus**

**Claim**

PoSy ensures safety (no conflicting finalized states) under partial synchrony and bounded byzantine participation.

**Sketch**

- PoSy employs committee-based BFT consensus with deterministic finality.

- Validator influence is weighted by Synergy Score, not raw stake, but quorum intersection properties are preserved.

- Safety follows from:

  - quorum intersection guarantees,

  - deterministic finalization rules,

  - explicit view-change protocols.

Replacing stake weight with Synergy Score does not alter the algebraic requirement for quorum intersection, only participant selection.

**B.2 Deterministic Cross-Chain Safety (SXCP)**

**Claim**

SXCP prevents incorrect cross-chain execution even under adversarial relayer behavior.

**Sketch**

- Execution depends exclusively on deterministic verification of finalized source-chain state.

- Relayers cannot fabricate finality proofs.

- Conflicting or ambiguous proofs resolve to non-execution.

- No execution path exists that bypasses verification.

Thus, incorrect execution requires cryptographic failure or violation of finality assumptions, not relayer misbehavior.

**B.3 Wallet Compromise Containment**

**Claim**

Wallet compromise does not imply identity takeover or irreversible authority loss.

**Sketch**

- Plane separation prevents execution authority from modifying identity state.

- Authority is generation-scoped; deprecated generations cannot authorize.

- Recovery establishes new authority epochs rather than resurrecting old keys.

- Policy gates block unauthorized signing even under execution-plane compromise.

Thus, compromise degrades availability but does not escalate authority.

**B.4 Governance Constraint Safety**

**Claim**

Governance cannot override protocol invariants.

**Sketch**

- Governance outputs are executed only via SynQ contracts.

- SynQ execution enforces Aegis authorization and execution constraints.

- Governance proposals are typed and scope-bounded.

- Execution outside authorized domains is syntactically and semantically impossible.

**  **

**Appendix C — Cryptographic Architecture and Properties (Non-Enabling)**

This appendix describes **cryptographic properties and relationships**, not concrete parameter choices.

**C.1 Cryptographic Role Separation**

Synergy enforces cryptographic role separation:

- identity verification,

- execution authorization,

- recovery coordination,

- cross-chain attestation.

Keys derived for one role are cryptographically unusable for others.

**C.2 Post-Quantum Security Model**

Synergy assumes adversaries capable of:

- harvesting encrypted data,

- breaking classical public-key cryptography,

- exploiting long-term key reuse.

Mitigation strategy:

- lattice-based cryptographic primitives standardized by NIST,

- cryptographic agility governed at the protocol level,

- hybrid verification during migration.

**C.3 Hash Functions and Entropy Expansion**

Hash functions in Synergy are used for:

- commitments,

- entropy expansion,

- domain separation,

- transcript binding.

Selection criteria include:

- resistance to quantum speedups beyond Grover bounds,

- wide internal state,

- suitability for tree and transcript constructions.

Hash functions are treated as **replaceable protocol components**, not hardcoded primitives.

**C.4 Threshold and Distributed Cryptography**

Threshold mechanisms provide:

- distributed key generation,

- partial authorization,

- aggregation without single-party control.

Threshold values and participant sets are governance-controlled and intentionally omitted.

**  **

**Appendix D — Economic and Incentive Models (Formalized)**

This appendix formalizes **economic behavior**, not numerical parameters.

**D.1 Incentive Alignment Model**

Let participants choose between cooperative behavior (C) and adversarial behavior (A).

Synergy designs payoffs such that:

- expected long-term payoff(C) \> payoff(A),

- adversarial behavior yields:

  - slashing,

  - Synergy Score degradation,

  - reduced future participation.

This holds even under partial collusion.

**D.2 Separation of Capital and Authority**

Capital contributes to participation but does not directly confer authority.

Authority weight = f(contribution, correctness, uptime, governance participation, stake)

Where *f* is monotonic but not linear in stake.

This prevents capital-only capture equilibria.

**D.3 Risk Internalization**

Cross-chain and recovery operations introduce correlated risk.

Synergy internalizes this risk via:

- compensation pools,

- slashing redistribution,

- governance-directed insurance mechanisms.

Losses are not socialized indiscriminately; they are attributed where possible.

**D.4 Long-Term Stability**

Economic parameters are governed, adjustable, and constrained by protocol invariants.

This allows adaptation without:

- breaking identity continuity,

- altering authority semantics,

- invalidating prior execution.

**  **

**Appendix E — Diagrams and System Schematics (Normative Descriptions)**

This appendix defines the **semantic meaning** of all diagrams referenced in the Synergy Network documentation. Diagrams are not illustrative metaphors; they are **formal abstractions** of system structure. Any visual rendering must preserve the relationships described here.

No diagram may contradict the invariants defined in the main body or prior appendices.

**E.1 Global Three-Plane Architecture Diagram**

**Purpose**

To illustrate the universal separation of **identity**, **routing**, and **execution** across all Synergy components.

**Description**

The diagram depicts three parallel planes extending across:

- the network protocol,

- the wallet,

- the execution environment.

Each plane is shown as horizontally continuous but vertically isolated.

- **Identity Plane**

Anchored by UMA. Contains identity state, eligibility, lifecycle, and attribution.

No execution arrows originate from this plane.

- **Routing Plane**

Contains intent formation, context binding, and dispatch logic.

Arrows originate here but do not terminate in state mutation.

- **Execution Plane**

Contains cryptographic authorization and deterministic state transitions.

Execution arrows terminate only after passing through policy and verification gates.

**Invariant Represented**

No arrow flows directly from Identity → Execution.

All execution flows must traverse Routing and Policy enforcement.

**E.2 Proof-of-Synergy Consensus Diagram**

**Purpose**

To depict cooperative consensus without competitive leader dominance.

**Description**

The diagram shows:

- validator clusters formed per epoch,

- Synergy Score–weighted participation,

- propose → vote → commit flow,

- deterministic finality points.

Clusters are depicted as rotating sets, not static validator pools.

**Invariant Represented**

Consensus authority is **collective and time-scoped**, not permanently delegated.

**E.3 Aegis Cryptographic Core Diagram**

**Purpose**

To visualize cryptographic role separation and lifecycle enforcement.

**Description**

The diagram depicts:

- cryptographic domains (identity, execution, recovery, attestation),

- domain-separated key derivation,

- lifecycle transitions (generation rotation, deprecation).

No key is shown spanning multiple domains.

**Invariant Represented**

Cryptographic material is **single-purpose and non-transferrable** across roles.

**E.4 Cross-Chain Attestation (SXCP) Flow Diagram**

**Purpose**

To replace bridge-centric mental models with attestation-centric ones.

**Description**

The diagram shows:

1.  Source chain state finalization

2.  Fact attestation generation

3.  Threshold verification

4.  Local execution conditioned on verified facts

Assets are never depicted as leaving their native chain.

**Invariant Represented**

Cross-chain interaction propagates **facts**, not custody.

**E.5 Wallet Authority Flow Diagram**

**Purpose**

To demonstrate that wallets are authority governors, not signing oracles.

**Description**

The diagram shows:

- user intent entering the Routing Plane,

- policy evaluation as a mandatory gate,

- execution authorization occurring only after policy approval,

- cryptographic signing as the final step.

**Invariant Represented**

Signing is a *consequence* of authorization, not the source of it.

**E.6 Failure Containment Diagram**

**Purpose**

To visualize how failures halt rather than escalate.

**Description**

Failures (validator outage, relayer failure, wallet compromise) are shown terminating flows rather than redirecting them.

**Invariant Represented**

Failure collapses capability, not authority boundaries.

**  **

**Appendix F — Comprehensive Glossary (Canonical Definitions)**

This glossary defines **canonical meanings**. Informal usage elsewhere must conform to these definitions.

**Aegis**

The post-quantum cryptographic core enforcing identity verification, authorization, key lifecycle management, and cryptographic agility.

**Authority**

A bounded, revocable capability to authorize specific actions under protocol constraints.

**Attestation**

A cryptographically verifiable assertion of external state under explicit finality assumptions.

**Execution Plane**

The domain in which deterministic state transitions occur after authorization.

**Guardian**

An identity authorized to participate in recovery workflows under quorum and time constraints.

**Identity (UMA)**

A persistent protocol entity independent of cryptographic keys and devices.

**Policy Engine**

A mandatory pre-signature gate enforcing identity-governed constraints.

**PoSy (Proof-of-Synergy)**

A cooperative BFT consensus mechanism weighting participation by multidimensional contribution rather than stake alone.

**Recovery**

A forward-moving reconstitution of authority, not reconstruction of cryptographic secrets.

**Routing Plane**

The domain responsible for intent formation, context binding, and dispatch.

**SCETP**

Same-Chain External Transfer Protocol enabling identity-routed transfers without cross-chain mechanics.

**SXCP**

Synergy Cross-Chain Protocol enabling deterministic, non-custodial cross-chain coordination.

**SynQ / SVM**

The smart-contract language and virtual machine enforcing deterministic, formally constrained execution.

**Synergy Score**

A composite influence metric incorporating stake, correctness, uptime, and contribution.

**  **

**Appendix G — References, Prior Art, and Differentiation**

This appendix situates Synergy Network within the existing body of work while clarifying **non-derivation** and **novel composition**.

This is not a literature survey; it is a **prior-art boundary**.

**G.1 Consensus Systems**

Synergy builds upon classical BFT foundations (e.g., PBFT-style quorum intersection) but diverges materially by:

- rejecting leader-centric competition,

- rejecting pure stake-based weighting,

- introducing cooperative, contribution-weighted authority.

PoSy is **not** a variant of PoS, DPoS, or classical BFT; it is a new composition.

**G.2 Interoperability Mechanisms**

Existing interoperability approaches rely on:

- custodial bridges,

- wrapped assets,

- oracle-based trust,

- optimistic verification.

SXCP differs fundamentally by:

- eliminating custody entirely,

- propagating attestations instead of assets,

- enforcing deterministic verification,

- resolving ambiguity via non-execution.

**G.3 Wallet and Key Management Systems**

Most wallets:

- conflate identity with keys,

- treat recovery as key reconstruction,

- expose signing as the primary security boundary.

Synergy Wallet introduces:

- identity continuity independent of keys,

- recovery without key resurrection,

- plane-separated authority,

- policy-governed execution.

These properties do not exist in combination in prior art.

**G.4 Post-Quantum Cryptography Integration**

Existing systems typically:

- retrofit PQC at the application layer,

- treat migration as a breaking event,

- ignore endpoint cryptography.

Synergy integrates PQC:

- at the protocol level,

- at the wallet level,

- with cryptographic agility and hybrid verification,

- without identity or execution breakage.

**G.5 Non-Obvious Composition**

The novelty of Synergy Network does not reside in any single primitive.

It resides in:

- the enforced separation of identity, routing, and execution,

- the coupling of governance, cryptography, and execution,

- the unified treatment of wallet and network threat models,

- the emergence of patent-relevant technical effects from interaction.

This composition is **not suggested or implied** by existing systems when considered individually.

**Appendix H — Sustainability Metrics Methodology (MiCA/ESMA-Aligned)**

This appendix defines the measurement boundaries, formulas, assumptions, emissions factor selection logic, uncertainty treatment, and telemetry plan used to produce Synergy Network’s sustainability disclosures for Proof-of-Synergy (PoSy). It is written to be operationally implementable, audit-friendly, and robust against “fake precision.”

The intent is twofold:

1.  **Comparability:** Metrics should be expressed in units and structures commonly used for crypto-asset sustainability reporting (energy, emissions, intensity per transaction).

2.  **Integrity:** Where measurement is incomplete, Synergy must disclose modeling choices, ranges, and confidence levels, and then converge toward direct measurement over time.

**H.1 Boundaries**

**H.1.1 Reporting entity and mechanism in scope**

This methodology covers the environmental impact associated with operating the **PoSy consensus mechanism** and the **minimum supporting infrastructure required for the network to function**.

PoSy’s sustainability profile is dominated by standard distributed-systems operations (compute, storage, networking, and hosting overhead). There is no competitive mining workload.

**H.1.2 Inclusion boundary (what is counted)**

The following are included in the reporting boundary **when they are required to achieve network consensus and core operation**:

**A) Validator infrastructure (consensus-critical)**

- Active consensus validator nodes participating in PoSy

- Standby / failover validator instances *if they are maintained specifically to support uptime and consensus continuity*

- Validator storage required for consensus (state DBs, block storage, logs necessary for safety/liveness)

**B) Protocol-operated infrastructure (if operated by Synergy directly)**

Only include these if they are operated/paid for by Synergy Network as part of the protocol’s minimal operation:

- Bootstrapping or seed nodes (if protocol-operated)

- Official archival nodes (if protocol-operated)

- Official telemetry/monitoring infrastructure (if protocol-operated)

- Official public RPC endpoints (if protocol-operated and positioned as part of baseline network access)

**C) Network overhead**

- Data-center overhead for included IT loads, accounted via PUE (Power Usage Effectiveness) or provider-specific overhead where available.

**H.1.3 Optional/extended boundary (disclosed separately if reported)**

These components may be relevant, but they are **not strictly consensus-critical**. If Synergy chooses to include them for fuller disclosure, they should be reported as a **separate line item** (“extended boundary”) to avoid blending apples with oranges:

- Relayer networks (SXCP relayers), witness registries, and cross-chain infrastructure not strictly required for base PoSy finality

- Explorer infrastructure and analytics endpoints

- Developer tooling infrastructure and CI/CD (unless explicitly declared as “protocol critical”)

- End-user wallet device energy consumption (phones/laptops)

- Third-party infrastructure (community RPCs, community explorers) unless formally designated as part of “baseline protocol service”

**H.1.4 Exclusion boundary (what is not counted)**

Unless explicitly included under the extended boundary, the following are excluded:

- Energy use of end-user devices

- Energy use of unrelated applications deployed on Synergy

- Corporate office energy consumption (unless Synergy elects to disclose corporate footprint separately)

- Upstream emissions of third-party vendors beyond what is necessary to compute electricity-related emissions factors

**H.1.5 Reporting period and cadence**

- Default reporting period: **annual**, with optional **quarterly** interim updates once telemetry coverage is stable.

- All intensity metrics (e.g., Wh/tx) must use transaction counts from the **same reporting period** as energy totals.

**H.2 Formulas**

This section defines the computation of core indicators. All computations must preserve units and document conversions.

**H.2.1 Power and energy conversions**

- ***1 kW = 1000 W***

- ***1 kWh = 1000 Wh***

- Annual hours (non-leap year): ***H<sub>yr</sub> = 8760***

- Reporting-period hours: compute from timestamps for better accuracy (especially for partial years).

**H.2.2 Node-level IT energy (measured)**

For each node $`\mathbf{i}`$, for time window $`\mathbf{t}`$:

- If direct power measurement exists:

> $`\mathbf{E}_{\mathbf{i,t}}^{\mathbf{IT}}\mathbf{= \ }\int_{\mathbf{t}}^{\mathbf{\ }}{\mathbf{P}_{\mathbf{i}}^{\mathbf{IT}}\left( \mathbf{\tau} \right)\mathbf{d\tau}}`$

- In implementation, this is a discrete sum:

> 
> ``` math
> \mathbf{E}_{\mathbf{i,t}}^{\mathbf{IT}}\mathbf{\approx \ }\sum_{\mathbf{k = 1}}^{\mathbf{n}}{\mathbf{P}_{\mathbf{i,k}}^{\mathbf{IT}}\mathbf{\  \times \ \Delta}\mathbf{t}_{\mathbf{k}}}
> ```

- where $`\mathbf{P}_{\mathbf{i,k}}^{\mathbf{IT}}\mathbf{\ }`$is average watts during interval $`\mathbf{k,}`$ $`\mathbf{\Delta}\mathbf{t}_{\mathbf{k}}\mathbf{\ }`$is hours.

**H.2.3 Node-level IT energy (modeled)**

If power is not measured, estimate IT power using a conservative model:

> 
> ``` math
> \mathbf{P}_{\mathbf{i}}^{\mathbf{IT}}\mathbf{=}\mathbf{P}_{\mathbf{idle}}\mathbf{+ \ }\mathbf{u}_{\mathbf{i}}\mathbf{\  \times \ (}\mathbf{P}_{\mathbf{peak}}\mathbf{-}\mathbf{P}_{\mathbf{idle}}\mathbf{)}
> ```

Where:

- P<sub>idle</sub>: baseline watts at idle

- P<sub>peak</sub>: watts at high utilization

- **u*<sub>i </sub>***: utilization factor in \[0,1\], derived from CPU/network/storage telemetry

Then:

> 
> ``` math
> \mathbf{E}_{\mathbf{i,t}}^{\mathbf{IT}}\mathbf{=}\mathbf{P}_{\mathbf{i}}^{\mathbf{IT}}\mathbf{\times}\mathbf{H}_{\mathbf{i,\ t}}
> ```

where $`\mathbf{H}_{\mathbf{i,\ t}}`$ is node uptime hours in period **t**.

**H.2.4 Facility energy using PUE**

Data-center overhead is incorporated by multiplying IT energy by PUE (or an equivalent provider-specific overhead factor).

> 
> ``` math
> \mathbf{E}_{\mathbf{t}}^{\mathbf{facility}}\mathbf{=}\sum_{\mathbf{i}}^{}\mathbf{E}_{\mathbf{i,t}}^{\mathbf{IT}}\mathbf{\times PU}\mathbf{E}_{\mathbf{i,t}}
> ```

If node-specific PUE is unknown, use a default PUE for the reporting class (e.g., “industry average” or provider published PUE) and disclose it.

**H.2.5 Network energy per transaction**

Let **TX<sub>t</sub>** be finalized transactions in period **t**.

> 
> ``` math
> \mathbf{Wh/t}\mathbf{x}_{\mathbf{t}}\mathbf{=}\frac{\mathbf{E}_{\mathbf{t}}^{\mathbf{facility}}\mathbf{\times 1000}}{\mathbf{T}\mathbf{X}_{\mathbf{i}}}
> ```

If reporting in kWh/tx:

> 
> ``` math
> \mathbf{kWh/t}\mathbf{x}_{\mathbf{t}}\mathbf{=}\frac{\mathbf{E}_{\mathbf{t}}^{\mathbf{facility}}}{\mathbf{T}\mathbf{X}_{\mathbf{t}}}
> ```

**H.2.6 Network energy per active validator hour (optional operational KPI)**

This is useful for operational efficiency and sanity-checking:

> 
> ``` math
> \mathbf{Wh\ per\ validator\ }\mathbf{hour}_{\mathbf{t}}\mathbf{=}\frac{\mathbf{E}_{\mathbf{t}}^{\mathbf{facility}}\mathbf{\times 1000}}{\sum_{\mathbf{i}}^{}\mathbf{H}_{\mathbf{i,t}}}
> ```

**H.2.7 Emissions (location-based electricity method)**

Let **CI<sub>i,t</sub>** be carbon intensity (kg CO₂e/kWh) for node i in period t.

> 
> ``` math
> \mathbf{CO2}\mathbf{e}_{\mathbf{t}}\mathbf{(kg) =}\sum_{\mathbf{i}}^{}{\mathbf{(}\mathbf{E}_{\mathbf{i,t}}^{\mathbf{facility}}}\mathbf{\times}\mathbf{CI}_{\mathbf{i,t}}
> ```

Convert to metric tons:

> 
> ``` math
> \mathbf{CO2}\mathbf{e}_{\mathbf{t}}\mathbf{= \ }\frac{\mathbf{CO2}\mathbf{e}_{\mathbf{t}}\mathbf{\ (kg)}}{\mathbf{1000}}
> ```

**H.2.8 Emissions per transaction**

> 
> ``` math
> \mathbf{gCO2e/t}\mathbf{x}_{\mathbf{t}}\mathbf{= \ }\frac{\mathbf{tCO2}\mathbf{e}_{\mathbf{t}}\mathbf{\ x\ }\mathbf{10}^{\mathbf{6}}}{\mathbf{T}\mathbf{X}_{\mathbf{t}}}
> ```

(because 1t = 1,000,000 g)

**H.2.9 E-waste (qualitative baseline + quantifiable pathway)**

Quantification is hard without strong hardware inventory reporting, so Synergy uses a staged approach:

**Stage 1 (qualitative):** disclose hardware type (commodity vs specialized), typical replacement cycles, and recycling policy.

**Stage 2 (semi-quant):** estimate hardware mass and replacement rate from declared inventories.

A simple quant model:

> 
> ``` math
> \mathbf{Ewast}\mathbf{e}_{\mathbf{t}}\mathbf{(kg) =}\sum_{\mathbf{i}}^{}\mathbf{m}_{\mathbf{i}}\mathbf{\times}\mathbf{r}_{\mathbf{i,t}}
> ```

Where:

- **m<sub>i</sub>** is mass of node hardware (kg)

- **r<sub>i,t</sub>** is fraction replaced/disposed in period **t**

All e-waste figures must be clearly labeled as **inventory-derived** unless validated by disposal records.

**H.3 Assumptions**

Assumptions exist to fill gaps until metering coverage is complete. The rule is: **assumptions must be explicit, conservative, and replaceable**.

**H.3.1 Operational assumptions**

- Validators are expected to run **24×7**, but actual uptime is measured and used when available.

- Validators may run on bare metal, hosted servers, or cloud VMs; power modeling differs by category.

**H.3.2 Hardware power assumptions (modeled nodes)**

When direct metering is absent, Synergy may use a conservative per-node power envelope based on validator class:

- Commodity server class: define **P<sub>idle</sub>** and **P<sub>peak</sub>** bands

- VM class: infer power by mapping VM allocation to host estimates or provider sustainability disclosures

These bands must be documented in the reporting package and periodically revised as telemetry improves.

**H.3.3 PUE assumptions**

- Prefer **provider-specific PUE** where published and applicable.

- If unavailable, use a declared default (e.g., an “industry average” PUE) and disclose that it is a proxy.

**H.3.4 Transaction counting assumptions**

- Use **finalized transactions** only.

- Define whether internal system messages, governance transactions, and protocol maintenance transactions count as transactions; apply consistently and disclose the definition once.

**H.3.5 Boundary assumptions for “protocol-operated”**

Synergy should explicitly list what infrastructure it operates as part of baseline protocol service (if any). Anything not listed is not included unless part of the extended boundary.

**H.4 Emissions Factors Used and Rationale**

Emissions factors can be a political minefield and a greenwashing playground. The methodology therefore prioritizes **traceability and comparability**.

**H.4.1 Hierarchy for selecting emissions factors (best → fallback)**

1.  **Node-specific provider factor** (best):

If the validator runs in a facility/provider that publishes region- and time-specific carbon intensity (or energy attribute), use that.

2.  **Grid region factor** (good):

If the node’s region is known (country/state/balancing authority), apply the corresponding grid emissions intensity.

3.  **National average factor** (acceptable fallback):

If only the country is known, use national average intensity.

4.  **Global average factor** (last resort):

If region cannot be determined, apply a global average factor and label it as such.

**H.4.2 Location-based vs market-based**

Synergy should default to **location-based** emissions (grid intensity) for comparability and integrity. Market-based (offsets/RECs) can be disclosed separately as a supplement, not as a replacement, unless required by a specific reporting framework.

**H.4.3 Time matching and temporal granularity**

If possible, apply time-matched emissions factors (monthly or hourly) for higher accuracy. If not, apply annual averages and disclose temporal limitations.

**H.4.4 Rationale for “conservative bias”**

Where uncertainty exists, Synergy should avoid the temptation to pick favorable factors. A conservative bias (err high) is reputationally and regulatorily safer than optimistic modeling.

**H.5 Uncertainty Ranges and Measurement vs Modeled Split**

This section is the difference between “responsible disclosure” and “marketing cosplay.”

**H.5.1 Data quality tiers**

Synergy should label reported metrics with a data quality tier, based on the share of energy that is directly measured.

Example tiers:

- **Tier 1 (High):** ≥80% of validator energy is directly metered or provider-reported

- **Tier 2 (Medium):** 40–80% metered/provider-reported, remainder modeled

- **Tier 3 (Low):** \<40% metered/provider-reported, heavy modeling reliance

The tier is not a shame badge; it’s honesty.

**H.5.2 Measurement vs modeled split (mandatory disclosure)**

For each reporting period, disclose:

- $`\mathbf{\%}\mathbf{E}_{\mathbf{measured}}\mathbf{=}\frac{\mathbf{E}_{\mathbf{measured}}^{\mathbf{facility}}}{\mathbf{E}_{\mathbf{total}}^{\mathbf{facility}}}\mathbf{\times 100}`$

- $`\mathbf{\%}\mathbf{E}_{\mathbf{modeled}}\mathbf{= 100 - \%}\mathbf{E}_{\mathbf{measured}}`$

Same disclosure for emissions if emissions factors differ in quality.

**H.5.3 Uncertainty ranges**

Synergy should publish ranges for key outputs when modeling is material.

A practical method:

- For modeled nodes, compute energy using **low/median/high** assumptions (e.g., 60 W / 75 W / 150 W IT load) and propagate through PUE and emissions factors.

- Publish:

  - **Low scenario, Mid scenario, High scenario** for kWh/year and tCO₂e/year

  - Corresponding intensity ranges (Wh/tx; gCO₂e/tx)

A stronger method (optional, more rigorous):

- Monte Carlo simulation using distributions for node power, PUE, and carbon intensity.

- Publish percentiles (P10, P50, P90) and document distribution choices.

**H.5.4 Outliers and anomaly handling**

Outliers are expected (someone runs a monster server; someone misreports). Synergy should:

- flag abnormal nodes,

- exclude clearly erroneous telemetry from aggregates (with documented reasons),

- and keep an “anomaly log” for auditability.

**H.5.5 Anti-gaming safeguards**

If sustainability metrics have reputational or economic implications, validators may game them. Minimal safeguards:

- require periodic attestations,

- cross-check declared hardware classes against observed resource profiles,

- optionally require third-party energy reports for “green claims.”

**H.6 Telemetry Plan**

This is the operational “how we stop guessing.”

**H.6.1 Objectives**

- Obtain sufficient telemetry to compute network-wide kWh/year and tCO₂e/year with narrowing uncertainty.

- Increase the measured share over time.

- Preserve validator privacy and security (no doxxing, no sensitive infra exposure).

**H.6.2 What is collected (minimum viable telemetry)**

At minimum, Synergy should collect (or enable validators to submit) the following per node, per reporting interval:

**Node identity and classification (pseudonymous)**

- Validator ID (non-personally identifying)

- Hosting type category: bare metal / colocation / cloud VM

- Region granularity: country + (optional) subregion for emissions factor mapping

**Uptime and workload**

- Uptime hours

- CPU utilization percentiles (P50/P95)

- Network ingress/egress (bytes)

- Storage footprint (GB)

- Transaction processing counts (if node-level available)

**Energy / power (preferred)**

- Direct measured power draw (W) time series or interval averages, OR

- Provider-reported energy use, OR

- Sufficient data to support modeled power estimation

**PUE / overhead**

- Provider PUE or facility overhead factor if known

- If unknown, apply default and label as modeled

**H.6.3 Collection mechanisms (privacy-aware options)**

A practical staged rollout:

**Stage 0 — Baseline modeling**

- No validator energy submission required

- Use default power envelopes + uptime + validator count for scenario estimates

**Stage 1 — Voluntary reporting**

- Validators can submit energy reports (metered or provider-based)

- Emphasize privacy: submit aggregated kWh for period, not granular traces if not needed

**Stage 2 — Standardized attestation**

- Validators submit signed attestations of:

  - hardware class,

  - hosting type,

  - region,

  - energy for period (metered/provider/modeled)

- Include penalties for fraudulent attestations if governance chooses

**Stage 3 — High-integrity measurement (selective)**

- For large validators or entities making sustainability claims, require higher assurance (e.g., provider sustainability report, third-party audit statement, or verifiable metering)

**H.6.4 Reporting pipeline**

- Data ingestion → validation → aggregation → publication

- Every reporting release should include:

  - methodology version,

  - boundary declaration,

  - measured vs modeled split,

  - emissions factor sources used,

  - and scenario/uncertainty ranges.

**H.6.5 Update triggers and governance**

Synergy should update published sustainability metrics when:

- mainnet launches (initial telemetry baseline),

- validator population changes materially (e.g., +/−25%),

- a protocol upgrade changes compute/network load materially,

- measurement coverage crosses a threshold (e.g., Tier 3 → Tier 2).

**H.6.6 Integrity controls and auditability**

- Keep an internal reproducibility package (inputs + formula implementation + outputs)

- Maintain a changelog of assumption updates

- Provide a public summary sufficient for third-party review without leaking validator-sensitive details
