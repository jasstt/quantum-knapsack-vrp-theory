# Quantum Advantage Boundary: Knapsack Hybrid & VRP Theory

> **A rigorous, honest investigation into where hybrid quantum-classical optimisation
> actually works — and where it cannot.**

This project is the theoretical extension of
[stochastic-VRP-decision-focused-learning](https://github.com/jasstt/stochastic-VRP-decision-focused-learning),
built on three empirical bugs discovered during that project.

---

## Research Lineage

```
stochastic-VRP-decision-focused-learning  (Phase 2d — three bugs)
        │
        │  Bug 1: λ too small → QAOA ignores capacity constraint
        │  Bug 2: MILP silently returns INFEASIBLE
        │  Bug 3: ⟨H⟩ expectation ≠ argmax-bit (two different metrics)
        │
        ▼
quantum-knapsack-vrp-theory  (this repo)
        │
        ├── Apply all three lessons from day one
        ├── Test hybrid approach on Knapsack (LP boundary = O(1))
        ├── Measure barren plateau gradient variance empirically
        └── Formalise why VRP encoding O(n²m) makes quantum advantage
            theoretically inaccessible at practical scale
```

### Intersection Point

Both projects converge at a single question:

> *What is the minimum problem structure for which quantum advantage
> is theoretically possible?*

The VRP project answers *"not here"* empirically (1.2M physical qubits, barren
plateau, encoding overhead). This project answers *"then where?"* theoretically.

---

## Project Highlights

| Result | Value |
|--------|-------|
| Barren plateau slope (measured) | **−0.986** (theory: −1.000, Δ = 1.4%) |
| Pearson r(n, log₂Var) | **−0.965**, p = 0.002 |
| VRP QUBO growth rate | **O(n^1.87) ≈ O(n²)** confirmed |
| MaxCut growth rate | **O(n^1.00) = O(n)** confirmed |
| LP boundary size — Knapsack | **O(1)** — at most 1 fractional variable |
| LP boundary size — CVRP | **O(n²)** — pairwise routing coupling |

---

## Repository Structure

```
quantum-knapsack-vrp-theory/
│
├── knapsack/
│   ├── knapsack_classical.py        # Task 1 — DP / LP-Relaxation / Greedy
│   ├── knapsack_qubo.py             # Task 2 — QUBO + λ validation [L1]
│   ├── knapsack_qaoa.qs             # Task 3 — Q# QAOA circuit (p layers)
│   ├── knapsack_hybrid.py           # Task 3 — NumPy statevector QAOA simulator
│   ├── knapsack_comparison.py       # Task 5 — Full comparison table
│   ├── knapsack_scaling.py          # Task 6 — Scaling analysis n=10→100
│   │
│   ├── encoding_complexity.py       # Theory Task 1 — QUBO qubit counts, 7 problems
│   ├── barren_plateau_analysis.py   # Theory Task 2 — Gradient variance vs n
│   └── knapsack_theory.md           # Theory report — 6 sections, 5 conjectures
│
└── README.md
```

---

## Key Results

### 1. Barren Plateau — Empirically Validated

McClean et al. (2018) predict `Var[∂⟨H⟩/∂γ] ∝ O(2⁻ⁿ)`.
Measured on MaxCut QAOA p=1 (normalized QUBO, 300–30 random initialisations per n):

| n | Measured Var | log₂(Var) | Theoretical |
|---|------------|----------|-------------|
| 4 | 0.01222 | −6.35 | −4.00 |
| 5 | 0.00182 | −9.10 | −5.00 |
| 6 | 0.00142 | −9.46 | −6.00 |
| 7 | 0.00093 | −10.07 | −7.00 |
| 8 | 0.00047 | −11.05 | −8.00 |
| 10 | 0.00012 | −13.00 | −10.00 |

**Fitted slope: −0.986** (theory: −1.000). Strong exponential decay confirmed.

> The offset between measured and theoretical (−3.27 vs 0) is expected:
> the proportionality constant C depends on the specific circuit structure
> and is not universal.

### 2. QUBO Encoding Complexity Classes

| Problem | QUBO Variables | Growth Rate | NISQ Feasible? |
|---------|---------------|-------------|----------------|
| MaxCut | n | **O(n)** | ✅ n ≤ 50 |
| Portfolio (sparse) | n | **O(n)** | ✅ n ≤ 50 |
| Knapsack | n + log W | **O(n)** | ✅ n ≤ 50 |
| Graph Colouring | nk | O(nk) | ⚠️ small k |
| TSP | n² | **O(n²)** | ❌ n > 7 |
| VRP (m vehicles) | n²m | **O(n²m)** | ❌ n > 5 |

### 3. The Circularity Conjecture (C3)

> Problems where the LP boundary zone is O(1) ≡ Problems where QUBO
> encoding is O(n) ≡ Problems where classical LP + rounding already works well.

This is not coincidence — it follows from the LP integrality structure
(TDI / total unimodularity). **[Conjecture — not yet formally proven]**

Full theoretical argument: [`knapsack/knapsack_theory.md`](knapsack/knapsack_theory.md)

---

## Three Lessons from Phase 2 (Applied from Day One)

These bugs were discovered in the VRP project and prevented here by design:

| Lesson | Rule | Implementation |
|--------|------|----------------|
| **[L1]** λ validation | `λ ≥ v_max / w_min² × 3×` safety margin | `knapsack_qubo.py` — validated before any QUBO run |
| **[L2]** No silent INFEASIBLE | Every solution explicitly checked against constraints | `knapsack_hybrid.py` — QAOA result `[1]` caught, corrected to `[0]` |
| **[L3]** ⟨H⟩ ≠ argmax | Two metrics always reported separately | All output tables distinguish `⟨H⟩` from argmax-bit energy |

---

## Running the Code

```bash
# Classical baseline (DP, LP, Greedy)
python knapsack/knapsack_classical.py

# QUBO formulation + λ validation
python knapsack/knapsack_qubo.py

# QAOA hybrid simulation (p = 1, 2, 3)
python knapsack/knapsack_hybrid.py

# Full comparison table
python knapsack/knapsack_comparison.py

# Scaling analysis n = 10, 20, 50, 100
python knapsack/knapsack_scaling.py

# Theory Task 1 — Encoding complexity for 7 problem types
python knapsack/encoding_complexity.py

# Theory Task 2 — Barren plateau gradient variance measurement
python knapsack/barren_plateau_analysis.py
```

**Dependencies:** `numpy`, `scipy`, `pandas`, `matplotlib`, `pulp`

---

## Theoretical Framework

Five conjectures are developed in [`knapsack_theory.md`](knapsack/knapsack_theory.md):

- **C1:** O(n²) QUBO encoding → gradient variance `O(2⁻ⁿ²)` → barren plateau unavoidable
- **C2:** O(n) encoding + local cost function → barren plateau potentially avoidable
- **C3:** Circularity Theorem — LP boundary O(1) ↔ QUBO O(n) ↔ classical LP easy (structural, not coincidence)
- **C4:** MaxCut is the strongest quantum advantage candidate (connection to Unique Games Conjecture)
- **C5:** VRP/TSP subtour encoding has `Ω(n log n)` lower bound

Claims are explicitly marked **[PROVEN]**, **[CONJECTURE]**, or **[EMPIRICAL]** throughout.

---

## Related Work

**Predecessor project:**
[stochastic-VRP-decision-focused-learning](https://github.com/jasstt/stochastic-VRP-decision-focused-learning)
— Cash-in-Transit SVRP with SPO+ decision-focused learning.
The quantum phases (2b–2f) of that project provide the empirical VRP data
that grounds this theoretical analysis.

**Key papers referenced:**
- McClean et al. (2018) — Barren plateaus in quantum neural networks
- Cerezo et al. (2021) — Variational quantum algorithms (Nature Reviews)
- Farhi, Goldstone, Gutmann (2014) — QAOA original paper
- Khot et al. (2007) — MaxCut inapproximability and Unique Games Conjecture
- Schrijver (1986) — TDI and total unimodularity

---

## Conclusion

> Classical combinatorial optimisation problems are *classical by definition* —
> their solutions are binary strings. Quantum advantage for such problems requires
> the algorithm to outperform the best classical approximation scheme.
> For problems where encoding complexity is O(n²) or higher, barren plateau
> gradient suppression is `O(2⁻ⁿ²)` — numerically zero for any practical n.
> The problems where quantum circuits are small enough to avoid this barrier
> are exactly the problems where classical LP already finds near-optimal solutions.
> This circularity is structural, not a hardware limitation, and will not be
> resolved by better quantum chips alone.

*This project does not claim quantum computing is useless —
it claims the right problem class has not yet been found for
gate-based QAOA on classical combinatorial optimisation.*
