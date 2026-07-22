# TASKS — pathology-image-benchmarks

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

Itemized backlog for the open whole-slide-image (WSI) computational-pathology benchmark suite. See
[`PLAN.md`](./PLAN.md) for context, the binding cancer guardrails, the licensing/de-identification
gate, and the roadmap (M0–M4).

## How these tasks map to Hee-Lee Oss

Each task below becomes a Hee-Lee Oss **Task JSON** validated against `packages/schema/src/schemas.ts`.
Field mapping:

- **id** — stable `pathology-image-benchmarks-<area>-NNN` (the table ID).
- **title** — the table Title.
- **project** — `pathology-image-benchmarks`.
- **type** — one of `code | research | writing | data | design-spec | maintenance` (table "Type").
- **lane** — `donated` for all tasks here. The donated lane is a **coding agent preparing PRs**, not
  GPU training; any heavy training run that needs metered compute would be a **`funded`** task and
  **must** add `fundedBudgetUsd` (a hard per-task budget cap). Heavy-baseline tasks below note this.
- **priority** — `high | medium | low`.
- **domain** — e.g. `["cancer-research","computational-pathology","open-data","machine-learning","reproducibility"]`.
- **riskTier** — `low | medium | high`. License/de-id/extraction and task/split/result definition tasks
  are **medium** (domain-accuracy + license gate); scaffolding/docs are **low**. **No `high` tasks
  exist here** — the project produces no patient-facing/clinical content; if scope ever crossed into
  patient-facing interpretation, those tasks would be `high` (oncologist + advocate sign-off).
- **urgent** — boolean (default `false`).
- **deliverable** — `pr | dataset | document | translation` (table "Deliverable").
- **tokenEstimate** — `small | medium | large` (table "Size").
- **status** — `open | in-progress | review | delivered | done` (start `open`).
- **context / objective / acceptanceCriteria[] / resources[] / output** — per task.
- **requestor** — `jdev1977` / beneficiary class (comp-path researchers) until a named partner is secured.
- **verifiedNeed** — **`false`** while no committed adopter/steward is secured (honest; the *gap* is
  real, but the last-mile beneficiary is TO BE SECURED).
- **outputLicense** — `MIT` (code), `CC-BY-4.0` (datasets/manifests/splits/docs preserving upstream
  attribution), or `CC0-1.0` (only for artifacts derived **solely** from CC0 sources with no inherited
  attribution obligation).

> **Standing guardrails on every data/ingest task (binding):**
> 1. **No source may be touched** until its `sources/allowlist.yml` entry is `approved` by the license
>    reviewer (open-access verified, reuse license recorded).
> 2. Any task proposing to ingest, link, or re-identify **controlled-access or identifiable data**
>    (dbGaP, EGA, individual-level genomics, PHI) is **refused and flagged** — out of scope, full stop.
> 3. **No derived artifact** (tile, thumbnail, feature) is produced or redistributed from a WSI that
>    has not **passed the de-identification re-scan**; any positive is **quarantined**.
> 4. Every dataset record and every reported result carries **full provenance**; every model card and
>    dataset carries **"not for clinical or diagnostic use; research only."**

---

## Milestone M0 — Licensing, de-identification & scaffold (cold-start)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| pathology-image-benchmarks-license-001 | Source allow-list schema + open-only licensing-gate policy | design-spec | small | medium | document | — | License reviewer |
| pathology-image-benchmarks-license-002 | Verify & record first ≥3 candidate open collections (≥2 approved) | research | medium | medium | document | pathology-image-benchmarks-license-001 | License reviewer |
| pathology-image-benchmarks-deid-001 | WSI de-identification re-scan policy + label/macro PHI quarantine spec | design-spec | small | medium | document | — | License reviewer |
| pathology-image-benchmarks-prov-001 | Provenance/manifest + results schema (countable record unit defined) | design-spec | small | low | document | pathology-image-benchmarks-license-001 | Maintainer |
| pathology-image-benchmarks-scaffold-001 | Monorepo + CI scaffold (license-gate + provenance + not-for-clinical-use linters) | code | medium | low | pr | pathology-image-benchmarks-prov-001, pathology-image-benchmarks-license-001 | Maintainer |
| pathology-image-benchmarks-partner-001 | Domain-reviewer, adopter & compute-path outreach | research | small | low | document | — | Maintainer |

