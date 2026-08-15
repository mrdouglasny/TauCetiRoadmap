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
`⟨T₁,T₁⟩ = π/2`. 51 `example`s in total. No gaps.

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

The natural reading of "archive it" is one PR that moves the directory and drops `Discharged.lean`
in beside it. I built that, and then the drift above convinced me it is the wrong shape.

Under `Completed/`, a file is outside the `TauCetiRoadmap.*` glob and **is not built by CI** — as
`Completed/README.md` already notes for archived `Suggested.lean` files. That is the right
behaviour, not a defect: an archived record is meant to stay true about the moment the plan was
retired, not to track a library that carries on moving. What it should not do is be asserted from
a local run made days before the decision, and the drift below shows days is enough to matter.

So, two PRs:

**PR 1 — land the discharge record while the roadmap is still active.**
`TauCetiRoadmap/OrthogonalL2Bases/Discharged.lean`, where the glob builds it, so **CI proves the
completion claim** rather than taking my word for it. **The bump is tested: the pin moves to
`bfeffdf0`, and a full `lake build` of every roadmap succeeds — 8791 jobs, exit 0, no new errors,
with `Discharged.lean` among the modules built.**

*Whether that pin bump should be part of the procedure in general is genuinely open, and I am
soliciting opinions rather than asserting an answer — see "The pin question" below.* Here it was
forced: the pin sat at `86cc55d9`, **between** the two drift events, so the file could not be
green against both the pin and `main`, and moving forward was the only way to have it green at
all.

**PR 2 — archive.** Move the directory to `Completed/`, with the completion note, the
`Completed/README.md` entry, the root README move, the `TauCetiRoadmap.lean` import, and the two
issue-template dropdown entries. Purely mechanical once PR 1 is green.

**One thing this move costs that `EffectiveBounds` did not.** That roadmap had no inbound links.
This one is a spine other roadmaps cite: five relative links from `RepresentationTheory/README.md`
and `CompactGroups/README.md`, plus two prose paths in `CompactGroups/Suggested.lean`, break when
the directory moves. They are repointed in PR 2. Worth knowing generally — archiving a *cited*
roadmap is not a pure directory move, and more of them will be cited than `EffectiveBounds` was.

### The pin question — where I would like other opinions

`Discharged.lean` has to be green, and this repository pins Tau Ceti. Green against *what*?

**(a) Bump the pin to current `main`, then verify.** The certificate then means "discharged in Tau
Ceti as it stands today", which is the claim a maintainer actually wants when retiring a plan.
Cost: every closure PR drags a dependency bump, and a bump can redden other roadmaps'
`Suggested.lean` for reasons having nothing to do with the roadmap being closed. Here it was
clean, but that is one data point.

**(b) Verify against the existing pin.** Cheap, self-contained, no risk to anything else. But the
certificate then means "discharged as of whenever the pin last moved", possibly weeks stale, and
the decision is about today's library.

**(c) Keep the record out of the repository entirely** — attach it to the closure PR as a review
artifact and let it evaporate on merge. Its whole job is to inform one decision, so this has an
honest appeal; the cost is that the evidence is not there later when someone asks why an area was
closed.

I went with (a) because this case forced it, and I do not think one forced case settles the rule.
The cost driver is how often we want that pin moving for unrelated reasons, which others will
judge better than I can.

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

This couples `TauCetiRoadmap` CI to Tau Ceti renames, so bumping the pin will sometimes turn the
roadmap build red for mechanical reasons. That cost is real. Two things make it acceptable: the
coupling already exists — four active `Suggested.lean` files (`QuiverRepresentations`,
`IntegralLattices`, `ContourIntegration`, `DGAInfinity`) already reference `TauCeti.` declarations,
so this extends an accepted pattern — and the repairs are one-line and mechanical, which is
precisely the work the project's workers are good at.

If you would rather not take on that coupling at all, the fallback is to archive with the record
clearly labelled a snapshot pinned to a named commit, and drop the claim that it stays true. I
would rather have the live version, but a labelled snapshot still beats prose.

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
