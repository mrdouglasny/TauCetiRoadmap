# Roadmap: weighted orthogonal L² bases — completeness, Hilbert-basis structure, and product bases of orthogonal systems

## Overview

Mathlib has several families of orthogonal polynomials as **algebraic** objects
(`Polynomial.hermite`, `Polynomial.Chebyshev`, `Polynomial.shiftedLegendre`) and a complete
abstract Hilbert-basis API (`HilbertBasis`, `mkOfOrthogonalEqBot`, `tsum_inner_mul_inner`), but
**no bridge between them**: nothing turns a polynomial **orthogonality relation**
`∫ pₘ(x) pₙ(x) w(x) dx = cₙ · δₘₙ` into a complete orthonormal **basis** of an L² space, and nothing
assembles one-dimensional bases into a basis of an L² product measure.

This area builds that **L² Hilbert-basis layer** for orthogonal systems:

- a **completeness** toolkit (moment determinacy / Fourier uniqueness) — the step that upgrades
  "orthogonal" to "complete orthonormal basis";
- the **orthogonality-relation → `HilbertBasis`** bridge (the √w normalization + Parseval);
- the **L²-product-basis** lemma (a Hilbert basis of `L²(μ ⊗ ν)` from bases of the factors).

The **Hermite basis of `L²(ℝ)`** is the worked anchor (**Part A**, the v1 deliverable) from which the
family-agnostic spine (**Part B**) is abstracted. The spine is then **exercised by a second family —
the Chebyshev basis of `L²([-1,1])`** (**Part C**), which uses the other completeness route
(Weierstrass density, not Fourier) and whose orthogonality relation is already in Mathlib; and it
yields the **multidimensional Hermite basis** (**Part D**, future). Laguerre and Jacobi are a
*separate* future roadmap — Mathlib has neither family, so grounding them means defining the
polynomials first.

### Scope boundary (what this area is, and is not)

This area provides **Hilbert-space structure**: completeness, `HilbertBasis`, Parseval, product
bases. The split is by *kind of statement*, and **everything is grounded** — it either exists in
Mathlib or is a target here (no dangling external dependencies):

- The family's **orthogonality relation** `∫ pₘ pₙ w = cₙδ` is an existing Mathlib lemma where one
  exists, and otherwise a **target of this roadmap**. For Hermite it is **A1**
  (`integral_hermite_mul_hermite_mul_gaussian`, below): Mathlib has `Polynomial.hermite` and
  `integral_gaussian`, but *not* the weighted orthogonality integral, so this roadmap proves it.
- The family's **polynomial calculus** (recurrences, generating functions, Rodrigues) is taken from
  Mathlib's `Polynomial.*` API where present; the Hermite-specific pieces Mathlib lacks (the
  derivative identity, the generating function) are likewise **A1 targets**.
- The general bridge **B2** takes the orthogonality relation as a *lemma hypothesis* — so it is
  family-agnostic and grounded by construction (a parametrized theorem, not a dependency).