**Acceptance criteria (key M0 tasks)**

- **pathology-image-benchmarks-license-001**
  - `sources/allowlist.yml` schema defines: title, custodian/archive, URL, accession/collection ID,
    format, **open-access tier confirmation**, **reuse license**, rights analysis, retrieval date,
    checksum, and `status: approved|rejected|pending`.
  - Policy text states **controlled-access / identifiable data (dbGaP, EGA, individual-level genomics,
    PHI) is categorically `rejected`**, and **non-commercial assets (COSMIC, OncoKB, non-commercial
    model weights) are excluded from the core path** (segregated + labeled only if ever used).
  - Per-collection (not source-family) license verification is required (TCIA terms vary per collection).
- **pathology-image-benchmarks-license-002**
  - ≥3 collections analyzed with recorded open-access + reuse-license determination; **≥2 `approved`**.
  - ≥1 approved source is a **TCGA open-access imaging cohort** (with TCGA citation requirement noted)
    **or** a **CC-BY TCIA/CPTAC collection** (CC-BY version recorded); any CC0 set (e.g. Camelyon)
    flagged CC0.
  - Zero controlled-access items approved; any ambiguous per-collection term routed to the board.
- **pathology-image-benchmarks-deid-001**
  - Policy specifies extraction + screening of each WSI's **label, macro, and associated/auxiliary
    images** (OCR + text/face heuristics) plus filename + embedded-metadata (SVS `ImageDescription`,
    DICOM tags) scanning for identifiers.
  - Defines `deidStatus: passed|quarantined|manual-cleared`, the **quarantine workflow**, a
    manual-review SLA, and the rule that **no derived artifact is produced from an unscanned slide**
    and **slide-label/macro images are never published**.
- **pathology-image-benchmarks-prov-001**
  - Manifest record fields fixed (source, accession, license, scanner/vendor, **MPP/objective power**,
    dimensions, de-id status, retrieval date, checksum); results record fields fixed (split id, seeds,
    across-seed variance, preprocessing hash, container digest, compute provenance).
  - The countable **"record" unit** is defined explicitly so the 100%-provenance CI gate is checkable.
- **pathology-image-benchmarks-scaffold-001**
  - CI fails on any dataset/result record missing a provenance field.
  - CI rejects any derived artifact referencing a source not `approved` in the allow-list.
  - CI linter asserts every model card + dataset carries the "not for clinical or diagnostic use" notice.
  - `pnpm build && pnpm test && pnpm lint` green; Python WSI tooling isolated as an adapter (not in core).

**M0 Definition of Done:** allow-list schema + open-only policy merged; **≥3 collections analyzed, ≥2
approved**; de-id re-scan policy + quarantine spec merged; provenance/results schema ratified **with
the countable record unit defined**; CI license + provenance + not-for-clinical-use gates live;
"research only" framing wired in; outreach initiated with status logged; **a qualified
computational-pathology domain reviewer named (hard gate — if the seat is empty, M2 cannot start;
escalate per the PLAN.md decision rule)**. `pnpm build && pnpm test && pnpm lint` green.

---

## Milestone M1 — First open WSI slice + data layer

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| pathology-image-benchmarks-ingest-001 | WSI ingest + checksum + per-slide provenance manifest (one approved collection) | code | medium | medium | pr | pathology-image-benchmarks-license-002, pathology-image-benchmarks-scaffold-001 | License reviewer + Maintainer |
| pathology-image-benchmarks-deid-002 | Run de-id re-scan on the ingested batch; publish quarantine report | data | medium | medium | dataset | pathology-image-benchmarks-ingest-001, pathology-image-benchmarks-deid-001 | License reviewer |
| pathology-image-benchmarks-tiling-001 | Tiling + tissue segmentation + MPP/magnification normalization pipeline | code | medium | medium | pr | pathology-image-benchmarks-ingest-001 | Maintainer + Domain reviewer |
| pathology-image-benchmarks-datasheet-001 | Datasheet for the ingested dataset (provenance + license + de-id) | writing | small | medium | document | pathology-image-benchmarks-deid-002, pathology-image-benchmarks-tiling-001 | Domain reviewer |

