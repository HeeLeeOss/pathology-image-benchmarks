# pathology-image-benchmarks — PLAN.md

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

Open, reproducible **whole-slide-image (WSI) benchmarks for computational pathology**, built
**exclusively on open-access cancer imaging** (open-tier TCGA diagnostic/tissue slides; CPTAC and
other CC-licensed collections via The Cancer Imaging Archive) with **no identifiable patient data**.
The deliverable is a public benchmark suite — leakage-safe data splits, an evaluation harness, honest
baselines, datasheets, and model cards — that lets method developers compare computational-pathology
models on a level, reproducible, license-clean footing. This is **research infrastructure, not a
medical device**: nothing here diagnoses, grades, or advises on the care of any real patient.

---

## Executive summary

Computational pathology is moving fast and reproducing badly. Papers report state-of-the-art numbers
on whole-slide images, but comparisons are routinely undermined by three problems: (1) **data-source
leakage** — TCGA slides carry strong site- and scanner-specific signatures, so a naive random split
lets a model "cheat" by recognizing the submitting site rather than the biology (Howard et al., 2021,
documented this confound directly); (2) **license and provenance fog** — collections are mixed
together with unverified reuse terms and no per-slide provenance, and some widely-used model weights
and molecular annotations (e.g. COSMIC, OncoKB) are **non-commercial only**; and (3)
**irreproducibility** — splits, preprocessing (tiling, magnification/MPP normalization, stain
normalization), and environments are not pinned, so reported numbers cannot be re-derived. The result
is an inflated, hard-to-trust literature on exactly the data that should be the field's shared commons.

This project builds the missing **open, license-clean, leakage-aware benchmark layer**. It ingests
only open-access imaging, re-verifies de-identification defensively, fixes **site-stratified splits**
that demonstrably do not leak source-site, ships a containerized evaluation harness with honest
baselines, and documents everything with datasheets, model cards, and per-slide provenance. The
public good is the **trustworthy, reusable benchmark itself** — the thing the field can cite, extend,
and reproduce.

**The binding constraint is data, not modeling.** The first build item — before any pixel is
ingested — is the **licensing + de-identification gate**: an allow-list that admits a collection only
after its open-access status and reuse license are verified and recorded, and a de-identification
re-scan that quarantines any WSI whose **label/macro/associated images** might carry burned-in PHI
(patient names, dates, MRNs). Controlled-access data (dbGaP, EGA, individual-level genomics,
identifiable patient data) is **categorically out of scope** — a task that proposes to touch it is
**refused and flagged**, full stop. This gate is treated as a safety-critical subsystem and gates CI.

**Risk tier: MEDIUM.** The project is research infrastructure on open data, requiring **domain
accuracy** (correct pathology task framing, leakage-safe methodology, honest metrics) — so a
qualified computational-pathology / bioinformatics reviewer signs off on task definitions, splits, and
results. It is **not** patient-facing and produces **no medical advice**; every model and dataset
artifact carries a **"not for clinical or diagnostic use"** notice. **If the scope ever expands to
patient-facing interpretation or clinical-decision content, that content becomes HIGH tier** and is
gated behind **oncologist + patient-advocate sign-off** — but that is explicitly out of scope here.

**Honesty note: no partner, steward, or compute sponsor is yet secured.** Every adoption- and
delivery-dependent task is marked `verifiedNeed: false` until a real downstream user (a research lab,
a benchmark/challenge organizer, or an educational program) commits to use the benchmark. The project
is **not "shipped" on merge**; it is shipped only when the benchmark is publicly released, methods-
reviewed, and demonstrably reused by someone outside the team. A **dated partner/compute-acquisition
plan** and a **build-vs-mothball decision rule** (below and in *Risks*) govern what happens if those
commitments slip, rather than shipping a benchmark to no one.

**Compute is a first-class, unsolved dependency.** Elyos's donated lane is a **coding agent preparing
PRs** — it does not run GPU training. The benchmark *harness, splits, datasheets, baseline code, and
small-scale runs* are produced in the donated lane, but **large WSI training runs require donated or
funded GPU compute** that is not yet secured. The plan is sequenced so that everything of lasting
value (the leakage-safe splits, the evaluation protocol, the provenance/license layer) is deliverable
**without** heavy compute, and heavy-compute baselines are isolated into clearly-gated tasks.

---

## Problem & beneficiaries

**Who is helped (directly):** computational-pathology and medical-imaging **method developers,
ML researchers, and students** who need a fair, reproducible, license-clean way to benchmark
whole-slide-image models — and reviewers/editors who need to check that a claimed result is not an
artifact of data leakage or an unverifiable split.

**Who is helped (ultimately):** **cancer patients**, distally. Better-validated, honestly-benchmarked
methods are a precondition for tools that may eventually (after their own clinical validation, which
this project does **not** perform) improve diagnosis and research throughput. This project's
contribution is **methodological trustworthiness**, not a clinical tool.

**The need.** The field has open imagery (TCGA, CPTAC, TCIA collections) but lacks a shared,
leakage-aware, license-verified benchmark on top of it. Concretely:

- **Leakage inflates results.** TCGA WSIs encode site/scanner signatures; random or patient-level-only
  splits still leak *site*, so reported generalization is optimistic. Site-stratified (site-preserved)
  splits are known to be necessary but are not packaged as a reusable, audited artifact.
- **Licenses are unverified and mixed.** Open-access TCGA imaging, per-collection CC-BY TCIA data,
  CC0 challenge sets (Camelyon), and **non-commercial** model weights/annotations are casually
  combined, creating reuse risk for downstream users.
- **De-identification is assumed, not checked.** WSI files (SVS/DICOM) can carry **slide-label and
  macro images** with burned-in identifiers; even nominally de-identified public sets warrant a
  defensive re-scan before redistribution of any derived tiles or thumbnails.
- **Results don't reproduce.** Splits, preprocessing, and environments are unpinned.

