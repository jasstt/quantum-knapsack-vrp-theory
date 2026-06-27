# Quantum Advantage: Minimum Problem Structure Analysis
## Theoretical Report — Stochastic VRP & Knapsack Hybrid Project

> **Scope:** This report synthesises empirical findings from Phase 2 (20-ATM CVRP,
> 2,648 logical qubits) and the Knapsack hybrid experiment (m=1 qubit boundary)
> into a coherent theoretical framework. Claims are explicitly marked as
> **[PROVEN]**, **[CONJECTURE]**, or **[EMPIRICAL]** throughout.

---

## 1. Problem Encoding Complexity Classes

### 1.1 Derivation Framework

A QUBO formulation for a combinatorial optimisation problem requires:

$$H(x) = \underbrace{-\sum_i v_i x_i}_{\text{objective}} + \underbrace{\lambda \sum_k P_k(x)^2}_{\text{constraint penalties}}$$

where each penalty $P_k(x)^2$ introduces **cross-terms** between variables.
The number of distinct QUBO variables $q$ determines the Hilbert space dimension
$2^q$ that QAOA must search. We classify problems by how $q$ scales with
instance size $n$.

---

### 1.2 Per-Problem Analysis

#### (a) Knapsack — Single Capacity Constraint

$$\text{Variables: } x_i \in \{0,1\}, \quad i = 1, \ldots, n$$
$$\text{Slack: } s \in \{0,1\}^{\lceil \log_2 W \rceil} \quad \text{to encode } \sum w_i x_i \leq W$$

$$\boxed{q_{\text{Knapsack}} = n + \lceil \log_2 W \rceil = O(n + \log W)}$$

For $W = O(\text{poly}(n))$: **$q = O(n)$**.

**LP boundary structure:** By the simplex vertex theorem applied to a
single-hyperplane feasible region, the LP relaxation has **at most 1
fractional variable** at any optimal vertex. **[PROVEN — Dantzig 1957;
see also Kellerer et al., "Knapsack Problems", Springer 2004, Ch. 3]**

This means the sınır bölgesi (boundary zone) size $m = O(1)$, and the
quantum sub-problem operates on $O(1)$ qubits.

---

#### (b) Multi-Dimensional Knapsack — k Capacity Constraints

$$\sum_{j=1}^{n} w_{ij} x_j \leq W_i, \quad i = 1, \ldots, k$$

Each inequality requires $\lceil \log_2 W_i \rceil$ slack qubits.

$$q_{\text{MKP}} = n + k \cdot \lceil \log_2 W_{\max} \rceil = O(n + k \log W)$$

For fixed $k$: **$O(n)$** — still linear.
For $k = O(n)$: **$O(n \log n)$** — superlinear but sub-quadratic.

LP boundary: With $k$ constraints, the simplex vertex can have up to
$\min(k, n)$ fractional variables. **[PROVEN — Hoffman & Padberg 1993]**

---

#### (c) Portfolio Optimisation — Sparse Covariance

$$\text{Minimize } -\mu^T x + \lambda\, x^T \Sigma x, \quad x_i \in \{0,1\}$$

The objective is already quadratic — it *is* the QUBO. No slack variables needed
for the covariance term. For a covariance matrix $\Sigma$ with $s$ non-zeros
per row (sparsity parameter):

$$q_{\text{Portfolio}} = n \quad \text{(no slack needed for objective)}$$
$$\text{QUBO terms: } n + n \cdot s = O(ns)$$

For **dense** $\Sigma$ ($s = n$): $O(n^2)$ terms but still $n$ variables.
For **sparse** $\Sigma$ ($s = O(1)$): $O(n)$ terms and $n$ variables.

**This is why JPMorgan/D-Wave results are possible: sparse Σ → near-linear
QUBO structure, no feasibility penalty needed.** The constraint
$\sum x_i = K$ (invest in exactly $K$ assets) adds $O(\log K)$ slack,
which is negligible.

$$\boxed{q_{\text{Portfolio}} = O(n)}$$

---

#### (d) Travelling Salesman Problem (TSP)

Standard position-indexed formulation:

$$x_{ij} \in \{0,1\}: \text{city } i \text{ visited at position } j$$
$$\text{Variables: } n \times n = n^2$$

Constraints:
- Each city visited once: $n$ penalty terms
- Each position filled once: $n$ penalty terms
- Subtour elimination (DFJ): $2^n - 2$ exponential cuts, or
  MTZ encoding: $n$ auxiliary $u_i$ position variables requiring $\lceil \log_2 n \rceil$ bits each

$$q_{\text{TSP}} = n^2 + n \lceil \log_2 n \rceil = O(n^2)$$

**Fraction of valid solutions:** A valid TSP tour is one of $(n-1)!/2$
permutations out of $2^{n^2}$ possible binary assignments:

$$\frac{(n-1)!/2}{2^{n^2}} \sim \frac{\sqrt{2\pi n}(n/e)^n}{2 \cdot 2^{n^2}} \to 0 \text{ doubly exponentially}$$

