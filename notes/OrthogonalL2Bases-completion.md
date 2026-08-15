<!-- Written by Claude Opus 5. Reviewed and submitted by @mrdouglasny. -->

> **Authorship.** This document was written by **Claude Opus 5** (Anthropic), which performed the
> audit it describes. It is submitted, reviewed and vouched for by @mrdouglasny. The Lean evidence
> it cites is machine-checked and reproducible from `Discharged.lean`; the prose around it is not,
> so read the argument critically and the numbers literally.

# Proposal: declare `OrthogonalL2Bases` complete

**Summary.** Every target of the `OrthogonalL2Bases` roadmap has landed sorry-free in Tau Ceti.
The last one landed on **2026-07-30**, so the roadmap has been finished for two weeks without
anyone noticing. I propose archiving it under `Completed/`, as `EffectiveBounds` was in #50.

The evidence is a Lean file rather than a prose claim, and it is offered as such: `Discharged.lean`
restates all 27 targets of `Suggested.lean` and closes each with the Tau Ceti declaration that
realizes it. If it elaborates, the correspondence holds — checked by the kernel, not by matching
names and not by me asserting it.

Completion is still your judgment against the roadmap `README.md`, which is the definitive
document. What `Discharged.lean` removes is the bookkeeping half of that judgment, not the
mathematical half.

Worth saying plainly, because it sets the bar: **closing a roadmap is a judgment about the
roadmap, not about the area.** Tau Ceti will go on developing weighted `L²` bases after this is
archived, and should. The question is only whether this plan has finished directing work — not
whether the subject is exhausted.

---

## How completion was checked

**27 of 27 targets discharge.** Definition targets are checked by type; theorem targets by
applying the Tau Ceti declaration to the roadmap's own statement. Verified clean at four Tau Ceti
commits: `ce9fe563` (08-11), `58d1ae1c` (08-14), `a7c87175` (08-15) and `73d0d896` (08-15).

**Name matching would have got this wrong in both directions**, which is the case for doing it in
Lean. Matching declaration names scores the roadmap 17/26 — nine false negatives:

| roadmap target | actual Tau Ceti declaration |
|---|---|
| `barePolyLp` | `bareNormalizedLp` |
| `orthonormal_barePolyLp` | `orthonormal_bareNormalizedLp` |
| `barePolyLp_ortho_eq_bot` | `orthogonal_span_range_bareNormalizedLp_eq_bot` |
| `_root_.HilbertBasis.map` | `HilbertBasis.mapₗᵢ` |
| `derivative_hermite_succ` | `_root_.Polynomial.derivative_hermite_succ` |
| `coe_gaussianHermiteHilbertBasis` | `coeFn_gaussianHermiteHilbertBasis` |
| `coe_gaussianHermitePiBasis` | `coeFn_gaussianHermitePiBasis` |
| `ae_eq_zero_..._of_finite_expMoments` | same name, generalized `volume` → any `ν : Measure ℝ` |
| `hermiteℝ` | `hermiteℝ` (extractor mangled the `ℝ`) |

and one false **positive**: `prodHilbertBasis_apply` matches by name, but Tau Ceti's declaration
under that name is the `L2prodMul` form. The roadmap's a.e.-product statement is
`coeFn_prodHilbertBasis`. A name-matching report would have scored that target green while
pointing at the wrong theorem.

**Three targets need more than a rename**, and this is the part no textual method can see. Tau Ceti
generalized the whole Part-B2 block from a polynomial family on `ℝ` to an arbitrary family
`f : ℕ → α → ℝ` on an arbitrary measurable space, so every B2 target discharges only through the
instance `f := fun n x => (p n).eval x`. Several also arrive *stronger* than asked:
`orthonormal_bareNormalizedLp` weakens `0 < w` a.e. to `0 ≤ w` a.e., and
`orthogonal_span_range_bareNormalizedLp_eq_bot` drops `hwpos` and `hwm` entirely and needs one
finite exponential moment where the roadmap asked for all rates. Discharging those requires
`hwpos.mono fun _ hx => hx.le` and `⟨1, one_pos, hexp 1 zero_le_one⟩` written out explicitly. The
kernel forces those bridges into the open; prose hides them.

**Beyond `Suggested.lean`.** The roadmap `README.md` covers material the target file disclaims, and
that material is now discharged to the same standard, in Part 2 of the same file: the A2 Hermite
function API (regularity, parity, ladder, oscillator eigen-equation, Schwartz packaging), the
function-side `hermiteHilbertBasis` with its element-level export and Parseval, the Fourier
eigenrelation `𝓕 ψₙ = (-i)ⁿ ψₙ` (which the README flags as a target absent from Mathlib), Part C
Chebyshev, Part D's `hermiteFunctionPiBasis`, and the README's own *Acceptance* criteria —
`⟨H₀,H₀⟩ = ⟨H₁,H₁⟩ = √(2π)`, `⟨H₀,H₂⟩ = 0`, the generating function at `t = 0`, `⟨T₀,T₀⟩ = π`,
`⟨T₁,T₁⟩ = π/2`, and A2's own — `ψ₀ = π^{-1/4}e^{-x²/2}`, `ψ₁ = √2·x·ψ₀`, `a ψ₀ = 0`,
`a† ψ₀ = ψ₁`. 55 `example`s in total. No gaps.

