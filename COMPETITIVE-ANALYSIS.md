# Competitive + Improvement Analysis — `pathology-image-benchmarks`

> Scope: open WSI computational-pathology benchmarks on open imagery (open-tier TCGA/CPTAC via GDC/TCIA, CAMELYON, etc.); no identifiable patient data; medium risk. Cancer guardrails: open/de-identified imagery only, per-source license verify, provenance, leakage-aware methodology, WSI de-identification.
> Method: reviewed PLAN.md (v0.1.0) + web research (June 2026). URLs cited inline.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually strong and internally consistent. It already names the field's three hardest problems (site/scanner leakage, license/provenance fog, irreproducibility) and treats the license/de-id gate as a CI-blocking safety subsystem. The review below stress-tests the methodology against the current literature and flags residual gaps.

**Site/scanner/stain batch effects — correct and well-motivated, but under-specified.** The plan correctly cites Howard et al., *Nat Commun* 2021 ([nature.com/articles/s41467-021-24698-1](https://www.nature.com/articles/s41467-021-24698-1)), which showed TCGA carries submitting-site signatures that survive color normalization/augmentation and bias survival, mutation, and stage prediction. The plan's "site-stratified, patient-grouped split + site-prediction probe" is the right design. Three refinements:
- **Adopt Howard's actual remedy explicitly.** Howard proposed a *quadratic-programming* site-preserving cross-validation that keeps each site wholly within one fold. The plan should name this (or an equivalent constrained-assignment algorithm) as the reference method, not just "site-stratified," so the audit is mechanically reproducible.
- **The probe metric needs a defined null.** "Site not trivially recoverable (report AUC)" is good but a raw AUC is not interpretable without a baseline — report site-prediction AUC **relative to** class-prevalence-matched chance and against the label-prediction AUC, because the threat is *correlation between site and label*, not site-predictability per se. If site and the biological label are confounded in the cohort (common in TCGA), even a perfect site-preserving split cannot fully decorrelate them — the plan should state that residual confounding is *surfaced and quantified*, not eliminated.
- **Foundation-model embeddings do NOT remove batch effects** — a critical, recent finding the plan misses. Kömen et al. (NeurIPS 2024, [arxiv 2411.05489](https://arxiv.org/abs/2411.05489)) show UNI/Virchow-class embeddings *still* encode distinct hospital/site signatures that dominate feature-space distance and are not removed by stain normalization. Implication: a foundation-model + linear-probe baseline on a naive split is *especially* vulnerable to site leakage, and any retrieval/distance-based eval on multi-site data is unsafe. The site-confounding audit must run **on the frozen FM embeddings**, not only on raw pixels.

**The famous TCGA site-signature leakage — covered, add the corroborating result.** Beyond Howard, Dehkharghanian et al. "Biased data, biased AI" (*Diagnostic Pathology* 2023, [link](https://diagnosticpathology.biomedcentral.com/articles/10.1186/s13000-023-01355-3)) independently shows deep nets predict TCGA acquisition site. Cite both; they make the audit non-optional and reviewer-defensible.

**Patient-level splits — correct, quantify the stakes.** Tile-from-same-patient contamination inflates reported performance by up to ~41% (widely reported in the pathology-leakage literature). The plan mandates patient-grouped splits; it should add a CI assertion that **no patient_id straddles split boundaries** and that **tiles inherit the slide's split** (a frequent silent bug). Good.

**WSI de-identification (label/macro carry PHI) — strongest part of the plan; tighten tooling.** The plan correctly treats SVS/DICOM **label, macro, and associated images** plus `ImageDescription`/DICOM tags as PHI carriers, with OCR + heuristics + quarantine. This matches the field: NCI's 2023 WSI de-identification workshop ([Springer](https://link.springer.com/article/10.1007/s10278-024-01183-x)) and tools like `pearcetm/svs-deidentifier` ([GitHub](https://github.com/pearcetm/svs-deidentifier)) and John Snow Labs Visual NLP `remove_phi()`. Refinements: (a) **reuse, don't reinvent** — wrap `svs-deidentifier`/MIDI-pipeline rather than building OCR from scratch; (b) note that **GDC/TCIA-delivered slides are largely pre-stripped of label/macro**, so the real residual risk is burned-in pixel text in the *tissue* image and stale metadata, not labels — calibrate the scan accordingly; (c) define the **manual-review SLA** the plan flags as open; (d) the audit itself must not log extracted PHI text (only hashes/flags) — restate the Elyos no-secrets-in-logs rule for the de-id reports.

**Metric correctness — mostly right, sharpen.** Balanced accuracy / AUROC / Cohen's κ are appropriate. Add: (a) for slide-level cancer tasks, **report CIs via patient-level bootstrap** (not tile-level — that under-estimates variance); (b) for imbalanced tasks prefer **AUPRC** alongside AUROC; (c) CAMELYON17 uses a **quadratic-weighted kappa** on pN-stage — match established metrics where a canonical task exists so results are comparable. The plan's "no single-seed headline; report across-seed variance" is excellent and rare in the field.

**Stain normalization — present but treat as an experimental axis, not a fixed step.** The multicenter benchmark of 8 normalization methods ([Sci Rep, arxiv 2506.19106](https://arxiv.org/abs/2506.19106)) and the stain-variation prognosis study ([Sci Rep](https://www.nature.com/articles/s41598-024-83267-w)) show normalization choice materially changes results and a single Macenko reference generalizes poorly. The plan should treat stain normalization as a **declared, hashed pre-processing variable swept in the harness** (Macenko/Vahadane/Reinhard/none), not a silent default — otherwise it becomes its own hidden confound.

**Foundation-model eval — add the leakage-aware angle as the differentiator (see §4).** The plan's "optional, license-segregated FM baseline" under-sells the opportunity. The unique contribution is **measuring how much of an FM's apparent skill is site-signature**, which no mainstream FM benchmark does.

**Compute scale for gigapixel WSIs — realistic but quantify.** A single 40× WSI is ~1–4 GB uncompressed and yields 10k–200k tiles; feature extraction across a TCGA cohort is hundreds of GPU-hours. The plan correctly front-loads compute-light value and isolates heavy baselines, but should add concrete numbers (target cohort slide count × tiles × encoder FLOPs) so the "trivial + 1 MIL baseline" floor is sized to what CPU/CI can actually run. Pre-extracted features (HEST/Patho-Bench distribute these) can sidestep most GPU cost — recommend depending on published features where license permits.

**License per slide source — correct and notably rigorous.** Open-tier TCGA imaging vs controlled genomic tier is correctly separated; per-collection TCIA/CPTAC CC-BY verification is right; COSMIC/OncoKB and several FM weights flagged non-commercial. One addition: **FM weight licenses are heterogeneous and shifting** (UNI/CONCH gated, research-use; Virchow non-commercial; check current terms at use-time, snapshot the license text). Add the **CAMELYON** license to the verify list — it is often cited as CC0 but confirm current Grand-Challenge terms per the plan's own caveat.

**Completeness gaps:** (1) no **negative-control / sanity task** beyond majority baseline — add a *shuffled-label* run that must score at chance (catches harness leakage); (2) no explicit **"label provenance/quality"** check — TCGA molecular/clinical labels have known errors and PANDA had documented noisy labels; (3) no statement on **inter-rater / ground-truth uncertainty** for the chosen task.

---

## 2. Competitive landscape

**Mahmood Lab ecosystem (the dominant incumbent).**
- **Patho-Bench** ([GitHub](https://github.com/mahmoodlab/patho-bench), [arxiv 2502.06750](https://arxiv.org/pdf/2502.06750), [HF](https://huggingface.co/datasets/MahmoodLab/Patho-Bench)) — standardized FM benchmark, canonical train-test splits for ~42 public WSI/patient tasks across 33 datasets, 6 task families; linear probe, Cox survival, retrieval, fine-tune. *Strength:* scale, adoption, pre-computed splits. *Weakness:* splits are train/test only (no published val), and the released splits are **not described as site-stratified/site-audited** — the very leakage Howard flagged is not a first-class control; provenance/de-id/license-gate is not the product.
- **HEST-1k** ([NeurIPS 2024](https://proceedings.neurips.cc/paper_files/paper/2024/file/60a899cc31f763be0bde781a75e04458-Paper-Datasets_and_Benchmarks_Track.pdf), [HF](https://huggingface.co/datasets/MahmoodLab/hest)) — 1,229 spatial-transcriptomics + WSI pairs, 9 gene-expression-from-histology tasks. *Strength:* multimodal, real biomarker tasks. *Weakness:* narrow (ST regression), not a general leakage-aware classification benchmark.
- **CLAM** ([arxiv 2004.09666](https://arxiv.org/abs/2004.09666), [GitHub](https://github.com/mahmoodlab/CLAM)) — the de-facto attention-MIL pipeline/baseline (tissue seg, tiling, ABMIL). *This is your MIL baseline reference implementation,* not a competitor.

**Evaluation frameworks.**
- **EVA (kaiko.ai)** ([GitHub](https://github.com/kaiko-ai/eva), [OpenReview](https://openreview.net/forum?id=FNBQOPj18N)) — open eval framework for pathology FMs: patch/slide/segmentation/VQA. *Strength:* clean, reproducible, open-source harness — closest in spirit to your harness. *Weakness:* a *runner*, not a curated leakage-safe + license-gated dataset layer; no de-id/provenance gate.
- **PathBench (birkhoffkiki)** ([arxiv 2505.20202](https://arxiv.org/abs/2505.20202)) — comparison benchmark explicitly claiming **leakage-free** eval, but via **private** medical-provider data excluded from pretraining. *Strength:* leakage-aware framing. *Weakness:* **not open/reproducible** (private data) — the opposite of your open-commons thesis.
- **PANDA-PLUS-Bench** ([arxiv 2512.14922](https://arxiv.org/pdf/2512.14922)) — robustness eval for prostate FMs.

**Challenges (Grand-Challenge.org).**
- **CAMELYON16/17** ([camelyon17.grand-challenge.org](https://camelyon17.grand-challenge.org/)) — breast lymph-node metastasis; **multi-center (5 hospitals)** with center metadata and a held-out scored test — partially site-aware already; pN-stage quadratic-kappa metric; widely cited; favorable (CC0-ish) terms. *Best candidate for a CC0 site-generalization control task.*
- **PANDA** ([panda.grand-challenge.org](https://panda.grand-challenge.org/)) — prostate ISUP grading, ~11k WSI, 2 centers; largest public set but **documented label noise** — good for a "label-quality matters" narrative.
- **MIDOG** — mitosis detection **domain-generalization across scanners** (directly your batch-effect theme). **TIGER** — TILs in breast. **TCGA/GDC** + **TCIA/CPTAC** are the upstream archives, not benchmarks.
- *Weakness of the challenge model generally:* frozen leaderboards, closed test sets, one-shot events, little ongoing license/provenance/de-id discipline, and rarely a published site-confounding audit.

**Foundation models (the things being benchmarked).** UNI (Mahmood, [GitHub](https://github.com/mahmoodlab/UNI)), Virchow/Virchow2 (Paige, DINOv2, 1.5M slides), CONCH (vision-language). Recent benchmarking ([Nat BME 2025](https://www.nature.com/articles/s41551-025-01516-3)) finds CONCH/Virchow2 lead — but Kömen et al. ([arxiv 2411.05489](https://arxiv.org/abs/2411.05489)) show all retain hospital signatures. These are baselines/subjects, not competitors.

**De-identification prior art.** `pearcetm/svs-deidentifier` ([GitHub](https://github.com/pearcetm/svs-deidentifier)), John Snow Labs Visual NLP, and the **NCI MIDI pipeline/datasets** ([Springer 2024](https://link.springer.com/article/10.1007/s10278-024-01183-x)) — reuse these rather than rebuild.

---

## 3. Gaps we can fill

1. **An open benchmark whose primary artifact is a leakage-safe split + a published site-confounding audit.** Patho-Bench/EVA/HEST ship splits and harnesses; none ship a *Howard-style site-preserving split with an audited site-probe AUC* as the headline deliverable on open data. This is the white space.
2. **A license + de-identification + provenance gate as a reusable, CI-enforced component.** No competitor packages per-collection license verification, controlled-access exclusion, and defensive WSI label/macro PHI re-scan as an auditable, versioned subsystem. This is genuinely missing infrastructure.
3. **FM batch-effect transparency.** Report, for each FM baseline, *how much apparent skill is site signature* (probe on embeddings) — operationalizing Kömen et al. as a standing benchmark metric. Nobody does this routinely.
4. **Reproducibility-by-construction** — digest-pinned container + mandated independent third-party re-run within tolerance + a results schema that makes single-seed/unpinned claims un-submittable. EVA is reproducible; few enforce *third-party* reproduction.
5. **Negative-control / shuffled-label sanity tasks** baked into the harness — a cheap, rare, high-trust feature.
6. **Open, pre-extracted, license-clean feature sets** (CPU-usable) so the benchmark is runnable without GPUs — fills the compute-access gap for students/low-resource labs.

---

## 4. Differentiators to win

1. **"Trust layer," not another leaderboard.** Position as the *audited, license-clean, leakage-safe substrate* others can build on (incl. Patho-Bench/EVA can consume your splits) — cooperative, not competitive.
2. **Site-confounding audit as a first-class, published metric** (raw-pixel *and* FM-embedding probe, with chance-relative reporting). Defensible, novel, directly answers the field's biggest credibility problem.
3. **The license/de-id/provenance gate as a portable artifact** other dataset builders can adopt — turns a compliance chore into a reusable public good.
4. **Honest reporting by schema** (seeds, variance, preprocessing hash, compute provenance, shuffled-label control) — structurally prevents the inflation the literature suffers from.
5. **Compute-light, open features** — runnable in CI/Colab, lowering the barrier vs GPU-heavy competitors.
6. **Provenance + "not for clinical use" + datasheets/model cards on every artifact** — the governance maturity reviewers and journals increasingly require.

---

## 5. Claude API leverage (and where Claude must NOT decide)

**High-leverage uses (Claude writes code/docs that humans then run and verify):**
1. **Build the evaluation/leakage-audit harness** — generate the Python WSI adapter (OpenSlide/pyvips tiling, MPP normalization), the site-preserving split generator (quadratic-programming/constrained assignment per Howard), the site-probe audit (train probe, report chance-relative AUC on pixels and FM embeddings), the shuffled-label negative control, and the bootstrap-CI metric code. Claude is strong at this scaffolding + tests.
2. **Write the license/provenance linter, allow-list schema, and de-id orchestration glue** — TypeScript schema validators, provenance-completeness CI checks, and wrappers around existing de-id tools (`svs-deidentifier`, MIDI). Claude drafts; it must not be the final license arbiter.
3. **Generate documentation at scale** — datasheets (Gebru et al.), model cards (Mitchell et al.), the methods write-up, per-collection rights analyses *as drafts for human review*, and reproducibility READMEs/container specs.
Secondary: draft baseline code (CLAM/ABMIL wrappers), summarize per-collection license terms into structured allow-list entries for human verification, and triage candidate collections.

**Where Claude must NOT decide (hard guardrails):**
- **Benchmark numbers come only from executed, pinned code** — Claude never estimates, infers, or "fills in" a metric. No fabricated results, ever. Every number carries split/seed/hash/compute provenance from a real run.
- **Leakage / site-confound audits and de-identification are methods + human calls.** Claude can implement the audit; a qualified comp-path/bioinformatics reviewer must interpret residual confounding and sign off, and a human must adjudicate any quarantine hit. A green automated de-id scan is *necessary, not sufficient*.
- **License & PHI determinations are human-verified.** Open-access tier, reuse rights, controlled-access exclusion, and "is this PHI" are reviewer decisions recorded in the allow-list — not Claude's verdict.
- **Refusal guardrails apply to Claude itself:** any task steering toward controlled-access/dbGaP data, re-identification, skipping the de-id re-scan, or presenting the benchmark as clinical must be refused and flagged.
- **No clinical/diagnostic claims** in any Claude-generated text; the "not for clinical use" notice is mandatory.

---

## 6. Ten concrete optimizations

1. **Name Howard's site-preserving (quadratic-programming) split as the reference algorithm** and ship it as runnable code, not just "site-stratified."
2. **Run the site-confounding probe on frozen FM embeddings, not only raw pixels** (operationalize Kömen et al.); report chance-relative AUC and label-vs-site correlation.
3. **Add a shuffled-label negative-control run** to CI that must score at chance — catches harness leakage cheaply.
4. **Reuse `svs-deidentifier` / NCI MIDI pipeline** for de-id instead of building OCR from scratch; calibrate the scan to GDC/TCIA's already-stripped delivery (residual risk = pixel-burned text + stale metadata).
5. **Treat stain normalization as a swept, hashed pre-processing axis** (Macenko/Vahadane/Reinhard/none), not a silent default; report sensitivity.
6. **Depend on published pre-extracted features** (HEST/Patho-Bench/CLAM-style) where license permits, to make the benchmark CPU/CI-runnable and sidestep GPU cost.
7. **Match established task metrics** where a canonical task exists (e.g. CAMELYON17 pN-stage quadratic-weighted kappa) so results are directly comparable.
8. **Add patient-level bootstrap CIs** to every metric and assert in CI that no patient_id straddles split boundaries and tiles inherit slide split.
9. **Snapshot FM-weight and per-collection license text at use-time** (they change); store the snapshot + retrieval date in the provenance ledger.
10. **Pick CAMELYON16/17 (CC0, multi-center) as the site-generalization control task** alongside a TCGA cohort — gives a clean, leakage-aware, redistributable cross-site benchmark on day one.

---

## 7. Parallel & perpendicular spin-offs

- **`ml-oncology-benchmarks` (sibling):** factor the leakage-audit harness, results schema, and reproducibility container into a **shared, modality-agnostic core**; pathology is the first modality, genomics/radiology reuse the same honest-reporting + site/batch-audit machinery. Avoid duplicating the gate.
- **`model-cards-oncology` (sibling):** the model-card/datasheet templates + "not for clinical use" lint here become a reusable artifact library that project consumes; the FM batch-effect/site-signature metric becomes a standard model-card field.
- **`ewing-expression-reanalysis` (sibling):** shares the **license/provenance/de-id gate** and the leakage-aware split philosophy (expression analyses have the same site/batch confounds); cross-link the allow-list + provenance schema.
- **Leakage-aware path-AI harness (perpendicular):** spin the site-preserving split generator + embedding site-probe into a **standalone pip package** other labs (and Patho-Bench/EVA) can drop in — high reuse, low maintenance, a credible "adopted by ≥1 external group" path to Definition of Shipped.
- **Public leaderboard (perpendicular, cautious):** a *trust-scored* leaderboard that reports site-probe AUC and reproduction status alongside accuracy — explicitly resisting single-number gaming (consistent with the plan's non-goal).
- **MCP server (perpendicular):** an "open-pathology-benchmark" MCP server exposing tools to query the allow-list, fetch a pinned split, run the site-audit, and validate a results record against the schema — lets agents consume the benchmark without touching raw pixels.

---

## 8. Open questions

1. **First cohort/task:** a TCGA open-access cohort (e.g. a binary molecular/subtype task with clean labels) vs CC-BY CPTAC vs CC0 CAMELYON as the site-generalization control — optimize for verified license, label quality, manageable size, and domain-reviewer availability. (Plan: decided by M1.)
2. **How is residual site↔label confounding handled** when the cohort itself confounds them (TCGA often does)? Report-and-bound, exclude the task, or down-weight? Needs a documented decision rule.
3. **Build vs reuse the de-id scanner** — wrap `svs-deidentifier`/MIDI vs custom OCR; and the **manual-review SLA** on a quarantine hit.
4. **Pre-extracted features vs raw-pixel re-tiling** — can we depend on others' published features (license-clean, CPU-cheap) without inheriting *their* preprocessing/leakage choices?
5. **Compute path** for any MIL/FM baseline — donated vs funded GPU, and the hard per-task budget cap if funded.
6. **Will Patho-Bench/EVA/a toolkit (OpenSlide/TIAToolbox/CLAM) adopt the splits/gate** — the most likely route to external adoption; start that conversation early (gates M4).
7. **CC-BY-4.0 vs CC0** per derived artifact, and **per-collection derived-artifact redistribution rights** (manifests-only vs tiles/features).
8. **Whether to include any non-commercial FM baseline** at all (aids comparison, risks reuse confusion).

---

### Sources
Howard et al. 2021 — https://www.nature.com/articles/s41467-021-24698-1 · Dehkharghanian 2023 — https://diagnosticpathology.biomedcentral.com/articles/10.1186/s13000-023-01355-3 · Kömen et al. 2024 (batch effects in FMs) — https://arxiv.org/abs/2411.05489 · Patho-Bench — https://github.com/mahmoodlab/patho-bench , https://arxiv.org/pdf/2502.06750 · HEST-1k — https://proceedings.neurips.cc/paper_files/paper/2024/file/60a899cc31f763be0bde781a75e04458-Paper-Datasets_and_Benchmarks_Track.pdf · CLAM — https://arxiv.org/abs/2004.09666 · EVA (kaiko.ai) — https://github.com/kaiko-ai/eva · PathBench — https://arxiv.org/abs/2505.20202 · PANDA-PLUS-Bench — https://arxiv.org/pdf/2512.14922 · CAMELYON17 — https://camelyon17.grand-challenge.org/ · PANDA — https://panda.grand-challenge.org/ · FM benchmarking (Nat BME 2025) — https://www.nature.com/articles/s41551-025-01516-3 · Stain-norm multicenter — https://arxiv.org/abs/2506.19106 · Stain variation/prognosis — https://www.nature.com/articles/s41598-024-83267-w · svs-deidentifier — https://github.com/pearcetm/svs-deidentifier · NCI WSI de-id workshop — https://link.springer.com/article/10.1007/s10278-024-01183-x · UNI — https://github.com/mahmoodlab/UNI