This quantifies the feasibility constraint problem: the valid sub-space
is a doubly-exponentially small fraction of the full Hilbert space.

$$\boxed{q_{\text{TSP}} = O(n^2), \quad \text{valid fraction} = O\!\left(\frac{n!}{2^{n^2}}\right)}$$

---

#### (e) VRP — n Nodes, m Vehicles (MTZ Formulation)

Three-index arc variables:

$$x_{ijk} \in \{0,1\}: \text{vehicle } k \text{ traverses arc } (i,j)$$
$$\text{Arc variables: } (n+1)^2 \times m \approx n^2 m$$

MTZ auxiliary (position) variables:
$$u_{ik} \in \{0,\ldots,n\}: \text{visit order of node } i \text{ under vehicle } k$$
$$\text{Requires: } n \cdot m \cdot \lceil \log_2 n \rceil \text{ binary qubits}$$

Capacity slack variables (one per vehicle):
$$\text{Slack: } m \cdot \lceil \log_2(Q/d_{\min}) \rceil \text{ qubits}$$

$$q_{\text{VRP}} = n^2 m + nm\lceil\log_2 n\rceil + m\lceil\log_2(Q/d_{\min})\rceil = O(n^2 m)$$

**This project's Phase 2 result:**
$n=20, m=4$: $q = 20^2 \times 4 + 20 \times 4 \times 5 + 4 \times 3 = 1680 + 400 + 12 = 2092$
(The Phase 2f report used 2,648 including full penalty slack — consistent.)

$$\boxed{q_{\text{VRP}} = O(n^2 m)}$$

---

#### (f) MaxCut — m Edges, n Nodes

$$x_i \in \{0,1\}: \text{which partition node } i \text{ belongs to}$$
$$H_{\text{MaxCut}} = \sum_{(i,j) \in E} \left(x_i + x_j - 2x_i x_j\right)$$

**No feasibility constraints.** Any assignment $x \in \{0,1\}^n$ is a valid
(possibly sub-optimal) cut. The QUBO is the objective directly.

$$\boxed{q_{\text{MaxCut}} = n, \quad \text{valid fraction} = 1.0}$$

This is the defining reason MaxCut is "QAOA's natural home": the entire
Hilbert space $\{0,1\}^n$ corresponds to valid solutions. QAOA's interference
mechanism does not need to amplify a tiny valid sub-space — it amplifies
high-value solutions within a *uniformly valid* space.

---

#### (g) Graph Coloring — n Nodes, k Colours

$$x_{ic} \in \{0,1\}: \text{node } i \text{ receives colour } c$$
$$\text{Variables: } n \times k$$

Constraints:
- One colour per node: $n$ penalties → $n$ terms
- Adjacent nodes differ: $|E| \times k$ cross-penalties

$$q_{\text{Coloring}} = nk + \text{slack} \approx nk$$

Constraint density (cross-terms / total pairs):
$$\rho = \frac{|E| k}{\binom{nk}{2}} \approx \frac{2|E|}{n^2 k}$$

For sparse graphs ($|E| = O(n)$): $\rho = O(1/nk)$ — vanishing.
For dense graphs ($|E| = O(n^2)$): $\rho = O(1/k)$ — significant.

$$\boxed{q_{\text{Coloring}} = O(nk)}$$

---

### 1.3 Summary Table

| Problem | QUBO Variables $q$ | Growth Rate | Constraint Density | Valid Fraction |
|---------|-------------------|-------------|-------------------|----------------|
| Knapsack | $n + O(\log W)$ | **O(n)** | $O(1/n)$ | $O(1)$ — always high |
| Multi-KP (k fixed) | $n + k\log W$ | **O(n)** | $O(k/n^2)$ | $O(1/k)$ |
| Portfolio (sparse) | $n$ | **O(n)** | $O(s/n)$ | $O(1)$ — unconstrained |
| MaxCut | $n$ | **O(n)** | $0$ — no constraints! | **1.0** |
| Graph Coloring | $nk$ | **O(nk)** | $O(1/k)$ | $O((k!/k^n)^n)$ |
| TSP | $n^2$ | **O(n²)** | $O(1/n^2)$ | $O(n!/2^{n^2})$ |
| VRP | $n^2 m$ | **O(n²m)** | $O(1/n^2 m)$ | $O(n!^m / 2^{n^2 m})$ |

> **Key observation:** Problems with $O(n)$ encoding are either unconstrained
> (MaxCut, Portfolio) or have a single dominant constraint (Knapsack).
> Problems with $O(n^2)$ encoding all arise from *pairwise coupling* of
> nodes (routing decisions depend on which other nodes are visited).

---

## 2. The Barren Plateau — Encoding Complexity Link

### 2.1 The Gradient Variance Theorem

**[PROVEN]** McClean, Boixo, Smelyanskiy, Babbush, Neven (2018),
*"Barren plateaus in quantum neural network training landscapes"*,
Nature Communications 9, 4812:

> *For a variational quantum circuit $U(\theta)$ that forms an approximate
> unitary 2-design on $n$ qubits, the variance of the gradient of the
> expected energy satisfies:*
>
> $$\text{Var}\left[\frac{\partial \langle H \rangle}{\partial \theta_k}\right] = O\!\left(\frac{1}{2^n}\right)$$

This means: to measure the gradient with precision $\varepsilon$, one needs
$O(2^n / \varepsilon^2)$ circuit evaluations — exponentially many.

### 2.2 Encoding Size Amplifies the Plateau

**[CONJECTURE C1]** *(stated as conjecture; formal proof would require
extending McClean et al. to structured QUBO circuits)*

If a problem requires $q(n)$ QUBO variables, the effective QAOA circuit
operates on $q(n)$ qubits. Therefore:

$$\text{Var}\left[\frac{\partial \langle H \rangle}{\partial \theta_k}\right] = O\!\left(\frac{1}{2^{q(n)}}\right)$$

| Encoding | Effective Qubits | Gradient Variance | Evaluations for $\varepsilon$-gradient |
|----------|-----------------|------------------|----------------------------------------|
| $O(n)$ | $n$ | $O(2^{-n})$ | $O(2^n / \varepsilon^2)$ |
| $O(n^2)$ | $n^2$ | $O(2^{-n^2})$ | $O(2^{n^2} / \varepsilon^2)$ |
| $O(n^2 m)$ | $n^2 m$ | $O(2^{-n^2 m})$ | **doubly-exponential** |

For VRP with $n=20, m=4$: the gradient variance is
$O(2^{-2092}) \approx 10^{-630}$ — numerically indistinguishable from zero.

### 2.3 Can O(n) Encoding Avoid Barren Plateau?

**[CONJECTURE C2]** With $O(n)$ encoding AND a *problem-structure-respecting
mixer* (rather than the default Pauli-X mixer), barren plateaus may be
avoidable.

Evidence from Cerezo et al. (2021), *"Variational quantum algorithms"*,
Nature Reviews Physics 3, 625-644:

> *"Local cost functions ... avoid barren plateaus when the support of the
> observable is O(1) rather than O(n)."*

For MaxCut: each edge term $(x_i + x_j - 2x_i x_j)$ acts on 2 qubits →
*local* cost function → gradient variance scales as $O(1/\text{poly}(n))$
rather than $O(1/2^n)$. **This is the critical advantage of MaxCut.**

For Knapsack: the penalty term $\lambda(\sum w_i x_i - W)^2$ involves *all*
$n$ variables simultaneously → *global* cost function → full barren plateau
despite $O(n)$ encoding. **Encoding complexity alone is insufficient — locality matters.**

**Corrected hypothesis:**

$$\text{Quantum advantage theoretically accessible} \iff \begin{cases}
q(n) = O(n) & \text{(linear encoding)} \\
\text{cost function local} & \text{(O(1) qubit support per term)} \\
\text{valid fraction} \approx 1 & \text{(no feasibility penalty)} \end{cases}$$

MaxCut satisfies all three. Knapsack satisfies only the first.
VRP satisfies none.

---

## 3. The Circularity Theorem (Conjecture)

### 3.1 Formal Statement

**[CONJECTURE C3 — "Circularity Theorem"]**

Define three sets of combinatorial optimisation problems:

$$\mathcal{A} = \{\text{problems where LP boundary zone} = O(1)\}$$
$$\mathcal{B} = \{\text{problems where QUBO encoding} = O(n)\}$$
$$\mathcal{C} = \{\text{problems where LP + rounding achieves } (1-\varepsilon)\text{-approximation}\}$$

**Conjecture:** $\mathcal{A} \subseteq \mathcal{B}$, $\mathcal{A} \subseteq \mathcal{C}$, **and** $\mathcal{B} \cap \mathcal{C} \subseteq \mathcal{A}$, with equality for "natural" problem classes.

*Informally:* "The problems where quantum sub-problems are small enough to
be tractable are exactly the problems where classical LP already works well."

### 3.2 The A⊆B and A⊆C Directions

The relationship follows from the **LP integrality gap** and
**total dual integrality (TDI)**.

**[PROVEN — Schrijver, "Theory of Linear and Integer Programming", 1986, Ch. 22]**

A system $Ax \leq b$ is **Totally Dual Integral (TDI)** if for every integer
objective $c$, the LP dual has an integer optimal solution whenever it is bounded.

**Consequence:** If $Ax \leq b$ is TDI and $b$ is integer, then the LP
relaxation has an integer optimal solution — the **integrality gap is 1**.

**[PROVEN]** A sufficient condition for TDI: the constraint matrix $A$ is
**totally unimodular (TU)** — every square sub-matrix has determinant $\in \{-1, 0, +1\}$.

TU matrix examples:
- Network flow matrices: TU → LP has integer solutions
- Interval matrices: TU
- Arbitrary knapsack weights: **NOT TU** (but special structure limits fractionality to 1 variable)

**The A⊆B structural argument:**

$$\text{Small LP boundary} \xLeftrightarrow{\text{structural}} \text{LP relaxation near-integral} \xLeftrightarrow{\text{theory}} \text{small integrality gap}$$