**Acceptance criteria (key M1 tasks)**

- **pathology-image-benchmarks-ingest-001**
  - Ingests only from the **approved** collection's authoritative archive (GDC/TCIA); **does not
    re-host** raw WSIs we lack redistribution rights to.
  - **100%** of ingested slides carry a resolvable per-slide provenance record (source, accession,
    license, scanner, MPP, dimensions, retrieval date, checksum).
  - Checksums verified against the archive; mismatches fail, not silently re-download.
- **pathology-image-benchmarks-deid-002**
  - **100%** of ingested slides screened (label/macro/associated images + filenames + metadata).
  - Any positive is **quarantined** and excluded from all derived artifacts pending manual review;
    a report lists screened/passed/quarantined counts and the screening method.
  - No slide-label or macro image is published anywhere in the artifacts.
- **pathology-image-benchmarks-tiling-001**
  - Tiling + tissue segmentation reproducible from a pinned config; **MPP-normalized** to a declared
    microns-per-pixel so different scanners/magnifications are comparable.
  - Per-slide MPP/scanner recorded; the preprocessing config produces a stable **preprocessing hash**.
  - Derived tiles inherit provenance + source license; only redistributed where the license permits.

**M1 Definition of Done:** one approved collection ingested with 100% provenance + verified checksums;
de-id re-scan run with a published report (100% screened, positives quarantined); tiling +
MPP-normalization pipeline reproducible and documented; a datasheet drafted; CI provenance + license
gates pass on the real batch.

---

## Milestone M2 — Benchmark v0: task, leakage-safe splits, baselines

> **Entry gate (hard):** the computational-pathology **domain reviewer is secured** (per the PLAN.md
> dated plan, by 2026-08-31). No task/split/result is published without sign-off.

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| pathology-image-benchmarks-splits-001 | Site-stratified, patient-grouped splits + site-confounding audit | data | medium | medium | dataset | pathology-image-benchmarks-tiling-001 | Domain reviewer |
| pathology-image-benchmarks-task-001 | Benchmark task v0 spec + metrics + evaluation protocol | design-spec | small | medium | document | pathology-image-benchmarks-splits-001 | Domain reviewer |
| pathology-image-benchmarks-baseline-001 | Trivial + ≥1 MIL baseline with model cards (honest reporting) | code | large | medium | pr | pathology-image-benchmarks-task-001 | Domain reviewer + Maintainer |
| pathology-image-benchmarks-harness-001 | Submission / held-out scoring harness | code | medium | medium | pr | pathology-image-benchmarks-task-001 | Maintainer |

**Acceptance criteria (key M2 tasks)**

- **pathology-image-benchmarks-splits-001**
  - Splits are **site-stratified and patient-grouped**: no patient straddles train/test, and
    submitting **site** is allocated so the split does not leak site across the train/test boundary.
  - A **site-confounding audit** trains a probe to predict submitting site and reports recoverability
    (e.g. AUC) with interpretation; residual confounding is surfaced, not hidden.
  - Split files are fixed, seeded, versioned, and published; **domain-reviewer sign-off** recorded.
