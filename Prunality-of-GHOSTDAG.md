# Prunality of the GHOSTDAG Protocol

**Authors:** Michael Sutton, Shai Wyborski  
**Date:** July 2020

*This is an unofficial translation for readability. Any errors are the translator's responsibility.*

═══════════════════

## Introduction

This document details a set of rules miners in the GHOSTDAG protocol must follow in order to support secure pruning of the DAG.

═══════════════════

## Background

Before diving into pruning rules, here are essential concepts from GHOSTDAG/PHANTOM:

**The DAG:** Blocks form a directed acyclic graph — each block has parent pointers to one or more existing **tips** (blocks with no children yet, representing the frontier of the DAG). Following parent pointers backward always leads to Genesis (the first block) and never creates cycles.

**Weight:** Each block has weight = 1 + sum of weights of all its **blue ancestors**. Only blue blocks contribute to weight calculations; red blocks are ignored for this purpose. This measures how much "work" backs the block through valid consensus paths.

**Past/Future/Chain:**
- **B.Past** = all ancestors of B (blocks reachable by following parent pointers backward)
- **B.Future** = all descendants of B (blocks that have B as an ancestor)
- **B.Chain** = the canonical path from B back to Genesis, formed by repeatedly following Selected Parents

**SelectedParent:** When a block has multiple parents, GHOSTDAG/PHANTOM selects exactly one as the "Selected Parent" — this forms the canonical chain used for ordering and consensus.

**Blue Blocks:** GHOSTDAG/PHANTOM colors each block blue or red based on how well-connected it is to the canonical chain. Blue blocks are those best connected; they form the backbone of consensus. *(Full coloring rules are in the GHOSTDAG paper; we only need to know that blue blocks obey certain bounds.)*

═══════════════════

## Notation and Rules

### Parameters:
- **phi (φ)** -- Finality Depth
- **k** -- PHANTOM Parameter
- **ell (ℓ)** -- MergeSet Size Limit
- **pi (π) = 2×φ + 4×ℓ×k + 2×k + 2** -- Pruning Depth

### Definitions:

Some notations:
- **InclusivePast, InclusiveFuture, InclusiveChain** are the inclusive counterparts of Past, Future, Chain
- **B.Subtree** = all blocks C where, following Selected Parent pointers backward from C to Genesis,
  you encounter B on that path  
  *(equivalently: all blocks that have B as their ancestor via the canonical chain)*
- **B.MergeSet** = B.Past minus B.SelectedParent.InclusivePast  
- **B.Anticone** = all blocks that are neither ancestors nor descendants of B
  *(blocks on unrelated forks — not reachable from B and cannot reach B)*
- **B.Blues** = all blue blocks that are ancestors of B (reachable by walking backward from B)
- A block B is said to be a *merging block* of block Y, if Y is in B.MergeSet.

═══════════════════

### Definition (Blue Depth):

For any integer n and any block B let:

**B_n** = the element C in B.Chain that maximizes weight(C) subject to weight(C) < weight(B) - n

*Equivalently: walk backward from B along its canonical chain, and stop at the first block whose weight drops below weight(B) minus n.*

We name some useful blocks:
- The block at *depth* n is **Virtual_n**
- The *finality block* is **F = Virtual_Finality**
- The *pruning block* is **P = Virtual_Pruning**

═══════════════════

### Corollary (Depth Bounds):

For any integer n and any block B:

weight(B) - n - k - 1 ≤ weight(B_n) < weight(B) - n

*Interpreted as distance: B_n is at least n weight units back from B, but no more than n + k + 1 weight units back. The "uncertainty window" of at most k+1 comes from the fact that each step along the canonical chain can add at most k+1 blue blocks.*

*Proof:* The second inequality follows directly from Definition of blue depth. The first follows from maximality of B_n. Assume weight(B_n) < weight(B) - n - k - 1. Since each chain block can add at most k+1 blue blocks, there must exist a block C in B.Chain where C.SelectedParent = B_n, and which satisfies weight(C) < weight(B) - n. Contradicting maximality of B_n.