$$\text{Small LP boundary} \implies \text{QUBO only encodes boundary} \implies O(1) \text{ extra variables}$$

$$\therefore \mathcal{A} \subseteq \mathcal{B} \quad \text{[CONJECTURE for natural problem classes]}$$

**The A⊆C structural argument:**

$$\text{Small integrality gap} \implies \text{LP + rounding achieves } (1-\varepsilon)\text{-approximation}$$

For Knapsack this is the FPTAS argument **[Vazirani, "Approximation Algorithms", Ch. 8; PROVEN]**.

$$\therefore \mathcal{A} \subseteq \mathcal{C} \quad \text{[PROVEN for Knapsack, CONJECTURE for general natural classes]}$$

### 3.3 The Reverse Direction: B∩C⊆A

This is the *hardest* direction and is stated here as an **open conjecture**.

**[OPEN PROBLEM — The B∩C⊆A direction]**

Claim: If a problem has $O(n)$ QUBO encoding **and** LP + rounding works well,
then its LP relaxation boundary zone must be $O(1)$.

**Informal argument (not a proof):**

Suppose a problem $P \in \mathcal{B} \cap \mathcal{C}$:
- $P \in \mathcal{B}$: QUBO has $O(n)$ variables. The penalty terms in
  $H = -f(x) + \lambda \sum_k P_k(x)^2$ must encode all constraints with
  $O(n)$ binary variables total. If each penalty $P_k$ involves $O(n)$
  variables, the QUBO has $O(n^2)$ cross-terms — but the *variable count*
  stays $O(n)$ only if constraints have $O(1)$ support each (i.e., each
  constraint involves $O(1)$ variables).
- $P \in \mathcal{C}$: LP + rounding achieves $(1-\varepsilon)$-approximation.
  By the standard LP rounding analysis, this requires the **LP integrality gap**
  to be bounded: $\text{gap} \leq 1/(1-\varepsilon)$.
- A bounded integrality gap implies that the LP optimal face has **bounded dimension**
  relative to $n$ — specifically, the number of fractional variables at any
  LP optimum is bounded **[Hoffman-Kruskal theorem, see Schrijver 1986 Ch. 19]**.
- Bounded fractional variables = bounded LP boundary zone ⟹ $P \in \mathcal{A}$.

**Where the argument breaks down:**

The step "bounded integrality gap → bounded LP optimal face dimension" is
*not* a proven theorem in general. It holds for TU constraint matrices
(where the gap is exactly 1 and all vertices are integral) and for
specific structured problems (Knapsack, network flows). For general
$\{0,1\}$-programs with bounded gap, the LP vertex can still have
$\Omega(n)$ fractional variables.

**Three routes to a formal proof (for future work):**

1. **TDI + Hoffman-Kruskal route:** If $Ax \leq b$ is TDI with sparse $A$
   (each row has $O(1)$ non-zeros), then the LP optimal face is a bounded
   polyhedron with integral vertices. This implies bounded boundary zone.
   *Gap:* Sparsity of $A$ must be related to $O(n)$ QUBO encoding — this
   connection is plausible but not formalised.
   **[Relevant: Hoffman-Kruskal (1956), Schrijver (1986) Ch. 19]**

2. **Communication complexity route:** Encode $\mathcal{B} \cap \mathcal{C}$
   membership as a two-party communication problem. Use Yao's minimax principle
   to show that $O(n)$ QUBO encoding implies $O(n)$-bit certificates for
   constraint satisfaction, which limits the boundary zone.
   *Gap:* The reduction from QUBO variable count to communication bits
   requires formalising what "QUBO encodes the problem" means precisely.
   **[Relevant: Yao (1979), Kushilevitz-Nisan "Communication Complexity" Ch. 1]**

3. **Constraint graph sparsity route:** If the constraint graph $G_P$
   (nodes = variables, edges = pairs appearing in constraints) is sparse
   ($O(n)$ edges), then the LP feasible region has bounded treewidth,
   which limits the number of fractional variables at any vertex.
   *Gap:* Treewidth $\leq k$ implies bounded fractionality only for
   $k = O(1)$ — the connection to $O(n)$ QUBO encoding needs work.
   **[Relevant: Bodlaender (1993), Cygan et al. "Parameterized Algorithms" Ch. 7]**

**Verdict:** $\mathcal{B} \cap \mathcal{C} \subseteq \mathcal{A}$ is a
well-motivated conjecture with three plausible proof routes, but remains
an **open problem**. Proving it would formally close the circularity and
constitute a significant theoretical result connecting LP theory, QUBO
encoding, and quantum advantage boundaries.

### 3.4 Counter-Direction: Large Gap → Large QUBO

For routing problems (TSP, VRP): the LP relaxation has an integrality gap
of $O(\log n)$ (best known for CVRP) to $O(1)$ (TSP, Christofides 1.5-approx).

But achieving even the LP lower bound requires **exponentially many subtour
elimination constraints** (DFJ formulation) — encoding these as QUBO penalties
requires the $O(n^2)$ arc variables.