Two items overshoot. The README lists the ladder operators `a`, `a†` on `𝒮(ℝ)` as a deliberate
*downstream* target rather than one of its own; we have them, with `[a,a†] = 1`. The Fourier
eigenrelation is likewise stronger than the minimum the README settles for.

What this does not cover: the README's narrative requirements — the generality bar, the scope
boundary, the instruction to state the ladder identities so they elevate to `𝒮(ℝ)` later. Those
are guidance for contributors while the roadmap is steering them, rather than conditions on
retiring it, so they bear on closure less than the target list does. They remain yours to weigh.

---

## The rename, and why it is not an error

Twice while this record was being written, Tau Ceti moved something under it.

**2026-08-13 — a namespace move.** `hermite_generating_function` moved from `TauCeti` to
`Polynomial`, tracking Kim's upstream adaptation in mathlib4#42724, so the name matches Mathlib's
existing Hermite API ahead of upstreaming. `GeneratingFunction.lean`'s docstring says exactly that.

**2026-08-15 — a module split.** `Gaussian/Hermite/PiBasis.lean` became
`Gaussian/Hermite/Pi/Basis.lean` (and likewise on the Hermite-function side). Declaration names
unchanged; only the import path moved.

Each broke `Discharged.lean`. Neither is an error, in three distinct senses worth separating,
because the natural reading is "your check is fragile" and that reading is wrong.

**1. The changes were correct.** One aligns a namespace with Mathlib ahead of upstreaming; the
other is ordinary module hygiene. Good things happened. Nothing regressed: the theorems are still
there, still proved, still sorry-free.

**2. Renaming is Tau Ceti's stated policy, not an accident.** The project does not preserve
backwards compatibility, and requires every *in-repository* use to be updated in the same PR. A
roadmap discharge record is by construction not an in-repository use, so it falls in the one gap
that rule does not close. That is an argument for making the record a build artifact — so the
coupling is enforced *somewhere* — rather than an argument against having one.

**3. A red build here does not mean the mathematics regressed.** It means one line of a bookkeeping
file names a declaration that moved. Both repairs were one line and took seconds. That is the
failure mode you want: loud, precise, trivially fixed, and attributable to a specific commit.

Now the counterfactual. A prose `STATUS.md` written on 08-11 would today still report the roadmap
complete while citing a declaration that no longer exists and an import path that no longer
resolves, and nothing anywhere would say so. Two drift events in four days, both silent to every
method the project currently uses. The audit's "failures" are the only thing that noticed.

**One hazard found along the way, worth recording.** After bumping the Tau Ceti pin, the file
appeared to elaborate cleanly — but it was reading a **stale `.olean`** from the previous pin. The
green was false. It only surfaced because I separately noticed the module build had failed. Anyone
running this check must build the imported modules at the target revision and confirm *that* step
succeeded, not merely that `lake env lean` exited 0. In CI this is automatic; by hand it is not.

---

## What I propose, concretely

Two PRs, and a record that pins itself.

**PR 1 — land the discharge record while the roadmap is still under `TauCetiRoadmap/`,** where
the `TauCetiRoadmap.*` glob builds it, so CI certifies the claim rather than a contributor's
laptop. This carries a forward bump of the Tau Ceti pin to `bfeffdf0`; a full `lake build` of
every roadmap passes, 8791 jobs, no new errors, with `Discharged.lean` among the modules built.

**PR 2 — archive.** Move the directory to `Completed/`, with the completion note, the
`Completed/README.md` entry, the root README move, the `TauCetiRoadmap.lean` import, and the two
issue-template dropdown entries. `.github/scripts/check_roadmap_areas.py` — the repository's own
consistency check, not mine — confirms the three copies of the roadmap list stay in sync.

### The pin question, and how it resolved

I originally put this section up as an open question: should a closure PR bump the pin and verify
against current `main`, verify against whatever the pin happens to be, or keep the record out of
the repository altogether? Each had a bad edge. Bumping drags a dependency change into every
closure PR. Not bumping certifies against a possibly weeks-stale library. Discarding the record
loses the evidence.

The resolution came from a better question: not *which moving target should the record chase*, but
*why is it chasing one at all*. Archival material should never need updating. So the record now
**names its own revisions** and is checked against those:

```lean
-- tauceti-discharge:v1 {"roadmap":"OrthogonalL2Bases","tauceti":"bfeffdf0...",
--                       "mathlib":"77cbcbc6...","toolchain":"leanprover/lean4:v4.34.0-rc1"}
```