═══════════════════

### Corollary (Depth Relation):

For any block B and any integers m, n such that m > n:

weight(B_m) < weight(B_n) + n + k + 1 - m

*Equivalently: B_m is always further back than B_n: by at least how much deeper you went (m-n), and at most that amount plus k+1.*

*Proof:* This is a direct result of applying the Depth Bounds corollary over B, m and obtaining weight(B_m) < weight(B) - m, applying the same corollary over B, n and obtaining weight(B) - n - k - 1 ≤ weight(B_n), and combining the inequalities.

═══════════════════

### Definition (Pruning Invalidation Rules):

A block B is considered invalid if one of the following holds:

> **(R-I) Objective finality violation:**  
> There exists a block Y in B.MergeSet that is also in the Anticone of B_Finality, AND there is no blue block that simultaneously:
> 1. is a descendant of Y (in Y.Future),
> 2. is a blue ancestor of B (in B.Blues), and
> 3. descends from B_Finality via Selected Parent chain (in B_Finality.Subtree).  
>   
> *Interpretation:* Block Y has not been "kosherized" — no blue block bridges the fork containing Y back to the canonical chain anchored at finality depth.

> **(R-II) Bounded merge:**  
> The size of B.MergeSet exceeds the Mergeset Size Limit (ℓ).

> **(R-III) Blood exile:**  
> At least one parent block in B.Parents is invalid.

═══════════════════

### Assumption (Bounded Reorganization):

There is no 51% attacker and Finality Depth was chosen so that (malicious or organic) splits of Finality Depth have negligible probability. We consider them as impossible.

---

## Main Proposition

### Proposition (Pruning):

If when block B was discovered it held that **B is not in P.Future**, then B will never be in the past of Virtual.

*Summary: A block found too far from the main chain at discovery time can never become part of the canonical history. The proof shows this by contradiction — assuming such a block enters Virtual.Past forces either the MergeSet to exceed ℓ (violating R-II) or two Anticone blocks onto the same chain (logical impossibility).*

*Proof:* Let Y be a block which, when it was discovered, was not in P.Future. Assume that at some future point in time Y was in Virtual.Past. 

Let X be a block in Virtual.Past with the following properties:
- F is in X.Chain  
- Y is in X.Past  
- X is *minimal* (no other block satisfying these conditions exists "earlier" than X)

Such an X must exist because we assumed Y eventually enters Virtual.Past, and by Assumption (Bounded Reorganization), F must be reachable from some ancestor of Y.

We now define a sequence of merging blocks **X = X⁰, X¹, ..., Xᵐ** as follows:

**Induction hypothesis:** If Y is in Xⁱ⁻¹.MergeSet, then rule (R-I) requires a blue block bridging Y to Xⁱ⁻¹_Finality — otherwise Xⁱ⁻¹ would be invalid.

**Step 1 (Find Bⁱ):** At iteration i, starting from Xⁱ⁻¹, find a blue block (which we call Bⁱ) that satisfies three conditions:
- Bⁱ is a descendant of Y (Y is an ancestor of Bⁱ)
- Bⁱ is a blue ancestor of Xⁱ⁻¹ (in Xⁱ⁻¹.Blues)  
- Bⁱ descends from Xⁱ⁻¹_Finality via Selected Parent chain

Such a block must exist by the induction hypothesis.

**Step 2 (Find Xⁱ):** Look for a merging block Xⁱ in Bⁱ's InclusiveChain — that is, a block where Y appears in Xⁱ's MergeSet. If such a block exists, it is unique.

**Step 3 (Halt condition):** If no such merging block exists, stop the process with m = i-1. This happens only when Y itself lies on Bⁱ's canonical chain.

***

**Counting Measures:** We now define two quantities to track progress:

