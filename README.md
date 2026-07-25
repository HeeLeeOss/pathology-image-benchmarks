# pathology-image-benchmarks

> Computational pathology is moving fast and reproducing badly. Papers report state-of-the-art numbers on whole-slide images, but comparisons are routinely undermined by three problems: (1) **data-sourc  ·  **Risk tier:** med  ·  **Status:** planning

Computational pathology is moving fast and reproducing badly. Papers report state-of-the-art numbers on whole-slide images, but comparisons are routinely undermined by three problems: (1) **data-source leakage** — TCGA slides carry strong site- and scanner-specific signatures, so a naive random split lets a model "cheat" by recognizing the submitting site rather than the biology (Howard et al., 2021, documented this confound directly); (2) **license and provenance fog** — collections are mixed together with unverified reuse terms and no per-slide provenance, and some widely-used model weights and molecular annotations (e.g. COSMIC, OncoKB) are **non-commercial only**; and (3) **irreproducibility** — splits, preprocessing (tiling, magnification/MPP normalization, stain normalization), and environments are not pinned, so reported numbers cannot be re-derived. The result is an inflated, hard-to-trust literature on exactly the data that should be the field's shared commons.

**Definition of shipped:** **Non-goals**

This is a **Hee-Lee Oss** good-deed project. Contributors pull a task, do it with their own coding agent, and open a PR. Get started: https://github.com/HeeLeeOss/hee-lee-oss-downloads

## Plan
- [PLAN.md](./PLAN.md) — robust enterprise plan (vision, architecture, roadmap, risks; includes an applied-improvements appendix + review sign-off)
- [TASKS.md](./TASKS.md) — schema-mapped task backlog
- [tasks/](./tasks/) — ready-to-pull task JSON(s)

## Contribute
```bash
hee-lee-oss browse
hee-lee-oss next --repo HeeLeeOss/pathology-image-benchmarks --no-fork
```

## Licensing & review
- Open license (see PLAN.md).
- Risk tier **med** — deeds are *delivered, not merged*; a domain reviewer (and expert sign-off for any high-stakes content) must approve before merge.

> Planning stage; no adopting partner secured yet (`verifiedNeed: false` on delivery-dependent tasks).
