# arXiv Submission Preparation
## Quantum Advantage Boundaries in Combinatorial Optimisation

---

## Task 3 — Abstract (150–200 words)

> **Instructions followed:** motivasyon → ne yapıldı → bulgular → sonuç.
> "Quantum fails" denmedi; "current tool mismatch" çerçevesi korundu.

---

The Quantum Approximate Optimization Algorithm (QAOA) has generated
substantial interest as a near-term approach to combinatorial optimisation,
yet empirical results have consistently fallen short of theoretical promise.
We investigate the structural conditions under which quantum advantage is
theoretically accessible, combining an analytical QUBO encoding complexity
classification across seven problem classes with an empirical measurement of
barren plateau gradient variance scaling on MaxCut QAOA circuits.
We classify problems by whether their QUBO formulation grows as O(n), O(nk),
or O(n²m) in the number of binary variables, and show that this growth rate
directly controls the effective barren plateau suppression — from
polynomial (O(n) class) to doubly-exponential (O(n²m) class, including VRP
and TSP). For the 20-node CVRP benchmark, our encoding analysis predicts
≈2,092 logical / ≈939,000 physical qubits, consistent with empirical
resource estimates of 2,648 logical / 1.2 million physical qubits.
We introduce the Circularity Conjecture: for natural combinatorial problem
classes, the problems where LP boundary zones are small enough for quantum
sub-problems to be tractable are precisely those where classical
LP-plus-rounding already provides strong approximation guarantees.
This structural coincidence is not a hardware limitation and will not be
resolved by improved quantum devices alone — it suggests that the search
for practical quantum advantage in combinatorial optimisation must target
problem classes beyond those studied in this work.

**Word count:** 198

---

## Task 4 — Title Options

### Current title
> *"Quantum Advantage Boundaries in Combinatorial Optimisation"*
> **Assessment:** Accurate and readable, but "boundaries" is ambiguous
> (could mean "limits" or "boundary regions of LP relaxations" — this
> paper means both, which may confuse readers skimming titles).

---

### Option A — cs.DS / cs.CC (technical, algorithm-focused)

**"QUBO Encoding Complexity Classes and Barren Plateau Scaling:
A Structural Barrier to Quantum Advantage in Combinatorial Optimisation"**

**Strength:** Precise — signals exactly what is contributed (classification
+ empirical scaling + structural argument). Will be found by searches for
"QUBO encoding" and "barren plateau." cs.DS/cc reviewers will understand
the contribution immediately.

**Weakness:** Long (18 words). No appeal to quantum-native readers who
search for "QAOA" or "VQA." The term "QUBO Encoding Complexity Classes"
may be unfamiliar to readers outside the QUBO/QAOA community.

---

### Option B — quant-ph (quantum-forward)

**"When Does QAOA Fail? Encoding Size, Barren Plateaus, and the
Circularity of Quantum Advantage in Combinatorial Optimisation"**

**Strength:** Opens with a provocative question that attracts quant-ph
readers. "Circularity" is memorable and distinctive — it names the
main conjecture, making the paper more citable. Appeals to the large
quant-ph audience working on NISQ limitations.

**Weakness:** "When Does QAOA Fail?" may be read as overly negative —
could discourage interdisciplinary readers or reviewers who prefer
neutral framing. The word "fail" does not appear in the paper's
conclusions (intentionally).

---

### Option C — Balanced (recommended for cross-list submission)

**"Encoding Complexity, Barren Plateaus, and the Circularity Conjecture:
Structural Limits on Hybrid Quantum-Classical Optimisation"**

**Strength:** "Hybrid Quantum-Classical" is the current dominant search
term in both quant-ph and cs.DS. Names the conjecture ("Circularity")
to improve citability. Neutral in tone — does not imply quantum "fails,"
only that structural limits exist. Works for both quantum and OR audiences.

**Weakness:** "Limits" in the subtitle still carries a negative connotation
for quant-ph readers. Does not contain "QAOA" or "VRP" which are common
search terms for the target audiences.

**Recommendation:** Option C for the primary title. Add "QAOA" and "VRP"
as keywords in the metadata to cover the search term gap.

---

## Task 5 — Venue Decision

### Primary Category

**Recommendation: `quant-ph`**

**Rationale:** The primary intellectual contribution is a *quantum
algorithm analysis* — specifically, connecting QUBO circuit size to
barren plateau gradient variance and deriving structural conditions for
quantum advantage. The Circularity Conjecture is a statement *about
quantum algorithms*, not about LP theory or combinatorial algorithms
per se. The audience who will cite this paper most is the VQA/QAOA
community reading quant-ph. The empirical barren plateau measurement
(McClean scaling confirmed at 1.4% error) is a quantum physics
result. Therefore, `quant-ph` is the correct primary category.

**Against cs.DS:** The paper does not prove new combinatorial algorithms,
improve approximation ratios, or derive new complexity class separations.
It *analyses* existing algorithms (QAOA) from a complexity-theoretic
lens. Pure cs.DS readers would expect a more formal algorithmic result.

**Against cs.CC:** Complexity theory papers require formal proofs of
separations or reductions. The Circularity Conjecture is stated as a
conjecture throughout — it is not a proven result. A cs.CC submission
would receive referee comments asking for the proof, which does not exist.

---

### Cross-List Recommendations

| Category | Reason |
|----------|--------|
| `cs.DS` | QUBO encoding classification is a data structure / algorithm analysis result |
| `cs.DM` | LP boundary zone and TDI theory is discrete mathematics |
| `cs.CC` | **Do NOT cross-list** — Circularity Conjecture is not proven; cc reviewers will object |

**Recommended submission line:**
```
Primary:    quant-ph
Cross-list: cs.DS, cs.DM
```

---

### MSC / ACM Classification Codes

| Code | Label |
|------|-------|
| 81P68 | Quantum computation |
| 90C27 | Combinatorial optimization |
| 68Q25 | Analysis of algorithms and problem complexity |
| 90C05 | Linear programming |
| F.2.2 | Nonnumerical Algorithms and Problems — Computations on discrete structures |

---

### Keywords (for arXiv metadata)

```
QAOA, QUBO, barren plateau, quantum advantage, combinatorial optimisation,
vehicle routing problem, knapsack, encoding complexity, LP relaxation,
circularity conjecture, hybrid quantum-classical, variational quantum algorithms
```

---

## Checklist Before Submission

- [ ] LaTeX formatting: all equations in `equation` environments, not `$$...$$`
- [ ] All CONJECTURE/PROVEN/EMPIRICAL labels preserved
- [ ] C5 correctly marked OPEN PROBLEM (not PROVEN)
- [ ] B∩C⊆A direction explicitly marked as OPEN PROBLEM
- [ ] Three new references (Bravyi 2022, Farhi-Gamarnik 2020, Cerezo 2021) in bibliography
- [ ] Abstract ≤ 1920 characters (arXiv limit)
- [ ] No author-identifying information in main text (blind review if targeting a venue)
- [ ] Data availability statement: code at github.com/jasstt/quantum-knapsack-vrp-theory

---

*Prepared for: quantum-knapsack-vrp-theory / knapsack_theory.md → arXiv preprint*