**wᵢ (freeloader distance):** How far Bⁱ sits from Xⁱ⁻¹'s main chain — specifically, the weight difference between Xⁱ⁻¹ and the heaviest block that is both on Xⁱ⁻¹'s canonical chain AND an ancestor of Bⁱ.

*(Intuition: wᵢ = 0 means Bⁱ IS on Xⁱ⁻¹'s main chain; larger values mean Bⁱ branches further off to a side fork.)*

**δᵢ (extra blocks):** The number of blocks on Bⁱ's canonical chain that are NOT also on Xⁱ's canonical chain.

*(Intuition: δᵢ measures how much the "main chain" shifted between iterations — zero means no shift, positive values mean we moved to a different fork.)*

Note: w₀ and δ₀ are undefined, and δᵢ = 0 exactly when Bⁱ equals Xⁱ.

***

**Claim 1 (Freeloader Distance):**

For all i = 1, 2, ..., m: **wᵢ ≤ 4k + 1**

*(Intuition: Blue blocks can't be arbitrarily far off the main chain. The GHOSTDAG/PHANTOM selection rules ensure any blue block is within 4k+1 weight units of the canonical chain — this loose bound is a fundamental security property of the protocol.)*

*Proof:* This follows from the fact that Bⁱ is blue and from an argument similar to GHOSTDAG's freeloader bound (Bⁱ being a block freeloaded by Xⁱ⁻¹).

***

**Claim 2 (Mergeset Limit):**

**m + δ₁ + δ₂ + ... + δₘ < ℓ**

*(Intuition: Each iteration contributes at least one unique block to the MergeSet — either a merging block or extra blocks from chain shifts. Since rule (R-II) caps the MergeSet size at ℓ, the process cannot continue indefinitely. This bound forces termination after finitely many steps.)*

*Proof:* Let Δᵢ denote the set of blocks in Bⁱ's InclusiveChain that are NOT in Xⁱ's Chain. We need to show that for all i = 1, 2, ..., m, each set Δᵢ is a disjoint subset of X.MergeSet (excluding Y itself), which has size less than ℓ by rule (R-II).

**The subset property:** All blocks in Δᵢ are both descendants of Y AND ancestors of X. If there were a block B that is both a descendant of Y and an ancestor of X, but NOT in X.MergeSet, then both B and Y would be in X.SelectedParent.Past — contradicting the minimality of X.

**The disjoint property:** Each Δᵢ contains different blocks because of how we select each Bⁱ inductively.

Since |Δᵢ| = δᵢ + 1, summing over all i gives the desired result.

***

**Claim 3 (Score Distance):**

**weight(Xᵐ) > weight(F) - 4×ℓ×k**

*(Intuition: Each step of the process can only move us back a limited amount — at most wᵢ + δᵢ(k+1). Summing over all steps using Claims 1 and 2, the total "drift" backward is bounded by 4×ℓ×k. This means Xᵐ cannot be arbitrarily far from F in weight terms — it stays within a predictable window.)*

*Proof:* By definition of wᵢ we have that **weight(Xⁱ⁻¹) - wᵢ < weight(Bⁱ)**. Additionally, since Xⁱ is in Bⁱ's InclusiveChain and each chain block can add at most k+1 blue blocks, we get that **weight(Bⁱ) ≤ weight(Xⁱ) + δᵢ × (k+1)**.

Reorganizing terms gives us a bound on the score distance between two consecutive merging blocks:

**weight(Xⁱ⁻¹) - weight(Xⁱ) < wᵢ + δᵢ × (k+1)**

Summing this inequality over i = 1, 2, ..., m and using Claims (Freeloader Distance) and (Mergeset Limit), we get:

**weight(X⁰) - weight(Xᵐ) < m × 4k + δ₁(k+1) + ... + δₘ(k+1) ≤ 4×ℓ×k**

where the last inequality holds since k > 0. The desired result follows because F is in X⁰.Past, so weight(F) < weight(X⁰).

***

**Claim 4 (Finality Violated):**

For all i = 0, 1, ..., m: **P is in Xⁱ_Finality.Chain**

*(Intuition: The pruning block P sits deep enough that it remains on the canonical chain of every finality block throughout the process. Claim 3 ensures Xⁱ doesn't drift too far back — and π was specifically chosen (π = 2φ + 4ℓk + 2k + 2) to ensure P stays "in reach" of all Xⁱ_Finality blocks. This is essential for the contradiction: if Y were in Xⁱ_Finality.Future, it would also be in P.Future — contradicting our setup.)*

*Proof:* By induction on i.

**Basis:** For i = 0 (where X = X⁰), the claim follows immediately since P and X⁰_Finality are both in X.Chain and weight(X⁰_Finality) > weight(P).

**Inductive step:** Assume that P is in Xⁱ⁻¹_Finality.Chain. We will now show that P is in Xⁱ_Finality.Chain.

By definition of Bⁱ we have that Xⁱ⁻¹_Finality is in Bⁱ.Chain. The selection process also implies that Xⁱ_Finality and Xⁱ are both in Bⁱ.Chain. Combining with the induction hypothesis, P and Xⁱ_Finality are both in Bⁱ.Chain. Since both blocks share a chain, it remains to show that **weight(P) < weight(Xⁱ_Finality)**.

Following Claim (Score Distance) and noting that Xᵐ is in Xⁱ.Past, we get that **weight(Xⁱ) > weight(F) - 4×ℓ×k**. Combining with Corollary (Depth Bounds) over Xⁱ and φ, we have:

**weight(Xⁱ_Finality) > weight(F) - 4×ℓ×k - φ - k - 1**

On the other hand, by plugging Virtual, π, and φ into Corollary (Depth Relation), we get that:

**weight(P) < weight(F) - 4×ℓ×k - φ - k - 1**

Therefore **weight(P) < weight(Xⁱ_Finality)**.

***

**Conclusion:**

From Claim (Finality Violated) it follows that for all i = 0, 1, ..., m, **Y is in the Anticone of Xⁱ_Finality**. To see this: if Y were in Xⁱ_Finality.Future then Y would be in P.Future (contradiction), and Y being in Xⁱ_Finality.Past contradicts Y being in Xⁱ.MergeSet.

This justifies the induction hypothesis that Bⁱ⁺¹ must exist for all i = 0, 1, ..., m — otherwise rules (R-I) or (R-III) would be violated.

Specifically, we must assume that a block B^(m+1) ≠ Y exists, where by the selection criteria for B^(m+1), we have Xᵐ_Finality ∈ B^(m+1).Chain. However, by the halting condition of our process, Y must also be in B^(m+1).Chain. Since Y is in the Anticone of Xᵐ_Finality, they cannot both lie on the same chain — leading to a contradiction.

*(Summary: We assumed Y eventually enters Virtual.Past. This forces an infinite regress or impossible chain membership. Therefore our assumption was wrong — blocks discovered outside P.Future can NEVER enter Virtual.Past.)*

═══════════════════

## Final Rule

From this follows that it is secure to implement the following:

> **Pruning Rule:**
> - All blocks in P.Past can be pruned, and
> - For block B, if it holds that B is not in P.Future and B is not in Virtual.Past (that is, B violates finality rules of Virtual), then B can be discarded.

═══════════════════

## References

**[1]** Yonatan Sompolinsky, Shai Wyborski, and Aviv Zohar. *PHANTOM and GHOSTDAG: A scalable generalization of nakamoto consensus*. Cryptology ePrint Archive, Report 2018/104, 2018. https://eprint.iacr.org/2018/104.

**[2]** Michael Sutton and Shai Wyborski. *Prunality of the GHOSTDAG Protocol*. July 2020. Available at: https://github.com/kaspanet/docs/blob/main/Reference/prunality/Prunality.pdf