**Verified need / partner:** **TO BE SECURED.** No research lab, challenge/benchmark organizer, or
educational program has yet committed to adopt or co-develop the benchmark, and **no GPU-compute
sponsor is secured**. Target partners to pursue: an academic computational-pathology lab; a
challenge organizer (e.g. a MICCAI/Grand-Challenge-style group); the maintainers of an open WSI
toolkit (OpenSlide, CLAM, TIAToolbox); or a data-science teaching program. Until one is secured, the
project builds the license/de-id gate, the data layer, the leakage-safe splits, the harness, and one
fully-documented benchmark task, marking all adoption work `verifiedNeed: false`. Outreach is
**dated, not open-ended** (see *Roadmap* and *Risks*): ≥3 candidate adopters and ≥1 compute path in
active conversation by **2026-09-30**; a domain (computational-pathology) reviewer secured by
**2026-08-31** (gates M2); an adopter/steward secured by **2026-12-31** (gates M4).

---

## Goals and non-goals

**Goals**

- Stand up an **open-only licensing + de-identification gate** that admits a collection only after its
  open-access status, reuse license, and de-identification are verified and recorded — and that
  categorically excludes controlled-access/identifiable data.
- Build a reproducible **WSI data layer**: ingest, checksum, per-slide provenance/manifest, tiling,
  and magnification/MPP normalization on open collections.
- Publish **leakage-safe, site-stratified splits** with an explicit **site-confounding audit** proving
  the splits do not leak submitting site/scanner.
- Define **≥1 well-specified benchmark task** with a frozen evaluation protocol, honest baselines, a
  submission/scoring harness, datasheets, and model cards.
- Make results **reproducible**: containerized environment, pinned splits and preprocessing, and a
  documented third-party reproduction of the headline baseline within tolerance.
- Get the benchmark **adopted and cited** by at least one external group (the Definition of Shipped).

**Non-goals**

- **Not** a diagnostic, screening, grading, or clinical-decision tool; **not** a medical device; **not**
  validated for any clinical use. No artifact is for patient care.
- **Not** patient-facing and **not** medical advice. (Any future patient-facing interpretation would be
  a separate, HIGH-tier, oncologist+advocate-reviewed effort — out of scope here.)
- **No controlled-access or identifiable data**, ever — no dbGaP/EGA individual-level genomics, no
  re-identification, no linkage that could re-identify a patient.
- **Not** a host of raw WSIs we have no right to redistribute — we publish **manifests, derived
  tiles/features where the license permits, splits, code, and metrics**, and point to the authoritative
  archive (GDC/TCIA) for the source pixels.
- **Not** a leaderboard chasing a single headline number — the deliverable is a *trustworthy*
  benchmark (leakage-safe, license-clean, reproducible), explicitly resisting the metric-gaming the
  field already suffers from.
- **Not** dependent on non-commercial assets in its core path — any non-commercial model weight or
  annotation used as a baseline is **clearly segregated and labeled**, never required to use the
  benchmark.

---

## Success metrics (outcomes)

Outcome-centric and beneficiary-first. Download counts alone are **not** success; trustworthy reuse is.

| Metric | Baseline | Target | How measured |
|---|---|---|---|
| Collections with verified open-access status + recorded reuse license | 0 | **100%** of collections used; **0** controlled-access items ingested | License reviewer checklist + allow-list `status: approved`; CI license-gate |
| Per-slide provenance coverage (source collection, accession, license, MPP, scanner, de-id status) | 0 | **100%** of dataset records | Provenance-completeness CI test |
| WSI de-identification re-scan pass rate (label/macro/associated images screened for burned-in PHI) | none | **100%** screened; any positive → **auto-quarantine** (0 redistributed unscanned) | De-id re-scan report + CI quarantine gate |
| Split leakage control (site-confounding) | unmeasured | site-stratified splits **published**; a **site-prediction probe** on the test split shows site is **not** trivially recoverable across the split boundary (report AUC + interpretation) | Site-confounding audit in CI/report |
| Reproducibility of headline baseline | n/a | an **independent re-run** from container + pinned splits reproduces the reported metric within a **pre-declared tolerance** | Third-party reproduction log |
| Honest baseline reporting | n/a | every reported number carries **split, seed(s), variance across seeds, preprocessing hash, and compute provenance**; **no single-seed headline claims** | Results schema + review |
| "Not for clinical use" labeling | n/a | **100%** of model cards + dataset artifacts carry the notice | Artifact lint |
| External adoption (the deed) | 0 | **≥1** external group uses **or** cites the benchmark with a documented instance | Reuse/citation tracking |
| Datasheet + model-card coverage | 0 | **100%** of released datasets have a datasheet; 100% of released baselines have a model card | Release checklist |
| Domain (methods) sign-off before a task/result ships | n/a | **100%** of benchmark tasks, splits, and headline results carry recorded reviewer sign-off | Governance log |

The **defining success outcome** (Definition of Shipped): the benchmark is **publicly released,
methods-reviewed, and reused or cited by at least one group outside the team**, with the
license/de-id/provenance gates verified, splits demonstrably leakage-safe, and the headline baseline
independently reproduced.

---

## Scope

**In scope**

- A **licensing + de-identification gate**: source allow-list (open-access only), per-collection
  license verification, and a defensive WSI label/macro/associated-image PHI re-scan with quarantine.
- A reproducible **WSI data layer** for open collections: download/verify (checksums), per-slide
  provenance manifest, tiling, tissue segmentation, and **MPP/magnification normalization**.
- **Leakage-safe, site-stratified splits** + a site-confounding audit.
- **≥1 benchmark task** (e.g. a tile- or slide-level classification on an approved open cohort) with a
  frozen evaluation protocol, metrics, submission/scoring harness, **honest baselines** (incl. a
  trivial/majority baseline and at least one MIL baseline), **datasheets**, and **model cards**.
- **Reproducibility infrastructure**: container image, pinned splits/preprocessing, seed control,
  variance reporting, and a documented third-party reproduction.
- Optional **license-segregated foundation-model baseline** (clearly labeled if its weights are
  non-commercial / gated).
- Public **release**: versioned, DOI-able artifacts (manifests, splits, code, metrics, cards), with
  attribution to upstream sources.