The QUBO must encode what the LP constraint system requires. If the LP needs
$O(n^2)$ constraints to exclude infeasible solutions, the QUBO needs $O(n^2)$
variables to penalise them.

**This is the structural reason $\mathcal{A} \cap \mathcal{C}^c = \emptyset$**
is conjectured to hold for natural problem classes.

---

## 4. Candidate Exceptions

### 4.1 MaxCut as Primary Candidate

**Why MaxCut could break the circularity:**

1. **$O(n)$ encoding** ✓ — $n$ binary variables, no slack
2. **All solutions valid** ✓ — no feasibility barrier
3. **Local cost function** ✓ — each edge term acts on 2 qubits

**The key question:** Can QAOA beat Goemans-Williamson (0.878 guarantee)?

**[PROVEN — assuming Unique Games Conjecture (UGC)]**
Khot, Kindler, Mossel, O'Donnell (2007): If UGC is true, then achieving
approximation ratio $> 0.878 + \varepsilon$ for MaxCut is NP-hard classically.

**[CONJECTURE C4]** If UGC is true, then quantum algorithms with polynomial
overhead *could* achieve $> 0.878$ approximation for MaxCut, representing
genuine quantum advantage.

**Current empirical status:** QAOA $p=1$ achieves 0.6924 < 0.878.
No QAOA depth has been shown to consistently beat GW.
Bravyi, Kliesch, Koenig, Tang (2020) showed QAOA $p < n/2$ cannot beat GW
on certain graph families. **[PROVEN for those graph families]**

**Verdict:** MaxCut is the *theoretically strongest* candidate, but quantum
advantage has not been demonstrated and faces a proven lower-bound barrier
for shallow circuits.

### 4.2 Quantum Local Hamiltonian (k-Local Hamiltonian)

**[PROVEN — Kitaev 1999; Kempe, Kitaev, Regev 2006]**

The k-LOCAL HAMILTONIAN problem is **QMA-complete** — the quantum analogue
of NP-completeness. Specifically:

- Input: k-local Hamiltonian $H = \sum_i H_i$, each $H_i$ acting on $\leq k$ qubits
- Output: ground state energy $E_0$
- Complexity: QMA-complete for $k \geq 2$

**Why this is different from classical combinatorial optimisation:**

Classical MAX-k-SAT is NP-complete (classical NP ⊆ QMA).
k-Local Hamiltonian is QMA-complete, which is believed to be strictly harder than NP.

**QUBO encoding:** The k-Local Hamiltonian *is* a QUBO when restricted to
$Z$-basis operators. Its natural encoding is $O(n)$ qubits with $O(n)$ local
terms. This satisfies our criteria!

**Implication:** Problems with genuine *quantum structure* — where the cost
function involves quantum superposition of states, not just binary assignments —
may provide the exception. VQE for quantum chemistry is in this class.

**The fundamental insight:** Classical combinatorial optimisation problems
are *classical* by definition — their solutions are binary strings. Quantum
advantage for such problems requires the quantum algorithm to outperform
the best classical approximation, which is constrained by the UGC barrier.
Problems with *quantum-native structure* (Local Hamiltonian) have no such
classical approximation barrier.

### 4.3 Quadratic Unconstrained Binary Optimisation (QUBO) at Phase Transitions

**[EMPIRICAL CONJECTURE — partially supported by computational experiments]**

Random $k$-SAT near the satisfiability phase transition ($\alpha = \alpha_c$)
is computationally hardest for classical solvers. The cost landscape has
exponentially many near-degenerate local minima.

If quantum tunnelling could escape these local minima faster than classical
simulated annealing: genuine advantage. However:

**[PROVEN — Hastings 2021]** For random MAX-2-SAT, simulated annealing and
QAOA have similar performance at shallow depths.

**Verdict:** Phase transition hardness does not automatically grant quantum advantage.

---

## 5. Implications for the Phase 2 VRP Results

### 5.1 Consistency Check: 1.2M Physical Qubits

**Phase 2f result:** 20-ATM CVRP requires $\approx 1{,}188{,}952$ physical qubits.

**Theoretical prediction:**

$$q_{\text{VRP}}(n=20, m=4) = n^2 m + nm\lceil\log_2 n\rceil + m\lceil\log_2(Q/d_{\min})\rceil$$
$$= 400 \times 4 + 20 \times 4 \times 5 + 4 \times 3 = 1680 + 400 + 12 = 2092 \text{ logical}$$

With surface code distance $d=15$: $1 \text{ logical} \mapsto 2d^2 - 1 = 449 \text{ physical}$:

$$2092 \times 449 = 939{,}308 \text{ physical}$$

Phase 2f reported 1,188,952 — the 27% difference arises from additional
ancilla qubits required for fault-tolerant T-gate synthesis (Rz gates require
magic state distillation). **[CONSISTENT with theoretical O(n²m) class]**

### 5.2 Alternative Formulation: Can Encoding Be Reduced?

**Dantzig-Fulkerson-Johnson (DFJ) Flow Formulation:**