`.github/scripts/check_discharged.py` reads that header, materializes Tau Ceti at *that* revision
with *that* toolchain in a scratch workspace, builds the modules the record imports, and
elaborates it there — never against what the repository pins today. The archived file is
therefore **not** in the ordinary `TauCetiRoadmap` build. A roadmap nobody is working on should
not redden CI every time the pin moves forward, and a frozen record should not need touching
again.

This also dissolves a requirement I thought I had. I had worried the record must be re-checked
after PR 2 as well as PR 1, since the pin could move between the two merges. With the record
pinned to a fixed commit, the claim cannot rot: "these 55 statements discharge against Tau Ceti
`bfeffdf0`" is as true a year later as the day it was written.

### On reproducibility, honestly

The re-check is a bonus with a decay curve, not the load-bearing part, and the script is
**deliberately not wired into required CI**.

**It does work today, and the cost is modest.** From a cold scratch workspace — clone Tau Ceti at
the pinned revision, fetch Mathlib's cache, build the 23 Tau Ceti modules the record imports,
elaborate it — the full re-verification takes **289 seconds** and exits 0. That is the number with
everything maximally favourable, so treat it as the floor rather than the estimate.

The floor rises. Re-verification leans on things outside this repository that will eventually give
way: Mathlib and Tau Ceti artifact caches retaining the pinned revisions, elan still serving a
pinned release-candidate toolchain, that toolchain still running on a future host. The expensive
step is never elaborating the record itself; it is materializing what the record imports. The
first attempt failed for exactly that reason, having fetched Tau Ceti without building it — five
minutes today becomes a from-source Mathlib build the day a cache is pruned. As a mandatory gate
this becomes a liability the first time that happens, to no one's benefit on a roadmap nobody is
working on; on demand it merely stops being useful, which is the better failure.

The header is worth having either way. Once the rebuild is impractical, "these statements were
discharged against Tau Ceti `bfeffdf0`, Mathlib `77cbcbc6`, toolchain `v4.34.0-rc1`" remains a
precise, falsifiable historical claim — considerably more than "the maintainers judged this
complete in August 2026".

**One thing this move costs that `EffectiveBounds` did not.** That roadmap had no inbound links.
This one is a spine other roadmaps cite: five relative links from `RepresentationTheory/README.md`
and `CompactGroups/README.md`, plus two prose paths in `CompactGroups/Suggested.lean`, break when
the directory moves. They are repointed in PR 2. Worth knowing generally — archiving a *cited*
roadmap is not a pure directory move, and more of them will be cited than `EffectiveBounds` was.

### Scope, and one thing deliberately left out

This is a procedure for **closing a roadmap out**, not a standing obligation on active ones. The
record is written when someone believes an area is done and needs to be green exactly once, at the
moment of the judgment.

There is an obvious cousin for active roadmaps — seed `Discharged.lean` with every target a
`sorry`, and let each PR replace one, so the remaining-`sorry` count is the remaining-work count.
That is a different proposal with a different cost profile, a standing maintenance obligation
rather than a one-time certificate, and it should be argued on its own merits. Future work,
flagged rather than smuggled in here.

### Answering the obvious objection

*"This couples `TauCetiRoadmap` CI to Tau Ceti renames, so the build will go red for mechanical
reasons."*

It does, but only for the single PR that lands the record, and only while the roadmap is still
active. That coupling already exists and is accepted: four active `Suggested.lean` files
(`QuiverRepresentations`, `IntegralLattices`, `ContourIntegration`, `DGAInfinity`) already
reference `TauCeti.` declarations, so a closure PR is not introducing a new kind of dependency.
The repairs are one-line and mechanical, which is the work the project's workers are good at.

On archival the coupling stops entirely. The record leaves the build and is thereafter checked, on
demand only, against the revisions it names itself. So the permanent cost of closing a roadmap
this way is zero: no archived file can ever redden CI, and no archived file ever needs updating.
That is the property the pinned header buys, and it is why I abandoned an earlier draft that kept
archived records in the build.

---

## If this is worth generalizing

There are **317 targets across 13 roadmaps** (CFSGStatement 79, HodgeStructures 66, EllipticCurves
32, Exchangeability 30, OrthogonalL2Bases 27, OptimalTransport 26, and a tail). Cheap model
proposes the discharge, Lean verifies, strong model spent only on residuals that fail to compile —
efficient precisely because verification is sound and free.

The metric it yields is *declared targets discharged*, which is not roadmap completion and should
not be reported as a percentage. The distribution is too lopsided to support one: `EllipticCurves`
states a single target against 73 merged PRs and eight layers, while `Exchangeability` states 19.
It is a leading indicator that saturates early. I would rather ship a cheap number with its limits
stated than a percentage nobody can defend.

A separate and smaller ask, independent of the above — decline it without prejudice. Roadmaps are
supposed to grow `Suggested.lean` as later layers become expressible; `EffectiveBounds`' own header
says so, and it was archived complete still holding three Layer-1 targets. If that became a step in
the completion checklist, this metric would get substantially more informative.