**Out of scope (explicitly will NOT do)**

- Any **controlled-access or identifiable patient data** (dbGaP, EGA, individual-level genomics,
  PHI). A task proposing this is **refused and flagged**.
- **Clinical/diagnostic deployment, grading, or any patient-care use**; regulatory (FDA/CE) validation.
- **Patient-facing content or medical advice** of any kind (that would be a separate HIGH-tier effort).
- **Re-identification, linkage, or de-anonymization** of any subject.
- **Redistribution of raw WSIs** we lack the right to redistribute — we publish manifests + derived
  artifacts the license permits, and link to the authoritative archive for source pixels.
- **Requiring non-commercial assets** to use the benchmark (they may appear only as labeled, optional
  baselines).
- A 50-collection breadth grab — **depth and verified correctness on one task/cohort first**, breadth
  later and never without the license/de-id gate.

When a request falls in the refused set (e.g. "just pull the matched germline VCFs from dbGaP," or
"skip the de-id re-scan to save time"), the work **stops and surfaces the concern** rather than
proceeding (Elyos refusal guardrail).

---

## Solution approach & architecture

This is a **data/ML-infrastructure** project; the architecture is a pipeline plus a set of versioned,
provenance-carrying artifacts — not a running service.

**Tech stack.** TypeScript/ESM + pnpm for the Elyos-facing tooling, CI, schema validation, and the
allow-list/provenance linters (consistent with the Elyos core conventions). The WSI/ML pipeline is
**Python** (the field's lingua franca: OpenSlide / `openslide-python`, `libvips`/`pyvips`, `tifffile`,
NumPy, PyTorch) invoked as a documented, containerized adapter — kept **out of the agent-neutral
core** (Elyos architecture rule: vendor/tool-specific logic lives in adapters). Containerization via
Docker/Apptainer for reproducible runs. Everything pinned (lockfiles + image digests).

**Components**

1. **Licensing + de-identification gate (`sources/`, `gate/`) — built first.** A safety-critical
   subsystem, not a README note.
   - *Source allow-list* (`sources/allowlist.yml`): every collection has title, custodian/archive
     (GDC/TCIA/Zenodo/Grand-Challenge), URL, accession/collection ID, format (SVS/DICOM/OME-TIFF),
     **open-access tier confirmation**, **reuse license** (e.g. TCGA open-access terms; per-collection
     CC-BY-3.0/4.0; CC0), a rights analysis, and `status: approved|rejected|pending`. **No pixel is
     touched until its entry is `approved`** by the license reviewer.
   - *Controlled-access exclusion*: any dbGaP/EGA/individual-level-genomic/identifiable source is
     categorically `rejected`; the policy text states this and CI enforces it.
   - *De-identification re-scan* (`gate/deid`): before any derived artifact (tile, thumbnail, feature)
     is produced or redistributed, each WSI's **label image, macro image, and associated/auxiliary
     images** are extracted and screened for burned-in text/faces (OCR + heuristic checks); filenames
     and embedded metadata (ImageDescription, DICOM tags) are scanned for identifiers. Any hit →
     **quarantine** + manual review; nothing redistributed unscanned. This is **defense-in-depth**:
     public sets are *expected* de-identified, but we verify rather than assume.

2. **Data layer (`data/`).** Manifest-driven ingest: resolve an approved collection, download from the
   authoritative archive, verify checksums, and write a **per-slide provenance record** (source,
   accession, license, scanner/vendor, **MPP / objective power**, dimensions, de-id status, retrieval
   date, checksum). Tiling + tissue segmentation + **MPP-normalized magnification** (resampling to a
   declared microns-per-pixel so 20×/40× scanners are comparable). Derived artifacts inherit and carry
   provenance and the source license.

3. **Splits + leakage control (`splits/`).** **Site-stratified, patient-grouped** train/val/test
   splits (no patient or site straddles the train/test boundary in a way that leaks), with a fixed
   random seed and a published split file. A **site-confounding audit** trains a probe to predict
   submitting site and reports how recoverable site is — surfacing (not hiding) residual confounding.