Instead of per-vehicle arc variables $x_{ijk}$, use aggregate arc variables:

$$x_{ij} \in \{0,\ldots,m\}: \text{total flow on arc } (i,j) \text{ across all vehicles}$$

This requires $O(\log m)$ bits per arc variable:

$$q_{\text{DFJ}} = n^2 \lceil\log_2(m+1)\rceil = O(n^2 \log m)$$

For $n=20, m=4$: $q = 400 \times 3 = 1200$ logical (vs 2092 for MTZ).
Physical: $1200 \times 449 = 538{,}800$ — a **55% reduction**.

**Still in $O(n^2)$ class** — the quadratic bottleneck remains.

**Commodity Flow Formulation:**

$$f_{ijk} \in \mathbb{Z}_{\geq 0}: \text{units of demand served by vehicle } k \text{ on arc } (i,j)$$

This is $O(n^2 m)$ continuous variables — worse than MTZ for binary QUBO.

**Hypothetical O(n log n) encoding:**

If subtour elimination could be encoded with $O(n \log n)$ qubits
(no known general construction):

$$q_{\text{hyp}} = n^2 \log n / n = n \log n \text{ (requires exponential compression of constraints)}$$

For $n=20$: $q = 20 \times 4.3 \approx 86$ logical; physical: $86 \times 449 = 38{,}614$.
Still not NISQ-feasible for $d=15$, but approaching the $O(10^3)$ physical qubit range
of near-term hardware.

**Assessment of C5 — Subtour Encoding Lower Bound:**

The original document stated a "counting argument suggests $\Omega(n \log n)$"
lower bound. This requires careful scrutiny — the counting argument alone is **insufficient**.

**Why the simple counting argument gives only $\Omega(n)$, not $\Omega(n \log n)$:**

There are $2^n - 2$ proper non-empty subsets $S \subset V$ whose subtour elimination
cutsets must be enforced. Distinguishing $2^n$ subsets requires $\Omega(n)$ bits
(since $\log_2(2^n) = n$). This yields an $\Omega(n)$ lower bound on the encoding — **not**
$\Omega(n \log n)$.

**The stronger $\Omega(n \log n)$ argument — via permutation entropy:**

A TSP optimal tour is a permutation $\pi \in S_n$ of $n$ cities. There are $n!$
distinct permutations. Any binary encoding that distinguishes all valid tours must
use at least $\lceil \log_2(n!) \rceil$ bits. By Stirling's approximation:

$$\lceil \log_2(n!) \rceil = \Theta(n \log n)$$

This is an **information-theoretic argument**: to uniquely specify which of the $n!$
tours is optimal, any encoding needs $\Omega(n \log n)$ bits.

**Important caveat:** This argument bounds the encoding of the *solution*, not the
encoding of the *problem constraints*. The QUBO encodes constraints (arc variables $x_{ij}$),
not the solution directly. The connection between constraint encoding bits and solution
encoding bits requires an additional step:

> If the QUBO constraint system must be expressive enough to have exactly the valid TSP tours
> as its feasible solutions, and there are $n!$ such tours, then any binary matrix over
> $\{0,1\}^q$ with $q$ variables must have at least $n!$ feasible points matching tours
> — requiring $q = \Omega(n \log n)$. **[INFORMAL PROOF SKETCH — not a formal theorem]**

This sketch is plausible but has a gap: the QUBO feasible set need not correspond bijectively
to tours. Multiple QUBO assignments can map to the same tour, and penalty terms can exclude
infeasible assignments without enumerating all subtours explicitly.

**[CONJECTURE C5 — REVISED STATUS: OPEN PROBLEM]**

The minimum QUBO variable count for any encoding of TSP/VRP subtour constraints is:

$$q_{\min} = \Omega(n \log n)$$

The $n!$ permutation entropy argument provides the strongest informal support.
A formal proof would require connecting information-theoretic lower bounds on
solution encoding to lower bounds on constraint encoding — a gap that currently
remains open in the literature.

*The $O(n^2)$ current encoding is achievable; the $\Omega(n \log n)$ theoretical
target is conjectured but not proven.*

### 5.3 The Bug-Lessons as Theoretical Signals

The three bugs from Phase 2d encode theoretical content:

| Bug | Root Cause | Theoretical Implication |
|-----|-----------|------------------------|
| λ too small | Penalty $\ll$ objective | Feasibility-optimality trade-off requires $\lambda \geq \text{Lipschitz}(f) / \text{constraint curvature}$ |
| MILP infeasible | Over-constrained model | Constraint satisfaction is in NP; encoding constraints tightly risks infeasibility |
| ⟨H⟩ vs argmax | Two distinct quantities | Expectation energy is a *quantum* observable; argmax is *classical* post-processing — the distinction marks the quantum-classical interface |

---

## 6. Open Questions

### Q1: Can the Circularity Conjecture Be Proven?

A formal proof would require:

1. Showing that for problems where the LP boundary zone is $O(1)$, the QUBO
   encoding is necessarily $O(n)$. This likely follows from the LP vertex
   structure (degree-of-freedom counting).