The line is: an *integral identity* is consumed (as a Mathlib lemma or a roadmap target); a
*Hilbert-space theorem* ("...hence the normalized functions are a complete orthonormal basis of
L²") is this area's contribution.

## Generality bar (decide up front; do not silently specialize)

- **The basis layer is family-agnostic and scalar-generic.** Bases are stated through the
  measure-generic `HilbertBasis ι 𝕜 (Lp 𝕜 2 μ)` API over `[RCLike 𝕜]` (the real pointwise
  functions cast via `algebraMap ℝ 𝕜`); do **not** duplicate the API for `ℝ` and `ℂ` separately.
- **The bridge is parametrized by the orthogonality relation.** Part B's bridge (B2) takes the
  relation `∫ pₘ pₙ w = cₙδ` (with `cₙ > 0`) and a polynomial-density/completeness hypothesis as
  **arguments** — family-agnostic and grounded by construction. The Hermite instance supplies that
  relation from **A1** (a target here); each family's polynomial identities are reused from
  Mathlib where present, and otherwise are targets (Hermite's are A1).
- **Weights are classical, with a stated completeness route.** Gaussian `e^{-x²}` (Hermite,
  Route 1 — moment determinacy) and the Chebyshev weight `(√(1-x²))⁻¹` on `[-1,1]` (Chebyshev,
  Route 2 — Weierstrass). The completeness hypothesis is explicit in the general lemma: finite
  exponential moments (Gaussian-type decay) for the unbounded case, or compact support for the
  Weierstrass case. (Laguerre `x^α e^{-x}` / Jacobi `(1-x)^α(1+x)^β` are a separate roadmap.)
- **Measures named explicitly.** `volume` on `ℝ`/intervals, `Measure.pi`/`Measure.prod` for
  products — never inferred — so the product-basis lemma and the multi-d instances are extensions,
  not refactors.
- **Deliberately out of scope:** the general orthogonal-polynomials-**of-a-measure** construction
  (Gram–Schmidt), Favard's theorem / three-term recurrences, and every per-family identity. Those
  are the polynomial-identity layer, not the L²-basis layer this area owns.

## What Mathlib already has (consume)

- Algebraic families: `Polynomial.hermite` (+ `hermite_succ`, `deriv_gaussian_eq_hermite_mul_gaussian`),
  `Polynomial.Chebyshev.T`, `Polynomial.shiftedLegendre`.
- **Chebyshev orthogonality, already proved** (Part C's input), w.r.t.
  `measureT = (volume.withDensity (√(1-x²))⁻¹).restrict (Ioc (-1) 1)`:
  `integral_eval_T_real_mul_eval_T_real_measureT_of_ne` (off-diagonal `= 0`),
  `integral_eval_T_real_mul_self_measureT_zero` (`= π`),
  `integral_T_real_mul_self_measureT_of_ne_zero` (`= π/2`), and `T_real_cos`
  (`(T ℝ n).eval (cos θ) = cos (n θ)`).
- **Stone–Weierstrass** `polynomialFunctions_closure_eq_top` — polynomials dense in `C([a,b])`, the
  compact-interval completeness route (B1 Route 2).
- Hilbert-basis API: `HilbertBasis`, `HilbertBasis.mkOfOrthogonalEqBot`,
  `HilbertBasis.hasSum_inner_mul_inner`, `HilbertBasis.tsum_inner_mul_inner`; `Orthonormal`; `MemLp.toLp`.
- `OrthonormalBasis.tensorProduct` — **finite-dimensional only** (the algebraic tensor of finite
  bases); there is no completed / `L²(Measure.pi)` product-basis API.
- `integral_gaussian`; the Fourier-transform API (`Fourier.fourierIntegral`) for the determinacy proof.

## What is missing (build here)

The completeness toolkit (both routes), the relation→`HilbertBasis` bridge, and the L²-product-basis
lemma; plus their instances: the Hermite basis (Part A) and the Chebyshev basis (Part C), and the
multidimensional Hermite basis (Part D). (Laguerre/Jacobi are a separate future roadmap.)

## Part A — The Hermite basis of `L²(ℝ)` (the v1 anchor)

The concrete milestone and the worked instance from which Part B is abstracted. The Hermite
functions are `ψₙ(x) = cₙ · Hₙ(x√2) · e^{-x²/2}` with `cₙ = (n!·√π)^{-1/2}` and
`Hₙ = Polynomial.hermite n` (probabilists'; built on Mathlib's existing family, *not* a new
polynomial). They are the eigenfunctions of the quantum harmonic oscillator and the canonical
orthonormal basis of `L²(ℝ)`. Part A ships first and develops the full object API; Part B is then
factored out of A1–A3's proofs.

*Placement:* polynomial facts (A1) mirror Mathlib's `RingTheory/Polynomial/Hermite/…`; the
analytic facts (A2–A3) live under `Analysis/SpecialFunctions/…`. Names describe the conclusion.

### A1 — Hermite polynomial API
- `derivative_hermite_succ : derivative (hermite (n+1)) = (n+1) • hermite n` (+ `@[simp]`); Mathlib's
  `degree_hermite` is reused (it makes the polynomials span the moment space in A3).
- Integrability of a polynomial against the Gaussian weight:
  `Integrable (fun x => aeval x p * Real.exp (-(x²/2)))`.
- The one-step weighted-pairing recursion `∫ p·H_{n+1}·w = ∫ p'·Hₙ·w` (one IBP via Rodrigues
  `(Hₙ·w)' = -(H_{n+1}·w)`); and the **generating function** `∑' n, Hₙ(x)·tⁿ/n! = e^{x·t - t²/2}`.
- **Milestone** `integral_hermite_mul_hermite_mul_gaussian`:
  `∫ x, Hₘ(x)·Hₙ(x)·e^{-x²/2} = if m = n then n!·√(2π) else 0`.
- *Acceptance:* `H₀=1, H₁=X, H₂=X²-1`; `⟨H₀,H₀⟩=⟨H₁,H₁⟩=√(2π)`; `⟨H₀,H₂⟩=0`; gen. fn. at `t=0` is `1`.

### A2 — The Hermite functions `ψₙ : ℝ → ℝ`
- `hermiteFunction n x = cₙ · aeval (x·√2) (hermite n) · e^{-x²/2}`, with regularity:
  `Continuous`, `ContDiff ℝ ⊤`, `MemLp _ 2 volume`, and a `SchwartzMap ℝ ℝ` with this underlying
  function. Companion API: parity `ψₙ(-x)=(-1)ⁿψₙ(x)`, `ψ₀`/`ψ₁` `@[simp]` forms, `‖ψₙ‖₂=1`.
- **Ladder relations** (pointwise/Schwartz-level): `x·ψₙ = √((n+1)/2)ψ_{n+1} + √(n/2)ψ_{n-1}` and
  `ψₙ' = √(n/2)ψ_{n-1} - √((n+1)/2)ψ_{n+1}`, whence `aψₙ=√n ψ_{n-1}`, `a†ψₙ=√(n+1)ψ_{n+1}`.
  Packaging `a, a†` as operators on `L²` with `[a,a†]=1` is a deliberate downstream target — state
  these ladder identities at the pointwise/`SchwartzMap` level so they elevate to continuous linear
  operators on `𝒮(ℝ)` later (the GFF / tempered-distribution use) without re-proving.
- **Milestones:** pointwise orthonormality `integral_hermiteFunction_mul_hermiteFunction`
  (`∫ ψₘψₙ = if m=n then 1 else 0`); the oscillator eigen-equation
  `-ψₙ'' + x²ψₙ = (2n+1)ψₙ`.
- *Acceptance:* `ψ₀ = π^{-1/4}e^{-x²/2}` (`n=0` boundary; `aψ₀=0`); `ψ₁=√2·x·ψ₀`; `a†ψ₀=ψ₁`.

### A3 — The Hermite basis of `L²(ℝ; 𝕜)`
- `hermiteFunctionLp (𝕜) : ℕ → Lp 𝕜 2 volume` (real `ψₙ` cast via `algebraMap ℝ 𝕜`);
  `Orthonormal 𝕜 (hermiteFunctionLp 𝕜)`.
- **Completeness** (the instance of **B1**): `(∀ n, ∫ x, f x·ψₙ(x) = 0) → f =ᵐ[volume] 0`, proved over
  `ℝ` by the **Fourier-integral route** of B1 — set `g := f·e^{-x²/2}`, which is `L¹` *and has every
  exponential moment finite* (`∫ e^{a|x|}|f|e^{-x²/2} ≤ ‖f‖₂·‖e^{a|x|}e^{-x²/2}‖₂ < ∞`, Cauchy–Schwarz),
  so `𝓕 g` (`Fourier.fourierIntegral`) is entire with all Taylor coefficients (= the moments) zero,
  hence `g = 0`. *Stay in the `fourierIntegral` API — do not detour through a finite-signed-measure API.*
- **Headline milestone** (for every `[RCLike 𝕜]`):
  `hermiteHilbertBasis 𝕜 : HilbertBasis ℕ 𝕜 (Lp 𝕜 2 volume)` (via `mkOfOrthogonalEqBot`), with
  Parseval in `tsum_inner_mul_inner` orientation `∑' n, ⟪f,ψₙ⟫·⟪ψₙ,g⟫ = ⟪f,g⟫`. `𝕜=ℝ` and `𝕜=ℂ`
  are one theorem. (Over `ℂ` the inner product is sesquilinear, so the Parseval proof for the
  `algebraMap ℝ 𝕜`-lifted *real* basis carries `starRingEnd` bookkeeping even though the `ψₙ` are
  real — a known boilerplate cost, not a design issue.)
- *Acceptance:* Parseval for an explicit `f`; coordinates of `ψ₀` are `Finsupp.single 0 1`;
  `‖f‖² = ∑' n, ‖⟪ψₙ,f⟫‖²`; both `ℝ` and `ℂ` instantiate.

## Part B — The family-agnostic spine (the reusable layer)

The three pieces that Part A's completeness/basis argument factors into, stated without reference to
any particular family.

### B1 — Completeness toolkit (two grounded routes)
*A polynomial-dense weighted system is complete in its L² space. Two grounded routes, used by the
two families below.*

**Route 1 — moment determinacy (unbounded support, e.g. Hermite).**
An `L¹` function `g` *all of whose exponential moments are finite* (`∫ e^{a|x|}·|g| < ∞` for every
`a ≥ 0` — and a single positive `a`, giving strip-analyticity, already suffices for uniqueness) and
all of whose polynomial moments vanish is a.e. `0` — because that integrability makes `𝓕 g`
**entire**, with the moments as its Taylor coefficients at `0`, so `𝓕 g ≡ 0`. **The decay
hypothesis is this weighted-`L¹` integrability, *not* a pointwise bound `|g| ≤ C·e^{-x²/2}`** — the
distinction matters because the completeness instance `g = (basis test fn)·√w` with an `L²` factor
is *not* pointwise-bounded, but *does* have finite exponential moments (Cauchy–Schwarz against the
Gaussian `√w`). *This is the load-bearing analytic content.*

**Weight-role note** (state explicitly): the completeness used by the
basis is in **`L²(dx)` with the `√w` envelope** — `g ∈ L²(dx)` orthogonal to every `pₙ·√w` ⟹ `g = 0`
— reduced to the moment lemma via `h := g·√w`. Do *not* conflate it with `L²(w)`-orthogonality (test
integrand `w`, not `√w`); they correspond under `g ↔ g·√w` but are distinct statements.

**Route 2 — Weierstrass density (compact support, e.g. Chebyshev).**
For a weight supported on a compact interval `[a,b]`, completeness is the density of polynomials in
`C([a,b])` (Stone–Weierstrass, `polynomialFunctions_closure_eq_top`) pushed into `L²`: a
square-integrable `g` orthogonal to every `pₙ` (against the weighted measure) vanishes a.e. This is
a **different mechanism** than Route 1 — included so the spine is exercised across both, and so the
compact families (Part C) are grounded without the moment-determinacy machinery.

### B2 — Orthogonality relation → `HilbertBasis`
Given polynomials `p : ℕ → ℝ[X]`, a weight `w` with `0 ≤ w` **and `0 < w` a.e.** (else `g·√w = 0`
fails to force `g = 0`), the relation `∫ pₘ pₙ w ∂μ = cₙ δ` with `cₙ > 0`, and completeness (B1),
the √w-normalized functions `x ↦ pₙ(x)·√(w x)/√(cₙ)` are orthonormal in `L²(μ)` and, with
completeness, form a `HilbertBasis ℕ 𝕜 (Lp 𝕜 2 μ)` (cast via `algebraMap ℝ 𝕜`), with Parseval —
**for any σ-finite reference measure `μ` on `ℝ`**, so the **Chebyshev** instance (Part C) lands on
`[-1,1]` — either as `μ = μ_T` (the Chebyshev measure) with `w = 1`, or as `μ = volume.restrict
[-1,1]` with `w = (√(1-x²))⁻¹` — not just `volume` on `ℝ`. The √w rescaling +
`mkOfOrthogonalEqBot` plumbing, once, for any family. **Instantiation carries a
change of variables when the family is defined on a rescaled argument:** the Hermite case uses
`w = e^{-x²}`, `pₙ = Hₙ(·√2)`, `cₙ = n!√π` (so `pₙ·√w/√cₙ = ψₙ`), which is A1's probabilists'
relation transported by `u = x√2` — *not* `w = e^{-x²/2}` read off A1 directly.

### B3 — L²-product basis
**Index-generic** (this resolves the `ℕ`-vs-product question): for Hilbert bases of `L²(μ)` and
`L²(ν)` indexed by *arbitrary* types `ι₁, ι₂`, B3 produces a Hilbert basis of `L²(μ ⊗ ν)` indexed
by `ι₁ × ι₂`; iterated over a finite family, a basis of `L²(Measure.pi μ)` indexed by `Π i, κ i`.
The family bridges (B2) are indexed by `ℕ` (degree), and B3 **consumes** those as the `ι = ℕ` case —
so the multidimensional basis is indexed by `Π i, ℕ` (multi-indices), and **B2 need not be
generalized away from `ℕ`**: the lift to products lives entirely in B3. (Mathlib has only the
finite-dim algebraic `OrthonormalBasis.tensorProduct`.) Not special-function-specific.

**Acceptance.** B1 applied to the envelope `√w = e^{-x²/2}` (i.e. `g = f·e^{-x²/2}`, `f ∈ L²`); B2
reproducing Part A's basis from the orthogonality relation; B3 giving an ON basis of
`L²(volume.prod volume)` from two copies of a 1-D basis.

## Part C — A second family: the Chebyshev basis of `L²([-1,1])`

A second instantiation, so the abstract spine (B1–B3) is **exercised by more than one family** and
across **both completeness routes** — the point of building the spine at all. The **orthogonality
relation is an existing Mathlib input**; the basis assembly + completeness are the targets (named
below).

- **Inputs (existing Mathlib).** The polynomials `Polynomial.Chebyshev.T` (`ℤ`-indexed; the basis
  uses `n : ℕ`), orthogonal w.r.t. the Chebyshev measure
  `measureT = (volume.withDensity (√(1-x²))⁻¹).restrict (Ioc (-1) 1)`. The orthogonality is already
  proved: `integral_eval_T_real_mul_eval_T_real_measureT_of_ne` (off-diagonal `= 0`, `n ≠ m`),
  `integral_eval_T_real_mul_self_measureT_zero` (`= π`), and `integral_T_real_mul_self_measureT_of_ne_zero`
  (`= π/2`). (`integral_eval_T_real_mul_eval_T_real_measureT` is the product-to-sum identity, not the
  vanishing.) Also `T_real_cos` (`(T ℝ n).eval (cos θ) = cos (n θ)`).
- **Completeness via B1 Route 2** (Weierstrass density), the compact-interval mechanism — *not* the
  Fourier/moment route, which is exactly why a second family is worth including.
- **Milestone:** `chebyshevHilbertBasis 𝕜 : HilbertBasis ℕ 𝕜 (Lp 𝕜 2 measureT)` (roadmap target) —
  the normalized `Tₙ/√cₙ` (`c₀ = π`, `cₙ = π/2`), assembled by **B2** from the Mathlib orthogonality
  + Route-2 completeness: the bridge demonstrated on a non-Hermite family.
- **Open targets this entails** (so "grounded" is honest, not hand-waved): `IsFiniteMeasure measureT`
  (derivable, currently no instance); the support/subtype bridge between `measureT` on `ℝ↾Ioc(-1,1)`
  and Weierstrass density on the compact subtype `Icc (-1) 1`; the `C(Icc) ↪ L²` density push
  (`ContinuousMap.toLp_denseRange`, needs `IsFiniteMeasure` + `WeaklyRegular`); the
  Chebyshev-span-to-polynomials step (`Polynomial.Sequence.span_degreeLT` via `chebyshevTsequence`);
  and the real→`[RCLike 𝕜]` extension of the real-valued Weierstrass density.
- *Acceptance:* `⟨T₀,T₀⟩ = π`, `⟨T₁,T₁⟩ = π/2`, `⟨T₀,T₁⟩ = 0`; and (a *target*, via
  `integral_measureT_eq_integral_cos` + a unitary-transfer statement, not from `T_real_cos` alone)
  `chebyshevHilbertBasis` corresponds to the cosine basis under `x = cos θ`.

## Part D — The multidimensional Hermite basis (future; consume B)

*Not v1; recorded so Parts A/B are built generically enough to instantiate. Fully grounded — `B3 ∘ A`.*

- **Multidimensional Hermite basis** of `L²(ℝᵈ)` — `Ψ_α(x) = ∏ᵢ ψ_{αᵢ}(xᵢ)`, `α : ι → ℕ` — the
  immediate `B3 ∘ A` instantiation: B3 over `ι` copies of the 1-D Hermite basis (A3), a
  `HilbertBasis (ι → ℕ) 𝕜 (Lp 𝕜 2 (Measure.pi (fun _ => volume)))`.

**A separate future roadmap:** Laguerre and Jacobi L² bases. Unlike Chebyshev, **Mathlib has neither
the Laguerre nor the Jacobi polynomials**, so grounding them means defining the families first —
which belongs in its own roadmap, not here.

## Dependency ordering

Build **A concretely first** (it is the v1 and the proving ground for the determinacy argument),
then **factor B1/B2 out** of A's completeness/orthonormality proofs into the family-agnostic lemmas
— so B is validated by reproducing A, not speculative. **B3** is independent of A/B1/B2 and can land
in parallel. **C** consumes A + B and is out of the v1 milestone.

## References

- G. Szegő, *Orthogonal Polynomials*, AMS Colloq. 23 (Chs. II–V).
- N. I. Akhiezer, *The Classical Moment Problem* (determinacy).
- B. Simon, *Szegő's Theorem and Its Descendants* (orthogonal polynomials and L² completeness).
- M. Reed, B. Simon, *Methods of Modern Mathematical Physics II*, §X.6.
- G. B. Folland, *Harmonic Analysis in Phase Space*, §1.