4. **Benchmark task + evaluation harness (`benchmark/`).** A frozen task spec (inputs, label
   definition, allowed data, metric(s) — e.g. balanced accuracy / AUROC / Cohen's κ as appropriate),
   a **held-out test protocol** (labels withheld; scoring via the harness), and a results schema that
   **requires** split id, seeds, across-seed variance, preprocessing hash, and compute provenance.

5. **Baselines + model cards (`baselines/`).** A trivial/majority baseline (sanity floor), a classic
   tile-encoder + **MIL** baseline (e.g. ABMIL/CLAM-style), and an optional **license-segregated
   foundation-model** baseline (weights' license recorded; if non-commercial/gated, clearly labeled
   and never required). Each baseline ships a **model card** (intended use = *research benchmarking
   only*, not clinical; training data + license; metrics with variance; known limitations/biases).

6. **Reproducibility (`repro/`).** Container image (digest-pinned), pinned splits + preprocessing,
   seed control, and a documented **third-party reproduction** of the headline baseline within a
   pre-declared tolerance. Small-scale/CPU-feasible runs in CI; heavy runs documented as compute-gated.

7. **Release artifacts.** Versioned, DOI-able bundle (Zenodo or similar): manifests, splits, harness,
   baselines, datasheets, model cards, license/provenance ledger, and a citation file — with upstream
   attribution preserved.

**Key decisions**

- The **license/de-id gate is the first build item and a release gate** — not documentation.
- **Open-access only**; controlled-access is excluded by policy *and* by CI.
- **Leakage-safe splits are a primary artifact**, audited, not an afterthought.
- **Agent-neutral core; Python WSI/ML tooling lives in adapters** (Elyos rule); the Elyos-facing
  layer is the schema, allow-list, provenance linter, and CI.
- **Honest reporting by construction**: the results schema makes single-seed, no-variance, unpinned
  claims impossible to submit.
- **Compute-light value first**: splits, gate, datasheets, harness need no GPU; heavy baselines are
  isolated, compute-gated tasks.

---

## Data, licensing & compliance

**This is the critical section and it leads with the binding cancer guardrails.**

**Binding cancer-domain guardrails (non-negotiable):**

- **Open-access / aggregate / de-identified imaging ONLY.** Controlled-access data (dbGaP, EGA,
  individual-level genomics, any identifiable patient data) is **out of scope** and a task touching it
  is **refused and flagged**. No re-identification or re-identifying linkage, ever.
- **Per-source license verification before use.** No collection is ingested until its open-access tier
  and reuse license are verified and recorded as `approved`. Known non-commercial assets (**COSMIC,
  OncoKB** for molecular annotation; certain foundation-model weights) are **non-commercial** and are
  **not** used in the core path; if used at all they are **segregated and labeled non-commercial**.
- **No medical advice; not for clinical use.** This is research-benchmarking infrastructure. Every
  model card and dataset artifact carries **"not for clinical or diagnostic use; research only."**
  Any future **patient-facing** content would be **HIGH risk-tier**, requiring **oncologist +
  patient-advocate** review and "not medical advice" framing — explicitly out of scope here.
- **Provenance on every assertion/record.** Every dataset record and every reported result carries its
  full provenance (source, accession, license, MPP/scanner, de-id status, retrieval date, checksum;
  for results: split, seeds, preprocessing hash, compute provenance).

**Sources and their licenses (to be verified per collection, recorded in `sources/allowlist.yml`):**

| Source | What | Access tier | License / terms (verify per collection) | Use here |
|---|---|---|---|---|
| **TCGA** diagnostic & tissue slide images (via NCI **GDC**) | H&E WSIs across cancer types | **Open-access** imaging tier (distinct from controlled genomic tier) | TCGA open-access data are usable without restriction; **citation/attribution to the TCGA Research Network required**; controlled genomic tier is separate and **excluded** | Primary WSI source (one cohort first) |
| **CPTAC** pathology images (via **TCIA**) | H&E/IHC WSIs | Open via TCIA | **Per-collection license — typically CC-BY 3.0/4.0**; verify each collection's stated license | Secondary WSI source |
| **TCIA** collections generally | Cancer imaging | Open | **Per-collection** (often CC-BY); verify each | Candidate sources |
| **Camelyon16/17** | Lymph-node metastasis WSIs | Open challenge data | **CC0** (verify current terms) | Optional CC0 task/control |
| **OpenSlide / TIAToolbox / CLAM** | Tooling | OSS | BSD/MIT/GPL (verify per tool) | Pipeline tooling (adapters) |
| **COSMIC, OncoKB** | Molecular annotation | Registered | **Non-commercial** | **Excluded from core**; segregated+labeled if ever used |
| Foundation-model weights (UNI/CONCH/etc.) | Tile encoders | Gated/HF | **Per-model; several non-commercial / usage-agreement** | Optional, **labeled non-commercial**, never required |

**Provenance model.** Each slide → a `Source`/manifest record (source, accession, license, scanner,
MPP, dimensions, de-id status, retrieval date, checksum). Derived tiles/features inherit provenance +
license. Each **result** → a record with split id, seeds, across-seed variance, preprocessing hash,
container digest, and compute provenance. A **provenance-completeness CI test** fails on any record
missing a field; a derived artifact referencing a non-`approved` source fails CI.

**De-identification stance (conservative, defense-in-depth).** Public TCGA/CPTAC imaging is provided
de-identified, **but we re-verify**: extract and screen each WSI's **label/macro/associated images**
for burned-in PHI (OCR + face/text heuristics), scan filenames and embedded metadata (SVS
`ImageDescription`, DICOM tags) for identifiers, and **quarantine** any positive for manual review. No
derived artifact is produced or redistributed from an unscanned slide. We never publish slide-label or
macro images. This is recorded per slide as `deidStatus: passed|quarantined|manual-cleared`.

**Redistribution stance.** We **do not re-host raw WSIs** we lack the right to redistribute. We publish
**manifests + splits + code + metrics + cards**, and **derived tiles/features only where the source
license permits** (e.g. CC-BY/CC0 collections), always pointing to GDC/TCIA for source pixels and
preserving attribution.

**Output licensing.** Code: **MIT**. Datasets/manifests/splits: **CC-BY-4.0** (preserving upstream
attribution; **CC0 where derived solely from CC0 sources** such as Camelyon and where no attribution
obligation is inherited). Docs/datasheets/model cards: **CC-BY-4.0**. Where a derived artifact
incorporates CC-BY source data, **attribution to the source collection is preserved** in the artifact
and its datasheet.

**Privacy / PII.** No PII is collected, stored, or redistributed. The de-id re-scan is the safeguard
against inadvertent PHI in slide labels. No secrets, tokens, or access credentials are written to
logs, receipts, or committed files (Elyos rule).

---

## Quality, review & risk gates

**Risk tier: MEDIUM** (per `docs/good-deed-definition.md`: needs **domain accuracy** —
computational-pathology methodology and honest metrics — and a reviewer with the relevant skill).
Scaffolding/docs are low; **license/de-id/extraction tasks and task/split/result definitions are
medium**. The project is deliberately scoped to **avoid HIGH-tier triggers**: it produces no
patient-facing or clinical-decision content. **Should that ever change, the relevant content becomes
HIGH tier and is gated behind credentialed oncologist + patient-advocate sign-off** — and is out of
scope until then.

**Required reviews before a deed is "done":**

- **Maintainer** review on all PRs (engineering quality, agent-neutral core, adapters isolation, no
  secrets, CI green).
- **License/data reviewer** sign-off before any collection is ingested or any derived artifact is
  released (open-access verified, reuse license recorded, controlled-access excluded, de-id re-scan
  passed). **No approved allow-list entry, no ingest.**
- **Domain (computational-pathology / bioinformatics) reviewer** sign-off on **task definitions,
  splits, the leakage/site-confounding audit, and headline results** (medium-tier accuracy gate).
- **Reproducibility check** before a headline baseline is published (independent re-run within
  tolerance).

**Definition of Shipped (project):** the benchmark is **publicly released, methods-reviewed, and
reused or cited by ≥1 external group**, with license/de-id/provenance gates verified, splits
demonstrably leakage-safe (audit published), the headline baseline independently reproduced, and every
artifact carrying its provenance and the "not for clinical use" notice.

---

## Roadmap & milestones

Phased: the license/de-id gate and data layer first; the leakage-safe benchmark and baselines next;
reproducibility and a second task after; public release + adoption last and gated on a secured
adopter. Compute-light value (gate, splits, harness, datasheets) is front-loaded so it lands even if
GPU compute is never secured.

- **M0 — Licensing, de-identification & scaffold (cold-start).**
  *Goal:* the data-safety spine and an agent-neutral repo skeleton exist before any pixel is ingested.
  *Exit:* allow-list schema + open-only licensing-gate policy merged; **≥3 candidate collections
  analyzed, ≥2 `approved`** (≥1 TCGA open-access cohort or a CC-BY TCIA/CPTAC collection); WSI de-id
  re-scan policy + quarantine spec merged; provenance/manifest schema ratified with the **countable
  record unit** defined; CI live (license-gate + provenance-completeness + "not-for-clinical-use"
  linter); "research only / not for clinical use" framing wired in; **domain reviewer named (hard
  gate; if the seat is empty, M2 cannot start)**. `pnpm build && pnpm test && pnpm lint` green.

- **M1 — First open WSI slice + data layer.**
  *Goal:* one approved open collection ingested with full provenance and a passing de-id re-scan, plus
  the tiling/normalization pipeline.
  *Exit:* one approved collection ingested with checksums + per-slide provenance; **de-id re-scan run,
  report published, 100% screened, any positive quarantined**; tiling + tissue segmentation +
  MPP/magnification normalization working and documented; a **datasheet** for the ingested dataset
  drafted; CI provenance + license gates pass on the real batch.

- **M2 — Benchmark v0: task, leakage-safe splits, baselines.**
  *Goal:* one fully-specified, leakage-safe benchmark task with honest baselines. *(Gated on the
  secured domain reviewer.)*
  *Exit:* **site-stratified, patient-grouped splits** published with a **site-confounding audit**;
  frozen task spec + metrics + evaluation harness; **trivial baseline + ≥1 MIL baseline** reported
  with split/seeds/variance/preprocessing-hash/compute provenance (no single-seed headline); model
  cards published; **domain-reviewer sign-off** on task, splits, and results recorded.

- **M3 — Reproducibility & expansion.**
  *Goal:* prove results reproduce and broaden coverage.
  *Exit:* container image (digest-pinned) + pinned splits/preprocessing; an **independent third-party
  reproduction** of the headline baseline within the pre-declared tolerance, logged; an optional
  **license-segregated foundation-model baseline** (labeled); a **second collection/task** (e.g.
  stain-normalization robustness or a held-out external-site evaluation) through the same gates.

- **M4 — Release, adoption & sustainability (the deed).**
  *Goal:* a public, methods-reviewed release that someone outside the team actually uses.
  *Exit (Definition of Shipped):* versioned, DOI-able release (manifests, splits, harness, baselines,
  datasheets, model cards, license/provenance ledger, citation file); methods review complete; **≥1
  external group uses or cites the benchmark (documented)**; reuse tracking + maintenance/versioning
  plan in effect. *(Gated on a secured adopter/steward — TO BE SECURED.)*

Dependencies flow M0 → M1 → M2 → M3 → M4. The **domain-reviewer decision (made by M0/M1) gates M2**;
the **adopter/steward and any heavy-compute path gate M3 heavy baselines and M4 adoption**.

**Dated acquisition plan + build-vs-mothball rule.** Domain reviewer secured by **2026-08-31** (gates
M2); ≥3 candidate adopters + ≥1 compute path in conversation by **2026-09-30**; adopter/steward
secured by **2026-12-31** (gates M4). **Decision rule:** if no domain reviewer by the M2 entry date,
M2 holds at M1 (data layer + gate) and does not produce headline results. If no adopter is secured by
**~2027-03-31**, the project does **not** ship to no one: it either (a) **pivots** the last-mile
beneficiary — hand the leakage-safe splits + harness + datasheets to an existing open WSI toolkit or a
teaching program as a reference artifact — or (b) **mothballs** to maintenance-only, decision recorded
in governance. The gate + splits + harness remain valuable in either case.

---

## Work breakdown

The itemized, schema-mapped backlog lives in **`TASKS.md`**: ~19 tasks across M0–M4 plus a future
backlog, each mapped to the Elyos Task JSON schema, with per-task acceptance criteria for the most
important items, milestone Definitions of Done, and a complete, schema-valid example Task JSON for the
first M0 task (the licensing-gate policy spec). The first build item is the **open-only licensing +
de-identification gate**, reflecting its status as a hard product requirement and the project's
binding constraint; the **leakage-safe splits + site-confounding audit** (M2) and the **third-party
reproduction** (M3) are sequenced as primary, gated artifacts rather than incidental steps.

---

## Governance, roles & stakeholders

- **Maintainer (Owner): TBD.** Owns architecture, the agent-neutral core, adapter isolation, CI, and
  merge quality.
- **License/data reviewer:** verifies open-access status + reuse license per collection, enforces the
  controlled-access exclusion, and signs off the de-id re-scan before any ingest or release. A **hard
  gate** — no approved allow-list entry, no ingest.
- **Domain (computational-pathology / bioinformatics) reviewer (MEDIUM tier): TO BE SECURED** — signs
  off task definitions, splits, the site-confounding audit, and headline results. Tracked in a
  reviewers ledger with relevant expertise and consent; **version-scoped** sign-off (attaches to a
  specific task/split/result version; edits require re-sign-off). **Disagreement fallback:** the
  domain reviewer holds a veto on whether a task/split/result is **methodologically sound to publish**;
  a maintainer cannot override a "not sound" on substance — the result does not ship, the disagreement
  is logged and escalated to Elyos governance / a second reviewer.
- **Oncologist + patient-advocate reviewers (HIGH tier): not engaged** — required only if scope ever
  expands to patient-facing/clinical-interpretation content (out of scope). Named here so the gate is
  explicit if that line is ever approached.
- **Steward (last-mile owner): TO BE SECURED** — owns the adopter relationship and the hand-off that
  constitutes shipping.
- **Partner / requestor: TO BE SECURED** — a research lab, challenge/benchmark organizer, open WSI
  toolkit, or teaching program (also a candidate compute path).
- **Community / board:** license edge-cases (e.g. ambiguous per-collection TCIA terms; whether to
  include any non-commercial baseline) and the CC-BY-vs-CC0 dataset-license decision go through Elyos
  governance.

---

## Dependencies & integrations

- **External services / archives:** NCI **GDC** (TCGA open-access imaging), **TCIA** (CPTAC + other
  collections), Grand-Challenge/Zenodo (Camelyon, releases), a DOI host (Zenodo) for release.
- **Datasets / sources:** approved open collections per `sources/allowlist.yml` (each with verified
  reuse terms + recorded provenance). **Excluded:** all controlled-access/identifiable data.
- **Tooling (adapters):** OpenSlide/`openslide-python`, `libvips`/`pyvips`, `tifffile`, PyTorch,
  CLAM/TIAToolbox (verify each tool's license); Docker/Apptainer for reproducible runs.
- **Elyos pieces:** `packages/schema` (Task JSON), `CLAUDE.md` (work rules, core/adapter rule, refusal
  guardrails), `docs/good-deed-definition.md` (risk tiers), Elyos governance for license edge-cases.
- **Human/decision dependencies (critical path):** a secured **domain reviewer** (gates M2); a
  secured **adopter/steward** (gates M4); and a **GPU-compute path** (gates M3 heavy baselines — the
  donated coding-agent lane does **not** provide training compute).

---

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|
| Controlled-access / identifiable data pulled in (esp. linked TCGA genomics) | Medium | Critical | Open-only allow-list; controlled-access categorically `rejected` in policy **and** CI; refuse-and-flag any task proposing dbGaP/EGA/individual-level data | License reviewer |
| Burned-in PHI in WSI slide-label / macro images redistributed | Medium | Critical | Defensive de-id re-scan (OCR + heuristics) of label/macro/associated images + metadata; quarantine on any hit; never publish label/macro; no derived artifact from unscanned slide | License reviewer / Maintainer |
| Data-source / site leakage inflates results | High | High | Site-stratified, patient-grouped splits as a primary artifact; **published site-confounding audit**; honest reporting (variance, seeds); domain-reviewer sign-off | Domain reviewer |
| Per-collection license misread (esp. TCIA per-collection terms; non-commercial COSMIC/OncoKB/model weights) | Medium | High | Verify + record reuse license per collection before use; exclude non-commercial assets from core; segregate+label any optional non-commercial baseline; community/board on ambiguity | License reviewer |
| No GPU-compute path → heavy baselines can't run | High | Medium | Front-load compute-light value (gate, splits, harness, datasheets); isolate heavy baselines into compute-gated tasks; pursue a compute sponsor; a trivial+CPU-feasible baseline still anchors the benchmark | Maintainer / Steward |
| No adopter secured → can't reach Definition of Shipped | High | High | Honest `verifiedNeed:false`; dated acquisition plan (reviewer 2026-08-31, adopters 2026-09-30, steward 2026-12-31); pivot-or-mothball rule at ~2027-03-31 rather than ship to no one | Steward / Maintainer |
| No domain reviewer → M2 results unreviewable | Medium | High | Recruit via comp-path labs, toolkit maintainers, challenge organizers; do not publish task/split/result without sign-off (hard gate); hold at M1 | Maintainer |
| Results don't reproduce | Medium | High | Container digest-pinning, pinned splits/preprocessing, seed control, mandatory third-party reproduction before headline publication | Maintainer / Domain reviewer |
| Benchmark misused as a clinical/diagnostic claim | Medium | High | "Not for clinical or diagnostic use" on every model card + dataset; model cards state research-only intended use; non-goals explicit | Maintainer / Domain reviewer |
| Upstream archive changes terms / withdraws a collection | Low | Medium | Record retrieval date + checksum + license snapshot; versioned releases; manifests point to authoritative archive; re-verify on refresh | License reviewer |
| MPP/magnification mismatch silently biases comparisons | Medium | Medium | Explicit MPP normalization to a declared microns-per-pixel; record scanner/MPP per slide; preprocessing hash in results | Domain reviewer |

---

## Security & privacy

**Threat surface.** Inadvertent ingestion of controlled-access/identifiable data; **burned-in PHI in
slide-label/macro images**; re-identification via linkage; license violation in redistributed
artifacts; supply-chain risk in the Python/ML toolchain; secrets/credential leakage; misrepresentation
of a research benchmark as a clinical tool.

**Controls.**
- **Open-only allow-list + controlled-access exclusion** enforced in policy and CI (the top control).
- **De-identification re-scan** of label/macro/associated images + metadata, with quarantine, before
  any derived artifact is produced or redistributed; slide-label/macro images never published.
- **Provenance + license metadata** on every record and artifact; provenance-completeness CI gate;
  derived-artifact-from-non-approved-source fails CI.
- **No PII collected or stored**; no credentials in logs/receipts/commits (Elyos rule); access tokens
  for archives kept out of the repo.
- **Reproducibility = integrity:** digest-pinned containers and lockfiles reduce supply-chain drift;
  dependency + secret scanning in CI.
- **Misuse prevention:** "not for clinical/diagnostic use" notice on every model card + dataset;
  research-only intended-use in model cards; refusal guardrail trips on any controlled-data or
  de-id-skipping request.

**Abuse/misuse prevention.** The refused set — controlled-access/identifiable data, re-identification,
skipping the de-id re-scan, requiring non-commercial assets in the core, or presenting the benchmark
as a clinical tool — is enforced and flagged, not merely documented.

---

## Sustainability & maintenance

After delivery, a named **maintenance rotation** owns the harness, gate, and CI; the **steward** owns
the adopter relationship and reuse tracking. Releases are **versioned and DOI-able**; each carries a
**license/provenance snapshot** (retrieval dates, checksums, per-collection license at time of
release) so a withdrawn or re-licensed upstream collection is detectable and re-verifiable on refresh.
The license/de-id gate is maintained as living policy + tests, expanded as new collections are
proposed. Outcomes tracked are **reuse/citation, reproductions, and gate integrity** — not vanity
download counts. Breadth (more collections/tasks) follows a documented, gated process and only after
the first task/release is stable. A scheduled **license re-verification cadence** re-checks
per-collection terms (which can change) and re-snapshots provenance.

---

## Open questions

- **Dataset license: CC-BY-4.0 vs CC0?** CC-BY preserves required upstream attribution (TCGA citation,
  CC-BY TCIA collections) and is the safe default; CC0 is only defensible for artifacts derived
  **solely** from CC0 sources (e.g. Camelyon) with no inherited attribution obligation. Needs a
  per-artifact governance call; the conservative default is CC-BY-4.0 unless provably CC0-only.
- **Which first cohort/task?** A specific TCGA open-access cohort + a tile/slide-level task with a
  clean, verifiable label, vs a CC-BY CPTAC collection, vs CC0 Camelyon as a control. Decision should
  optimize for: verified open license, label quality, manageable size for compute-light baselines, and
  availability of a domain reviewer for that cancer type. **Decided by M1; gates M2.**
- **Compute path.** Is there a donated/funded GPU path for the MIL/foundation-model baselines, and (if
  funded) what is the **hard per-task budget cap**? Without one, the benchmark still ships its gate +
  splits + harness + trivial baseline; heavy baselines wait.
- **Any non-commercial foundation-model baseline at all?** Including a labeled non-commercial encoder
  baseline aids comparison but risks confusing reuse terms; governance decision on whether to include
  it (and never require it).
- **Derived-artifact redistribution per collection.** For each approved collection, may we redistribute
  derived tiles/features, or only manifests + links? (Per-license; recorded in the allow-list.)
- **How far does the de-id re-scan go** — OCR + metadata is baseline; do we add a vision check for
  faces/handwriting on macro images, and what is the manual-review SLA on a quarantine hit?

## References

- Proposal / portfolio entry: `planning/ROADMAP.md` (Track 8b — `pathology-image-benchmarks`)
- Elyos work rules, core/adapter rule & refusal guardrails: `CLAUDE.md`
- Good-deed definition & risk tiers: `docs/good-deed-definition.md`
- Task JSON schema: `packages/schema/src/schemas.ts`
- Sibling Elyos plans for house style: `planning/projects/revolutionary-patriots-kg/{PLAN,TASKS}.md`,
  `planning/projects/public-official-guide/PLAN.md`
- Data archives: NCI Genomic Data Commons (TCGA open-access imaging); The Cancer Imaging Archive
  (TCIA; CPTAC + per-collection licenses); Grand Challenge / Camelyon (CC0)
- Methodology: Howard et al., "The impact of site-specific digital histology signatures on deep
  learning model accuracy and bias" (*Nature Communications*, 2021) — motivates site-stratified splits
- Reporting/documentation: "Datasheets for Datasets" (Gebru et al.); "Model Cards for Model Reporting"
  (Mitchell et al.)
- Non-commercial assets flagged in guardrails: COSMIC, OncoKB (non-commercial terms)

---

## Appendix A — Improvements applied

The 25 improvements below were identified during a self-review pass and **applied** to the plan above
(and to `TASKS.md`). Each names the gap and the concrete change made.

1. **Compute is a first-class dependency, not an assumption.** Made explicit that the donated lane is
   a *coding agent preparing PRs*, not GPU training, and front-loaded all compute-light value (gate,
   splits, harness, datasheets) so the deed lands even if no GPU sponsor is secured. (*Exec summary,
   Dependencies, Risks, M2/M3 task notes.*)
2. **Site-leakage made a primary, audited artifact.** Added site-stratified, patient-grouped splits
   **plus a site-confounding probe audit** as a named deliverable and success metric, citing the
   Howard et al. (2021) confound — rather than treating splitting as incidental.
3. **Defensive WSI de-identification re-scan.** Added a label/macro/associated-image PHI re-scan
   (OCR + heuristics + metadata/filename scan) with a quarantine workflow — the well-known burned-in-
   PHI failure mode in SVS/DICOM slides — instead of assuming public sets are clean.
4. **Open-access tier vs controlled genomic tier disambiguated.** Explicitly separated TCGA's
   open-access *imaging* tier from its controlled *genomic* tier, so the guardrail is precise and
   reviewers don't conflate them.
5. **Per-collection (not per-source-family) license verification.** Encoded that TCIA/CPTAC terms
   vary per collection (often CC-BY 3.0/4.0), so each collection is verified individually.
6. **Non-commercial assets segregated from the core path.** COSMIC/OncoKB and non-commercial
   foundation-model weights are excluded from the required path and only ever appear as labeled,
   optional baselines — protecting downstream reuse.
7. **"Not for clinical or diagnostic use" as an enforced lint, not a footer.** Added a CI linter
   requiring the notice on every model card + dataset, plus a success metric.
8. **Honest-reporting-by-construction results schema.** The results record *requires* split id, seeds,
   across-seed variance, preprocessing hash, and compute provenance — making single-seed, unpinned
   headline claims structurally impossible to submit.
9. **Reproducibility operationalized.** Added digest-pinned containers, pinned splits/preprocessing,
   and a **mandatory independent third-party reproduction within a pre-declared tolerance** before any
   headline baseline is published (M3 exit + metric).
10. **MPP/magnification normalization called out.** Added explicit microns-per-pixel normalization and
    per-slide MPP/scanner recording, so 20×/40× scanners are comparable and don't silently bias results.
11. **No-rehosting redistribution stance.** Clarified we publish manifests + splits + code + metrics +
    derived artifacts *where the license permits*, linking to GDC/TCIA for source pixels — avoiding a
    redistribution-rights violation.
12. **Trivial/majority baseline mandated as a sanity floor.** Required alongside the MIL baseline so
    reported gains are interpretable, not just a single fancy model.
13. **Dated acquisition plan + build-vs-mothball/pivot rule.** Replaced an open-ended `TBD` with dated
    reviewer/adopter/compute milestones and an explicit pivot-or-mothball decision at ~2027-03-31.
14. **Domain-reviewer hard gate on M2.** Made the medium-tier methods reviewer a hard entry gate for
    publishing any task/split/result, with a version-scoped sign-off and a disagreement/veto fallback.
15. **HIGH-tier boundary stated precisely.** Documented that the project deliberately avoids
    patient-facing/clinical content (which would be HIGH tier, oncologist + advocate-gated) and named
    the gate so the line is explicit if ever approached.
16. **Agent-neutral core / Python-in-adapters separation.** Applied the Elyos core/adapter rule:
    Python WSI/ML tooling is an isolated adapter; the Elyos-facing layer is schema + allow-list +
    provenance linter + CI.
17. **Countable provenance "record" unit defined.** So the 100%-provenance CI gate is mechanically
    checkable (mirrors the sibling-plan convention) rather than aspirational.
18. **Funded-lane budget cap wired into heavy tasks.** Heavy training baselines explicitly note they
    become `funded` tasks requiring `fundedBudgetUsd` — satisfying the schema's funded-lane constraint.
19. **License-reverification cadence.** Added a scheduled re-check of per-collection terms (which can
    change) plus a per-release license/provenance snapshot, so a withdrawn/re-licensed upstream is
    detectable.
20. **CC-BY-vs-CC0 dataset-license decision made explicit and conservative.** Default CC-BY-4.0 to
    preserve TCGA/CC-BY attribution; CC0 only for artifacts derived *solely* from CC0 sources.
21. **External-site / stain-normalization robustness evaluation.** Added a second-collection task that
    tests generalization across scanner/site or stain normalization — the realistic failure mode for
    pathology models — rather than only in-distribution accuracy.
22. **Refusal guardrail given concrete trigger examples.** Named the exact requests that must be
    refused-and-flagged (pull dbGaP germline VCFs; skip the de-id re-scan) so the guardrail is testable.
23. **Distal-vs-direct beneficiary honesty.** Stated plainly that patients are helped *distally* via
    methodological trustworthiness, and that this project performs **no** clinical validation —
    avoiding overclaiming.
24. **Datasheets + model cards as required release artifacts.** Adopted Datasheets-for-Datasets and
    Model-Cards conventions with coverage metrics, not ad-hoc READMEs.
25. **Outcome metrics over vanity metrics.** Success is reuse/citation, reproductions, and gate
    integrity — explicitly *not* download counts — consistent with the Elyos "delivered, not merged" bar.

---

## Review sign-off

A completeness + correctness review was performed against the PLAN_SPEC 17-section structure, the
Elyos CLAUDE.md work rules, the good-deed definition + risk tiers, the binding cancer guardrails, and
the Task JSON schema. Findings and fixes:

**Completeness.** All 17 required H2 sections are present and in order. `TASKS.md` carries the
"How these tasks map to Elyos" field mapping, one section per milestone (M0–M4) with task tables,
per-task acceptance criteria for the 2–4 most important tasks per milestone, milestone Definitions of
Done, a backlog, and a complete example Task JSON. Task count: **19 across M0–M4 + 7 backlog** (within
the 12–20 target for scheduled work). *No gaps found.*

**Schema validity.** The example Task JSON was validated programmatically: all 17 required fields
present, no extra properties (schema is `additionalProperties: false`), all enum-constrained fields
(`type`, `lane`, `priority`, `riskTier`, `deliverable`, `tokenEstimate`, `status`) within their
enums, `acceptanceCriteria` non-empty (6 items, satisfies `minItems: 1`), `verifiedNeed: false`
(honest — no partner secured). Since the example task is `lane: donated`, no `fundedBudgetUsd` is
required; the conditional funded-lane requirement is documented for heavy tasks. **PASS.**

**Correctness fixes applied during review.**
- Verified that **no task is mislabeled `low` risk where a license/de-id/methods gate applies** —
  ingest, de-id, splits, task/result definitions are all `medium`; only pure scaffold/docs/outreach
  are `low`. Consistent with the risk-tier rules.
- Confirmed the plan claims **no HIGH-tier work** and that this is *justified* (no patient-facing or
  clinical content), with the HIGH-tier gate named for the out-of-scope case — so the medium tier is
  not an under-classification.
- Ensured every data/ingest task is bound by the four standing guardrails (approved-allow-list-only,
  controlled-access refusal, de-id-pass-before-derivative, provenance + not-for-clinical-use) restated
  at the top of `TASKS.md`.
- Reconciled the dated acquisition plan across Exec summary, Problem & beneficiaries, Roadmap, and
  Risks (reviewer 2026-08-31; adopters/compute 2026-09-30; steward 2026-12-31; pivot/mothball
  ~2027-03-31) — dates are now consistent in all four places.
- Confirmed `outputLicense` guidance is internally consistent (MIT code / CC-BY-4.0 data+docs /
  CC0 only for CC0-derived-only artifacts) between PLAN §Data and TASKS field mapping.

**Honesty check.** `verifiedNeed: false` throughout; partner, steward, domain reviewer, and compute
path all marked TO BE SECURED with a dated plan and a build-vs-mothball/pivot fallback; patients
identified as *distal* beneficiaries with **no** clinical-validation claim. No invented partner.

**Residual open items (tracked in *Open questions*, require a human decision):** first cohort/task
selection; whether any compute (donated vs funded, with cap) path exists; CC-BY-vs-CC0 per artifact;
whether to include any non-commercial FM baseline; per-collection derived-artifact redistribution
rights; depth of the de-id vision check + quarantine SLA.

**Verdict:** the plan is internally consistent, schema-valid, guardrail-compliant, and ready for
maintainer review. The single most important human decision before M1 is **securing a
computational-pathology domain reviewer** (hard gate for M2); the single highest residual risk is the
**unsecured GPU-compute path** for heavy baselines (mitigated by front-loading compute-light value).