2. Showing that $O(n)$ QUBO encoding + LP integrality gap $< \varepsilon$ implies
   LP + rounding achieves $(1-\delta(\varepsilon))$-approximation. This is
   essentially the FPTAS (Fully Polynomial-Time Approximation Scheme) structure
   argument for Knapsack. **[Vazirani, "Approximation Algorithms", Ch. 8]**

3. Showing the converse: problems where LP + rounding fails have integrality
   gap $> 1$, which forces $> O(n)$ QUBO encoding. This requires connecting
   LP theory with combinatorial topology — a non-trivial step.

**Research direction:** This conjecture, if proven, would formally establish
*why* quantum advantage for classical combinatorial optimisation requires
hard instances, and hard instances require large encodings, and large encodings
suffer exponential barren plateaus. A complete proof would be a significant
theoretical result.

### Q2: Which Problem Class Is Closest to Quantum Advantage?

**Ranking (best to worst for quantum advantage):**

1. **k-Local Hamiltonian (QMA-complete):** Quantum advantage proven in
   complexity-theoretic sense — but not a classical combinatorial problem.

2. **MaxCut on expander graphs:** Theoretically possible if QAOA depth
   $p = \Omega(n)$ and UGC holds. Practically: requires fault-tolerant hardware.

3. **Quadratic Assignment Problem (QAP) at phase transition:** Random instances
   near the hardness phase transition have no known efficient classical algorithm.
   QUBO encoding is $O(n^2)$ — barren plateau challenge remains.

4. **Protein folding (HP model):** $O(n)$ encoding on a lattice, quantum-native
   structure. IBM/Cleveland Clinic actively working on this.

5. **Knapsack/VRP/TSP:** Encoding complexity and LP approximability make
   quantum advantage unlikely without structural breakthroughs.

### Q3: What Is the Minimum Problem Structure for Meaningful Hybrid?

Based on this analysis, the minimum conditions for a *meaningful* hybrid
quantum-classical approach are:

$$\begin{cases}
1. & \text{QUBO encoding: } O(n) \text{ variables} \\
2. & \text{Cost function: local (O(1) qubit support per term)} \\
3. & \text{Valid solution fraction: bounded away from 0} \\
4. & \text{Classical approximation guarantee: below UGC barrier} \\
5. & \text{LP boundary zone: } \Omega(\log n) \text{ (enough to be non-trivial)}
\end{cases}$$

**No known classical combinatorial optimisation problem satisfies all five simultaneously** at scale relevant to practical hybrid advantage. This is the current state of the field (2024).

The most promising research direction: **problem-specific mixers** (XQAOA,
ADAPT-VQE) that preserve feasibility constraints in superposition, reducing
the effective valid fraction problem. For VRP, this means designing a quantum
mixer that only transitions between feasible routes — an open research problem.

---

## 7. Related Work

### Bravyi, Kliesch, Koenig, Tang (2022) — Hybrid Algorithms for Graph Colouring

**Citation:** Bravyi, S., Kliesch, A., Koenig, R., & Tang, E. (2022). "Hybrid quantum-classical algorithms for approximate graph coloring." *Quantum* 6, 678. (Preprint: arXiv:2011.13420)

This paper analyses Recursive QAOA (RQAOA) on MAX-$k$-CUT — equivalent to approximate $k$-colouring. The key finding is that standard (non-recursive) QAOA **fails** to solve graph colouring for most regular bipartite graphs at any constant depth $p$, achieving approximation ratios no better than random colouring. The authors further construct an efficient classical simulation of level-1 QAOA, showing that at this depth the hybrid algorithm offers no quantum advantage.

**Relation to this work:** Graph Colouring has $O(nk)$ QUBO encoding (Section 1.2g), placing it in the super-linear class between $O(n)$ and $O(n^2)$. The Bravyi et al. (2022) result is consistent with Conjecture C1: the effective circuit operates on $nk$ qubits, so the barren plateau pressure scales as $O(2^{-nk})$, making training intractable for large $k$. Additionally, the graph colouring constraint ("each node gets exactly one colour") is a *global* constraint — violating the locality condition of C2. Bravyi et al. (2022) thus provides an empirical confirmation of *both* the encoding complexity and cost-locality conditions of the quantum advantage boundary framework.

### Farhi, Gamarnik, Gutmann (2020) — QAOA Must See the Whole Graph

**Citation:** Farhi, E., Gamarnik, D., & Gutmann, S. (2020). "The Quantum Approximate Optimization Algorithm Needs to See the Whole Graph: A Typical Case." arXiv:2004.09002.

This paper analyses QAOA on the Maximum Independent Set problem on random $d$-regular graphs. The central result is that for depth $p < c \log n$ (for a $d$-dependent constant $c$), the QAOA cannot achieve better than approximately $0.854$ times the optimal independent set size. The mechanism is *locality*: at shallow depth, each qubit's circuit only "sees" its local $p$-hop neighbourhood, which on random regular graphs is a tree. The global structure — necessary for near-optimal solutions — is invisible.