- **pathology-image-benchmarks-task-001**
  - Task fully specified: inputs, label definition + label provenance, allowed data, and metric(s)
    (e.g. balanced accuracy / AUROC / Cohen's κ as appropriate to the task) with rationale.
  - Evaluation protocol frozen (held-out test, labels withheld, scoring via the harness); explicitly
    framed as **research benchmarking, not clinical use**.
- **pathology-image-benchmarks-baseline-001**
  - Reports a **trivial/majority baseline** (sanity floor) and **≥1 MIL baseline**.
  - Every reported number carries **split id, seed(s), across-seed variance, preprocessing hash, and
    compute provenance** — **no single-seed headline claims**.
  - Each baseline ships a **model card**: intended use = *research benchmarking only, not clinical*;
    training data + license; metrics with variance; known limitations/biases (incl. site effects).
  - **If run as a heavy/metered job**, it is split into a `funded` task with a `fundedBudgetUsd` cap;
    the donated-lane portion is the code + a CPU-feasible/trivial baseline.

**M2 Definition of Done:** leakage-safe splits + site-confounding audit published; frozen task spec +
metrics; scoring harness working; trivial + ≥1 MIL baseline reported with full honest provenance and
model cards; **domain-reviewer sign-off** on task, splits, and results recorded.

---

## Milestone M3 — Reproducibility & expansion

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| pathology-image-benchmarks-repro-001 | Container (digest-pinned) + third-party reproduction of headline baseline | code | medium | medium | pr | pathology-image-benchmarks-baseline-001 | Maintainer + Domain reviewer |
| pathology-image-benchmarks-fm-001 | License-segregated foundation-model baseline (labeled, optional) | code | large | medium | pr | pathology-image-benchmarks-baseline-001 | Domain reviewer + License reviewer |
| pathology-image-benchmarks-data-002 | Second collection/task (stain-norm robustness or external-site eval) | data | large | medium | dataset | pathology-image-benchmarks-splits-001, pathology-image-benchmarks-license-002 | Domain + License reviewers |

**Acceptance criteria (key M3 tasks)**

- **pathology-image-benchmarks-repro-001**
  - Container image is **digest-pinned**; splits + preprocessing + seeds pinned.
  - An **independent re-run** (different operator/machine) reproduces the headline baseline within a
    **pre-declared tolerance**; the reproduction is logged with environment + result.
- **pathology-image-benchmarks-fm-001**
  - The foundation-model encoder's **weights license is recorded**; if **non-commercial / gated**, it
    is **clearly labeled** and **never required** to use the benchmark (the permissive path stands alone).
  - Reported with the same honest provenance (split/seeds/variance/preprocessing-hash/compute) + model card.
  - **Heavy/metered runs** become `funded` tasks with `fundedBudgetUsd` caps.
- **pathology-image-benchmarks-data-002**
  - Second collection passes the **license gate + de-id re-scan** before any derived artifact; 100%
    provenance maintained.
  - Adds either a **stain-normalization robustness** evaluation or an **external-site (held-out scanner/
    site) generalization** evaluation, with results reported honestly.

**M3 Definition of Done:** headline baseline independently reproduced within tolerance from a pinned
container; optional license-segregated FM baseline labeled; a second collection/task through the full
gate with a robustness or external-site evaluation; all new artifacts carry provenance + model cards.

---

## Milestone M4 — Release, adoption & sustainability (the deed)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| pathology-image-benchmarks-release-001 | Public versioned/DOI release + methods review | writing | medium | medium | document | pathology-image-benchmarks-repro-001 | Maintainer + Domain reviewer |
| pathology-image-benchmarks-partner-002 | Secure ≥1 adopter + reuse/citation tracking | research | medium | low | document | pathology-image-benchmarks-release-001, pathology-image-benchmarks-partner-001 | Maintainer |
| pathology-image-benchmarks-sustain-001 | Sustainability, versioning + license-reverification cadence plan | writing | small | low | document | pathology-image-benchmarks-partner-002 | Maintainer |

**Acceptance criteria (key M4 tasks)**

- **pathology-image-benchmarks-release-001**
  - Versioned, **DOI-able** bundle: manifests, splits, harness, baselines, datasheets, model cards,
    license/provenance ledger, citation file; upstream attribution (TCGA citation, CC-BY collections)
    preserved.
  - Methods review complete; every artifact carries the "not for clinical or diagnostic use" notice.
- **pathology-image-benchmarks-partner-002**
  - A named external group (lab, challenge/toolkit, or teaching program) **uses or cites** the
    benchmark, with a documented instance; reuse/citation tracking in place.

**M4 Definition of Done (project "shipped"):** public versioned/DOI release; methods review complete;
license/de-id/provenance gates verified; splits demonstrably leakage-safe (audit published); headline
baseline independently reproduced; **≥1 external group uses or cites the benchmark (documented)**;
reuse tracking + sustainability/versioning + license-reverification cadence in effect.

---

## Backlog / future (sized, unscheduled)

| ID | Title | Type | Size | Risk | Deliverable | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| pathology-image-benchmarks-dicom-001 | DICOM-WSI ingest + OME-Zarr conversion path | code | medium | low | pr | Interop; verify per-collection license |
| pathology-image-benchmarks-task-002 | Slide-level grading/subtyping task (research-only label) | data | large | medium | dataset | Research label; not clinical grading |
| pathology-image-benchmarks-fairness-001 | Site/scanner disparity + bias analysis across splits | research | medium | medium | document | Extends site-confounding audit |
| pathology-image-benchmarks-seg-001 | Tissue/tumor segmentation task (CC0 Camelyon control) | data | large | medium | dataset | CC0 source → CC0 derivatives possible |
| pathology-image-benchmarks-leaderboard-001 | Hosted public leaderboard with held-out test | code | large | low | pr | Depends on steward hosting |
| pathology-image-benchmarks-features-001 | Precomputed tile-feature release (license-permitting) | data | large | low | dataset | Only where source license permits redistribution |
| pathology-image-benchmarks-modelcard-001 | Model-card template + checklist for external submissions | writing | small | low | document | Standardize honest reporting |

---

## Generated task index

> Auto-generated by Hee-Lee Oss task-decomposition agent on 2026-06-29. All 27 tasks below are
> schema-validated (`packages/schema/validateTask`). Seeds kept; fan-out dimensions: none
> (all tasks enumerated explicitly in the backlog tables above).

| File | Title | Milestone | Type | Lane | Status |
| --- | --- | --- | --- | --- | --- |
| tasks/pathology-image-benchmarks-license-001.json | Source allow-list schema + open-only licensing-gate policy | M0 | design-spec | donated | open |
| tasks/pathology-image-benchmarks-license-002.json | Verify & record first ≥3 candidate open collections (≥2 approved) | M0 | research | donated | open |
| tasks/pathology-image-benchmarks-deid-001.json | WSI de-identification re-scan policy + label/macro PHI quarantine spec | M0 | design-spec | donated | open |
| tasks/pathology-image-benchmarks-prov-001.json | Provenance/manifest + results schema (countable record unit defined) | M0 | design-spec | donated | open |
| tasks/pathology-image-benchmarks-scaffold-001.json | Monorepo + CI scaffold (license-gate + provenance + not-for-clinical-use linters) | M0 | code | donated | open |
| tasks/pathology-image-benchmarks-partner-001.json | Domain-reviewer, adopter & compute-path outreach | M0 | research | donated | open |
| tasks/pathology-image-benchmarks-ingest-001.json | WSI ingest + checksum + per-slide provenance manifest (one approved collection) | M1 | code | donated | open |
| tasks/pathology-image-benchmarks-deid-002.json | Run de-id re-scan on the ingested batch; publish quarantine report | M1 | data | donated | open |
| tasks/pathology-image-benchmarks-tiling-001.json | Tiling + tissue segmentation + MPP/magnification normalization pipeline | M1 | code | donated | open |
| tasks/pathology-image-benchmarks-datasheet-001.json | Datasheet for the ingested dataset (provenance + license + de-id) | M1 | writing | donated | open |
| tasks/pathology-image-benchmarks-splits-001.json | Site-stratified, patient-grouped splits + site-confounding audit | M2 | data | donated | open |
| tasks/pathology-image-benchmarks-task-001.json | Benchmark task v0 spec + metrics + evaluation protocol | M2 | design-spec | donated | open |
| tasks/pathology-image-benchmarks-baseline-001.json | Trivial + ≥1 MIL baseline with model cards (honest reporting) | M2 | code | donated | open |
| tasks/pathology-image-benchmarks-harness-001.json | Submission / held-out scoring harness | M2 | code | donated | open |
| tasks/pathology-image-benchmarks-repro-001.json | Container (digest-pinned) + third-party reproduction of headline baseline | M3 | code | donated | open |
| tasks/pathology-image-benchmarks-fm-001.json | License-segregated foundation-model baseline (labeled, optional) | M3 | code | donated | open |
| tasks/pathology-image-benchmarks-data-002.json | Second collection/task (stain-norm robustness or external-site eval) | M3 | data | donated | open |
| tasks/pathology-image-benchmarks-release-001.json | Public versioned/DOI release + methods review | M4 | writing | donated | open |
| tasks/pathology-image-benchmarks-partner-002.json | Secure ≥1 adopter + reuse/citation tracking | M4 | research | donated | open |
| tasks/pathology-image-benchmarks-sustain-001.json | Sustainability, versioning + license-reverification cadence plan | M4 | writing | donated | open |
| tasks/pathology-image-benchmarks-dicom-001.json | DICOM-WSI ingest + OME-Zarr conversion path | Backlog | code | donated | open |
| tasks/pathology-image-benchmarks-task-002.json | Slide-level grading/subtyping task (research-only label) | Backlog | data | donated | open |
| tasks/pathology-image-benchmarks-fairness-001.json | Site/scanner disparity + bias analysis across splits | Backlog | research | donated | open |
| tasks/pathology-image-benchmarks-seg-001.json | Tissue/tumor segmentation task (CC0 Camelyon control) | Backlog | data | donated | open |
| tasks/pathology-image-benchmarks-leaderboard-001.json | Hosted public leaderboard with held-out test | Backlog | code | donated | open |
| tasks/pathology-image-benchmarks-features-001.json | Precomputed tile-feature release (license-permitting) | Backlog | data | donated | open |
| tasks/pathology-image-benchmarks-modelcard-001.json | Model-card template + checklist for external submissions | Backlog | writing | donated | open |

---

## Example task JSON

Schema-valid Task JSON for the first M0 task (`pathology-image-benchmarks-license-001`):

```json
{
  "id": "pathology-image-benchmarks-license-001",
  "title": "Source allow-list schema + open-only licensing-gate policy",
  "project": "pathology-image-benchmarks",
  "type": "design-spec",
  "lane": "donated",
  "priority": "high",
  "domain": ["cancer-research", "computational-pathology", "open-data", "machine-learning", "reproducibility"],
  "riskTier": "medium",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "small",
  "status": "open",
  "context": "An open whole-slide-image benchmark suite for computational pathology must build only on OPEN-ACCESS cancer imaging (open-tier TCGA diagnostic/tissue slides via NCI GDC; CPTAC and other CC-licensed collections via TCIA; CC0 challenge sets like Camelyon). Controlled-access and identifiable patient data (dbGaP, EGA, individual-level genomics, PHI) are categorically OUT OF SCOPE, and non-commercial assets (COSMIC, OncoKB, certain foundation-model weights) must be excluded from the core path. Before any pixel is ingested, the project needs a source allow-list and an open-only licensing-gate policy that admits a collection only after its open-access status and reuse license are verified and recorded. See PLAN.md (Data, licensing & compliance).",
  "objective": "Define the sources/allowlist.yml schema and the open-only licensing-gate policy: the fields recorded per collection (title, custodian/archive, URL, accession/collection ID, format, open-access tier confirmation, reuse license, rights analysis, retrieval date, checksum, status), the categorical rejection of controlled-access/identifiable data, the exclusion of non-commercial assets from the core path, and the requirement that no source is touched until its entry is approved by the license reviewer.",
  "acceptanceCriteria": [
    "sources/allowlist.yml schema defines all required per-collection fields plus status: approved|rejected|pending.",
    "Policy text categorically rejects controlled-access/identifiable data (dbGaP, EGA, individual-level genomics, PHI).",
    "Policy excludes non-commercial assets (COSMIC, OncoKB, non-commercial model weights) from the core path; any optional use must be segregated and labeled non-commercial.",
    "Per-collection (not source-family) license verification is required, since TCIA terms vary per collection.",
    "Rule stated: no pixel is touched until the collection's allow-list entry is approved by the license reviewer.",
    "Every collection entry reserves provenance fields (retrieval date, checksum, license) for the downstream provenance model."
  ],
  "resources": [
    "planning/projects/pathology-image-benchmarks/PLAN.md",
    "CLAUDE.md",
    "docs/good-deed-definition.md",
    "https://portal.gdc.cancer.gov/",
    "https://www.cancerimagingarchive.net/"
  ],
  "output": "A licensing-gate policy document plus the sources/allowlist.yml schema (committed via PR) defining open-only admission, controlled-access exclusion, non-commercial exclusion, and per-collection license verification.",
  "requestor": "jdev1977",
  "verifiedNeed": false,
  "outputLicense": "CC-BY-4.0"
}
```