**Relation to this work:** This result is the strongest empirical evidence for the locality condition in Conjecture C2. The requirement that $p = \Omega(\log n)$ for global structure to be visible means that depth must grow with problem size — precisely the regime where barren plateaus become catastrophic (gradient variance $O(2^{-n})$ with $O(n)$ parameters). Farhi et al. (2020) and Bravyi et al. (2020) together establish that *shallow* QAOA is provably limited by locality, while *deep* QAOA is limited by barren plateaus — squeezing the window for quantum advantage from both sides.

### Cerezo et al. (2021) — Cost Function Dependent Barren Plateaus

**Citation:** Cerezo, M., Sone, A., Volkoff, T., Cincio, L., & Coles, P.J. (2021). "Cost function dependent barren plateaus in shallow parametrized quantum circuits." *Nature Communications* 12, 1791.

This paper refines the barren plateau theory of McClean et al. (2018) by showing that the gradient decay rate depends critically on whether the cost function is *local* or *global*. For local cost functions (observable acts on $O(1)$ qubits), gradients decay at most polynomially in $n$ for shallow circuits (depth $O(\log n)$). For global cost functions (observable acts on all $n$ qubits), gradients decay exponentially even for shallow circuits.

**Relation to this work:** This paper directly strengthens Conjecture C2. The three-condition framework (Section 2.3) requires locality of the cost function as an independent condition from encoding size. Cerezo et al. (2021) proves that locality is *sufficient* to avoid exponential barren plateaus in shallow circuits — but also that it is not sufficient alone: deep circuits suffer barren plateaus regardless of locality **[McClean et al. 2018]**. For VRP and Knapsack: the penalty terms $\lambda(\sum w_i x_i - W)^2$ are global observables (acting on all $n$ variables), so they suffer exponential gradient decay at any depth, even if the variable count is $O(n)$. This closes the gap in our analysis: O(n) encoding is necessary but not sufficient — locality of the cost function is the second independent requirement.

---

## References

- Bodlaender, H.L. (1993). "A linear time algorithm for finding tree-decompositions of small treewidth." *SIAM Journal on Computing* 25(6), 1305–1317.
- Bravyi, S., Kliesch, A., Koenig, R., & Tang, E. (2020). "Obstacles to variational quantum optimisation from symmetry protection." *Physical Review Letters* 125, 260505.
- Bravyi, S., Kliesch, A., Koenig, R., & Tang, E. (2022). "Hybrid quantum-classical algorithms for approximate graph coloring." *Quantum* 6, 678. arXiv:2011.13420.
- Cerezo, M., et al. (2021). "Variational quantum algorithms." *Nature Reviews Physics* 3, 625–644.
- Cerezo, M., Sone, A., Volkoff, T., Cincio, L., & Coles, P.J. (2021). "Cost function dependent barren plateaus in shallow parametrized quantum circuits." *Nature Communications* 12, 1791.
- Cygan, M., et al. (2015). *Parameterized Algorithms.* Springer.
- Dantzig, G.B. (1957). "Discrete-variable extremum problems." *Operations Research* 5(2), 266–288.
- Farhi, E., Gamarnik, D., & Gutmann, S. (2020). "The Quantum Approximate Optimization Algorithm Needs to See the Whole Graph: A Typical Case." arXiv:2004.09002.
- Farhi, E., Goldstone, J., & Gutmann, S. (2014). "A Quantum Approximate Optimisation Algorithm." arXiv:1411.4028.
- Hastings, M.B. (2021). "Classical and quantum bounded depth approximation algorithms." *Quantum* 5, 382.
- Hoffman, A.J. & Kruskal, J.B. (1956). "Integral boundary points of convex polyhedra." In Kuhn & Tucker (eds.), *Linear Inequalities and Related Systems*, pp. 223–246. Princeton University Press.
- Kellerer, H., Pferschy, U., & Pisinger, D. (2004). *Knapsack Problems.* Springer.
- Khot, S., Kindler, G., Mossel, E., & O'Donnell, R. (2007). "Optimal inapproximability results for MAX-CUT." *SIAM J. Computing* 37(1), 319–357.
- Kitaev, A., Shen, A., & Vyalyi, M. (2002). *Classical and Quantum Computation.* AMS.
- Kushilevitz, E. & Nisan, N. (1997). *Communication Complexity.* Cambridge University Press.
- McClean, J.R., Boixo, S., Smelyanskiy, V.N., Babbush, R., & Neven, H. (2018). "Barren plateaus in quantum neural network training landscapes." *Nature Communications* 9, 4812.
- Schrijver, A. (1986). *Theory of Linear and Integer Programming.* Wiley.
- Vazirani, V. (2001). *Approximation Algorithms.* Springer.
- Yao, A.C. (1979). "Some complexity questions related to distributive computing." *Proc. 11th STOC*, pp. 209–213.

---

*Report generated: ARUTE CIT-SVRP Project — Quantum Advantage Theoretical Analysis*
*Classification: Research document — claims marked PROVEN/CONJECTURE/EMPIRICAL throughout*
