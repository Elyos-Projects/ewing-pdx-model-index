# Competitive + Improvement Analysis — `ewing-pdx-model-index`

> Scope: open metadata index of Ewing sarcoma in-vivo / preclinical models (PDX, cell line, organoid)
> with harmonized metadata, per-assertion provenance, license clearance, and re-identification review.
> Index/metadata only — never patient-level or controlled-access data. Web-researched and cited.
> Prepared 2026-06-29 against PLAN.md v0.1.0 + TASKS.md.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually disciplined: it leads §7 with binding cancer guardrails, makes provenance
invalid-by-construction, code-enforces an `ingest:true|false` switch, names re-identification as a
release gate, and sets `verifiedNeed=false` honestly. That foundation is genuinely strong. The gaps
below are about *domain fidelity* and *competitive realism*, not structure.

**1.1 — PDX-MINIMUM-information (PDX-MI) alignment is asserted but not operationalized.**
The plan never references PDX-MI, the actual community standard (Meehan et al., *Cancer Res* 2017),
even though the Model Datasheet is functionally a competing schema. PDX-MI defines four modules —
clinical, model creation, model QA, and model study/associated metadata — covering host strain,
implantation site, engraftment/passage, tumor-vs-PDX concordance QA, etc.
([PDX-MI paper](https://pmc.ncbi.nlm.nih.gov/articles/PMC5738926/)). **Gap:** the Datasheet field
list (§6) omits nearly all *model-creation* and *QA* fields that PDX-MI considers minimal — host
mouse strain, implantation site (subcutaneous vs orthotopic), passage number, engraftment rate,
and STR/fusion-confirmation method. For a PDX index these are not optional; they are the fields
researchers select models on. **Fix:** declare the Datasheet a PDX-MI *profile/extension* (map each
field to a PDX-MI element; mark which PDX-MI fields you intentionally drop and why), so you inherit
credibility and interoperability instead of inventing a parallel vocabulary. This also de-risks the
schema task (`ewing-pdx-schema-001`), which currently lists no host-strain/passage/implant fields.

**1.2 — Model provenance & passage drift is under-modeled.** A "model" is not a fixed object: PDX
late passages diverge from the patient tumor (clonal selection, mouse-stromal replacement), and
cell lines drift. The Datasheet has no field for *passage at which a given molecular dataset was
generated*, no derivation lineage (patient → P0 → Px → cell line spun out of a PDX), and no
host-strain. Without these, two records that look like "the same model" may be biologically
different. **Fix:** add `derivationLineage`, `passageOfRecord`, and per-`molecularData` entry a
`passageAtAssay`. JAX/MTB and PDCM Finder both track this; omitting it makes the index look naive to
the target users. ([JAX MTB PDX](https://tumor.informatics.jax.org/mtbwi/pdxSearch.do),
[PDCM Finder](https://pmc.ncbi.nlm.nih.gov/articles/PMC9825610/)).

**1.3 — Fusion-status reliability is treated as a boolean, not a graded evidence claim.** `fusion`
+ `fusionConfirmed` (bool) is too coarse. Ewing fusion evidence ranges from *named in a paper* to
*FISH-confirmed* to *RNA-seq breakpoint-resolved* to *EWSR1-FLI1 type 1 vs type 2 exon structure*.
Cell-line literature also carries **misidentification / cross-contamination** history (the whole
reason Cellosaurus tracks problematic lines and STR profiles). **Gap:** no `fusionEvidence`
(FISH / RT-PCR / RNA-seq / reported-only), no `fusionBreakpoint`, no link to STR-authentication or
Cellosaurus "problematic" flags. **Fix:** model fusion as `{partner, type, evidenceMethod, citation,
confidence}` and surface Cellosaurus misidentification/contamination flags as a first-class warning.
This is a differentiator *and* a correctness requirement — publishing an unconfirmed fusion as
"confirmed" would be exactly the kind of unsourced biological claim §7 forbids.

**1.4 — Donor de-identification & consent-at-source is asserted but not verifiable per model.**
The plan correctly keeps patient-level data out of scope and coarsens identifying fields. But PDX
models *derive from a real patient*, and the live ethical question is whether the **source
repository's** model was de-identified and consented at source. The plan's re-id review checks *our*
fields; it does not record *whether the upstream model has documented consent/ethics provenance*.
**Gap:** no `donorConsentProvenance` / `sourceEthicsStatement` field capturing "source states model
derived under IRB/consent X" with a citation, and no policy for what to do when a source is silent.
**Fix:** add a (sourced, non-identifying) `sourceConsentAttestation` field — link to the provider's
ethics statement — and make "source provides no de-identification/consent statement" a reviewer flag,
not an automatic include. This keeps the human-verified consent determination where §5 of the
guardrails wants it.

**1.5 — Availability / access terms are modeled, but the hardest case is missing.** `accessTerms`
(open|mta|commercial|unknown) is good, but real availability is multi-dimensional: *who can request*
(academic-only vs anyone), *MTA flavor* (UBMTA vs custom vs commercial-prohibited), *fee/shipping*,
*still distributing?* (CSTN "upon request", PDMR custom MTA, DFCI LLX core for ProXe subset,
ATCC/vendor purchase). **Gap:** `accessTerms` collapses these. Two "mta" models can be a 1-week
academic UBMTA vs a 6-month custom negotiation. **Fix:** split into `distributor`,
`requestPathwayUrl`, `mtaType`, `eligibility`, `lastConfirmedAvailable`. Availability-awareness is
your headline differentiator (§4 below) — it deserves more than one enum.
([PDMR MTA](https://pdmr.cancer.gov/request/default.htm),
[CSTN resources](https://www.stjude.org/research/why-st-jude/data-tools/childhood-solid-tumor-network/available-resources.html),
[ProXe/LLX](https://www.proxe.org/)).

**1.6 — License of the AGGREGATED metadata: the plan's stated PDCM Finder license is wrong/unverified.**
§7's source table marks PDCM Finder as "Open (verify CC0/CC-BY + terms)". **Verified finding:** PDCM
Finder model metadata is distributed "under the general **EMBL-EBI terms of use**" — *not* a named
CC0/CC-BY license — and only the *software* is Apache-2.0 and the *paper* is CC-BY
([PDCM Finder, NAR 2023](https://academic.oup.com/nar/article/51/D1/D1360/6833240),
[PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC9825610/);
[terms page defers to EMBL-EBI + JAX terms](https://www.cancermodels.org/terms-of-use)). EMBL-EBI
terms-of-use permit reuse but are not a standard CC license, which complicates re-publishing the
aggregate as a clean CC-BY-4.0 dataset (plan §7's stated output license). **This directly answers
Open Question #3 and shows the plan's tentative assumption is optimistic.** **Fix:** do not assume
CC0/CC-BY for PDCM Finder; record "EMBL-EBI terms of use" verbatim, and treat the *aggregation* as a
collection of facts each carrying its own upstream terms rather than a single re-licensable corpus.
The plan's "segregate share-alike subsets" idea is good but insufficient if a major source isn't CC
at all. Add a fourth license posture beyond ingest/reference/out-of-scope: **"fact-cite-only"**
(record the datum + cite, but do not redistribute the source's compiled metadata table).

**1.7 — Dedup across repositories is the central technical risk and is under-specified.** The same
physical model appears as a Cellosaurus `CVCL_*`, a DepMap `ACH-*`, a PDCM Finder model id, possibly
a CCMA entry, possibly a PPTC/PIVOT model, and a lab name in a paper — and Ewing is small enough that
classic lines (A673, TC71, SK-N-MC, EW8, CHLA-9/10/258, RD-ES, SK-ES-1) recur everywhere. The
resolver (§6 stage 4) is described as "explicit, logged, reversible" but with **no matching
algorithm, no authority precedence, and no handling of the PDX-vs-derived-cell-line-of-the-same-patient
case** (sibling models, not duplicates — must NOT be merged). **Fix:** specify (a) Cellosaurus RRID
as the anchoring authority for cell lines (it already cross-references DepMap/Cosmic/ATCC), (b) a
"same patient ≠ same model" rule preserving sibling lineage, (c) a confidence-scored match with human
confirmation for anything below threshold. ([Cellosaurus is the natural identity backbone](https://www.cellosaurus.org/)).

**1.8 — Metric realism.** O1 target "≥60 distinct models" — plausibility check: CCMA alone lists
**12 Ewing cell lines**, the Ewing Sarcoma Cell Line Atlas has **18**, plus PDX/O-PDX from CSTN,
PPTC/PIVOT (phs001437), PDMR, and EuroPDX. ≥60 distinct *de-duplicated* models is ambitious but
plausible; however O3's "≥70% of cell-line models linked to Cellosaurus + DepMap" will be easy for
cell lines and near-impossible for **PDX/organoids** (DepMap is cell-line-centric). **Fix:** split O3
by model type (cell line: ≥90% Cellosaurus; PDX: link to PDCM Finder/PDMR/EuroPDX, not DepMap).
([CCMA](https://www.cell.com/cancer-cell/fulltext/S1535-6108(23)00080-6),
[ESCLA](https://www.cell.com/cell-reports/fulltext/S2211-1247(22)01644-8)).

**1.9 — Other concrete gaps.** (a) No GEMM/zebrafish note — Ewing has *no* spontaneous animal model
and no bona fide GEMM, so "in vivo model index" is really PDX/O-PDX/xenograft-of-cell-line; state
this explicitly. (b) No handling of **molecular-subtype confounders**: *EWSR1::non-ETS* and
*Ewing-like / CIC-DUX4 / BCOR-CCNB3* round-cell sarcomas are frequently mis-shelved as "Ewing" —
the index must distinguish *Ewing sarcoma (EWSR1-ETS)* from *Ewing-like*. (c) O7 "<5% dead links"
has no decay model for the many supplementary-table/orphan accessions §2 complains about. (d) No
versioned **stable identifier scheme** for your own `modelId` (needed so external citations in O6
don't break). (e) "verifiedNeed=false" is honest but M0–M2 build a 60-model catalog *before*
securing a partner — consider gating M2 breadth on at least one validation signal.

---

## 2. Competitive landscape

No existing resource is a *Ewing-specific, open, harmonized, availability-aware* model index. But
several strong **horizontal** aggregators and **pediatric** repositories overlap heavily. Honest
read: PDCM Finder is the 800-lb gorilla and the project must differentiate against it explicitly.

**PDCM Finder / PDX Finder (EMBL-EBI + The Jackson Laboratory)** — the dominant open aggregator.
6,316 model entries (4,661 xenograft, 1,547 cell line, 108 organoid) from 27 providers, all
standardized to **PDX-MI**, with API + bulk download; ~17% rare/pediatric models.
*Strengths:* the de-facto standard, real harmonization, API, ongoing funding, links molecular +
drug-dosing data. *Weaknesses/gaps:* breadth-over-depth — Ewing is a thin slice with no
disease-specific curation; **availability/MTA terms are not systematically actionable**; metadata is
under EMBL-EBI terms-of-use (not a clean CC license), complicating downstream re-licensing; provider
coverage is whoever submits, so absence ≠ nonexistence; no per-assertion provenance to the originating
paper. ([NAR 2023](https://academic.oup.com/nar/article/51/D1/D1360/6833240),
[PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC9825610/), [cancermodels.org](https://www.cancermodels.org/)).

**PDXNet portal (NCI)** — consortium portal; 334 new models across 33 cancer types, with workflows,
tools, and tight coupling to PDMR. *Strengths:* standardized generation, RNA-seq/WES, diversity
focus, reproducible pipelines. *Weaknesses:* consortium-scoped (PDXNet-generated models only), not a
cross-repository index; Ewing not a focus; portal is discovery-of-PDXNet, not the field.
([PDXNet portal, NAR Cancer 2022](https://academic.oup.com/narcancer/article/4/2/zcac014/6572305),
[pdxnetwork.org](https://www.pdxnetwork.org/)).

**EuroPDX Data Portal** — distributed European PDX collection with clinical/molecular/pharmacological
annotation, searchable by metadata; models available "on a cooperative basis." *Strengths:* deep
treatment-study + molecular profiles, real European model access pathway, named as a candidate
partner in the plan. *Weaknesses:* cooperative/consortium access (not fully open distribution); not
pediatric-specialized; a portal, not a harmonized cross-source index.
([BMC Genomics 2022](https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-022-08367-1),
[dataportal.europdx.eu](https://dataportal.europdx.eu/)).

**NCI PDMR** — US national patient-derived models repository (PDX, organoid, PDC, CAF) that actually
*distributes* material under a (non-negotiable) NCI MTA. *Strengths:* authoritative, distributes
real material, public model metadata, sarcoma included. *Weaknesses:* a single-provider repository,
not an index; MTA is custom/US-gov flavored; Ewing-specific browse is limited.
([pdmr.cancer.gov](https://pdmr.cancer.gov/), [MTA SOP](https://pdmr.cancer.gov/request/default.htm)).

**Childhood Cancer Model Atlas (CCMA) (Hudson Institute / ZERO)** — largest open pediatric
**cell-line** atlas: >400 high-risk pediatric lines incl. **~12 Ewing sarcoma** (within 33 bone/soft-
tissue sarcoma lines), multi-omics + functional screens, shiny portal, CCDI-listed. *Strengths:*
pediatric-native, deeply profiled, open, sarcoma representation. *Weaknesses:* **cell lines (not
PDX/in-vivo)**; a profiling atlas, not an availability/provenance index; no cross-repo identifier
resolution. ([Cancer Cell 2023](https://www.cell.com/cancer-cell/fulltext/S1535-6108(23)00080-6),
[CCMA portal](https://ccma.shinyapps.io/ccma/),
[CCDI listing](https://datacatalog.ccdi.cancer.gov/dataset/VPCC-CCMA)).

**PPTP → PPTC → PIVOT (NCI pediatric preclinical in vivo testing)** — the canonical pediatric PDX
*testing* programs, with established sarcoma (incl. Ewing) panels; PPTC PDX model data deposited as
**phs001437** and an active 2025 Ewing preclinical-biology effort. *Strengths:* gold-standard
pediatric in-vivo panels, drug-response data, Ewing-specific activity. *Weaknesses:* program/portal
fragmented across eras; model metadata is spread across publications + a dbGaP study (controlled
tiers); no unified open availability-aware index.
([PIVOT](https://preclinicalpivot.org/about-pivot/),
[Ewing preclinical biology 2025](https://aacrjournals.org/mct/article-pdf/doi/10.1158/1535-7163.MCT-25-0428/3656065/mct-25-0428.pdf),
[GCCRI PPTC](https://gccri.uthscsa.edu/research/project-b-the-pediatric-preclinical-consortium-pptc/)).

**Childhood Solid Tumor Network (CSTN), St. Jude** — 166 orthotopic PDX (O-PDX) across 21 diagnoses
incl. Ewing, with genomic/epigenomic/drug-screen data and material "available upon request,"
cloud-based portal. *Strengths:* deep O-PDX characterization, real (free) distribution pathway,
pediatric-native, Ewing included. *Weaknesses:* single-institution; portal-not-index; "upon request"
availability undocumented in machine-readable form.
([CSTN](https://www.stjude.org/research/why-st-jude/data-tools/childhood-solid-tumor-network.html),
[CSTN portal](https://cstn.stjude.cloud/)).

**ProXe (Public Repository of Xenografts), DFCI** — heme-malignancy PDX repository (AML/ALL/NHL),
heavily patient-annotated, subset distributed via DFCI LLX core. *Strengths:* model of how to pair an
index with a distribution core; rich annotation. *Weaknesses:* **leukemia/lymphoma only — adjacent,
not competing** for Ewing; notably carries patient-level demographics (a contrast: the Ewing index
must *not*). ([proxe.org](https://www.proxe.org/),
[PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC5177991/)).

**Cell Model Passports (Sanger)** — hub for >2,000 cell-line + organoid models with clinical/genomic/
functional data, REST API, HCMI integration, >30 attributes incl. patient gender/ethnicity/age.
*Strengths:* excellent identifier hygiene (ICLAC, relationships), API, organoids. *Weaknesses:*
**no PDX/in-vivo**; pan-cancer (Ewing not curated); records patient age/ethnicity (re-id-relevant
contrast). ([NAR 2019](https://academic.oup.com/nar/article/47/D1/D923/5107576),
[cellmodelpassports.sanger.ac.uk](https://cellmodelpassports.sanger.ac.uk/)).

**Cellosaurus (SIB)** — CC-BY-4.0 authoritative cell-line knowledge base (RRIDs, cross-refs to
DepMap/COSMIC/ATCC, misidentification + STR flags). *Strengths:* the identity backbone; clean
license; tracks problematic/contaminated lines. *Weaknesses:* cell lines only (no PDX as such), not
availability- or disease-curated. **Best used as a dependency, not a competitor.**
([cellosaurus.org](https://www.cellosaurus.org/)).

**JAX Mouse Tumor Biology (MTB) / JAX PDX** — 455 clinically + genomically annotated JAX PDX models
(NSG host), searchable by site/diagnosis/genomics/dosing. *Strengths:* deep single-provider PDX
characterization + free informatics. *Weaknesses:* JAX-generated models only; commercial-services
adjacency; not a cross-repo index.
([MTB PDX](https://tumor.informatics.jax.org/mtbwi/pdxSearch.do),
[MTB DB, PMC](https://pmc.ncbi.nlm.nih.gov/articles/PMC4384039/)).

**DepMap (Broad)** — cell-line dependency/omics (CC-BY-4.0 metadata). *Adjacent dependency*, not a
model index; cell-line-only; no PDX, no availability/MTA layer.

**Net competitive picture:** lots of *deep verticals* (CCMA, CSTN, PPTC, PDMR, JAX) and one *broad
aggregator* (PDCM Finder), but **nobody offers a Ewing-focused, cross-repository, provenance-stamped,
availability-aware index that tells a researcher "here is every described Ewing model, can you get
it, and on what terms."**

---

## 3. Gaps we can fill

1. **Disease-complete, cross-repository view.** PDCM Finder, CCMA, CSTN, PDMR, PPTC, EuroPDX each hold
   *some* Ewing models; none unions them. We are the only resource that says "across all open
   sources, here is the de-duplicated Ewing model set."
2. **Availability-as-data.** Convert "available upon request" / MTA prose into structured
   `distributor + requestPathway + mtaType + eligibility + lastConfirmedAvailable`. No competitor
   makes obtainability machine-queryable — this is the researcher's actual blocker.
3. **Per-assertion provenance to the originating paper.** Aggregators record a value; we record
   *value + who said it + where + license + retrieval date*, making every claim auditable — uniquely
   suited to AI-assisted curation with human sign-off.
4. **Ewing taxonomy correctness.** Explicitly separate *Ewing sarcoma (EWSR1-ETS)* from *Ewing-like /
   CIC-DUX4 / BCOR-CCNB3*, and grade fusion evidence — a curation depth horizontal tools won't invest
   in for one rare disease.
5. **Misidentification / authentication surfacing.** Pull Cellosaurus problematic-line + STR flags to
   the front so nobody bases work on a contaminated "Ewing" line.
6. **Orphan-accession reconciliation.** The dead-link / orphaned-supplementary-table pain §2 names is
   real and unowned; a quarterly link-resolver is a concrete, deliverable good.
7. **Re-identification-aware coarsening as a published standard** for a rare pediatric cancer — a
   reusable pattern other rare-cancer indexes lack (ProXe/Cell Model Passports actually publish
   patient age/ethnicity; we deliberately don't).
8. **Honest "what's missing" coverage ledger** (TASKS `gaps-103`) — a funding/gap-analysis artifact
   no aggregator produces.

---

## 4. Differentiators to win

- **The single strongest differentiator: an availability-aware provenance map.** "Every Ewing model,
  de-duplicated across every open source, with a machine-readable answer to *can I obtain it and on
  what terms*, every claim cited to its source." No competitor combines (cross-repo union) ×
  (structured obtainability) × (per-assertion provenance) for Ewing.
- **Provenance-first, AI-curated-but-human-verified** — facts-with-citations, never model claims.
- **Re-identification-gated** publishing for a rare pediatric cancer (a trust signal families and
  ethics reviewers can verify).
- **PDX-MI-compatible profile** — interoperable with the field's standard rather than a rival schema.
- **Files-in-Git, CC-BY data / MIT code, quarterly-refresh** — auditable, forkable, low-cost, the
  opposite of a portal that rots.
- **Vendor-neutral & conflict-of-interest-governed** — reference-only for vendor catalogs.

---

## 5. Claude API leverage

**Where Claude clearly helps (each output provenance-stamped + human-verified before publish):**
1. **Harmonize heterogeneous metadata to the PDX-MI profile** — map Cellosaurus/DepMap/PDCM Finder/
   PDMR/CSTN fields and free-text methods sections into the Datasheet schema, normalizing host-strain,
   passage, implant-site, and fusion nomenclature. (Large structured-extraction win; tool-use +
   JSON-schema-constrained output.)
2. **Cross-link / resolve identities** — propose candidate `CVCL_*`↔`ACH-*`↔PDCM-id↔paper-name
   matches with a confidence + rationale, *flagging* the sibling-model (same patient ≠ same model)
   trap for a human, never auto-merging.
3. **Extract model facts from PMC-OA literature with citations** — pull fusion type, passage,
   derivation, drug-response availability from open-access papers, emitting `{value, quote, PMCID,
   license}` so every assertion is back-traceable.
4. **Flag availability & access terms** — read provider pages/MTAs and draft structured `distributor/
   mtaType/eligibility` candidates for human confirmation.
5. **Re-identification *lint* (assistive)** — surface small-cell-count attributes, free-text leakage,
   and identifying combinations for the human reviewer to adjudicate.
6. **Dead-link triage & orphan reconciliation** — propose the new home of a moved accession.
7. **Coverage-gap synthesis** — draft the "what's missing" ledger from the catalog state.

**Where Claude must NOT decide (hard human-verified boundaries):**
- **De-identification & consent-at-source determinations** — whether an upstream model is consented/
  de-identified is a human ethics call; Claude may *summarize* a source's stated ethics, never
  *infer* consent or clear a model for inclusion.
- **License determinations** — Claude may extract a stated license string; a human verifies and sets
  the binding `ingest`/`reference`/`fact-cite-only` flag. (The PDCM Finder "EMBL-EBI terms" case in
  §1.6 shows why a naive CC0/CC-BY guess is dangerous.)
- **No fabricated provenance** — never synthesize a citation, PMID, breakpoint, or accession; an
  assertion without a real, retrievable source is dropped, not guessed.
- **Never ingest controlled/identifiable data** — Claude must refuse to process dbGaP/EGA individual-
  level or patient-identifiable inputs; the pipeline must not feed such data to the model at all.
- **Final fusion/biology calls and the re-id release sign-off remain human** — Claude lints; the
  domain + re-id reviewers decide. No release without a signed human checklist.

---

## 6. Ten concrete optimizations

1. **Adopt PDX-MI as the schema backbone** — declare the Datasheet a PDX-MI profile and add the
   missing model-creation/QA fields (host strain, implant site, passage, engraftment, QA method).
2. **Model fusion as graded evidence** — `{partner, type, breakpoint, evidenceMethod, citation,
   confidence}`; never publish "confirmed" without method-level provenance.
3. **Split `accessTerms` into an obtainability sub-object** — `distributor, requestPathwayUrl,
   mtaType, eligibility, fee, lastConfirmedAvailable` — and make it a queryable facet.
4. **Anchor identity on Cellosaurus RRID** for cell lines; define PDCM Finder/PDMR/EuroPDX ids as the
   PDX anchors; encode authority precedence in the resolver.
5. **Add the "same patient ≠ same model" sibling rule** to the resolver, with confidence-scored
   matches and mandatory human confirmation below threshold.
6. **Add a fourth license posture: `fact-cite-only`** for sources (like PDCM Finder under EMBL-EBI
   terms) whose compiled tables aren't cleanly re-licensable — record the datum + cite, don't
   redistribute the compilation.
7. **Add `sourceConsentAttestation`** (sourced, non-identifying) and make "no upstream
   de-id/consent statement" a reviewer flag rather than an auto-include.
8. **Distinguish Ewing (EWSR1-ETS) from Ewing-like** (CIC-DUX4 / BCOR-CCNB3 / EWSR1-non-ETS) as a
   required, validated field — prevents the most common mis-shelving error.
9. **Surface Cellosaurus misidentification/STR-authentication flags** as a front-page model warning.
10. **Ship an API + bulk download from day one** (static JSON/CSV is already API-able; add an OpenAPI
    description) and split O3 coverage targets by model type so PDX isn't measured against DepMap.

---

## 7. Parallel & perpendicular spin-offs

- **`ewing-cell-line-catalog` (sibling, tight)** — the cell-line half could be its own deeply-curated
  Cellosaurus/DepMap/CCMA/ESCLA-anchored catalog; the PDX index links to it rather than duplicating.
- **`ewing-expression-reanalysis` (perpendicular)** — once models are identity-resolved, a uniform
  re-analysis of their open GEO/SRA expression data; the index becomes the *sample manifest* for it.
- **`ewing-open-data-catalog` (umbrella)** — models are one facet; datasets, trials, registries are
  others. The model index is the in-vivo/preclinical pillar of a broader Ewing open-data hub.
- **Generalized rare-cancer in-vivo model index** — the schema + provenance + re-id pattern is
  disease-agnostic; spin a template that other rare/pediatric cancers (DSRCT, rhabdoid, CIC-sarcoma)
  fork. The re-id-aware coarsening standard is the reusable crown jewel.
- **MCP server** — expose the catalog as a Model Context Protocol server so any agent can query
  "Ewing models with confirmed EWSR1-FLI1, openly distributable, with RNA-seq" — turning the dataset
  into agent-callable infrastructure (and a clean Claude-tool-use showcase).
- **Pediatric-preclinical-testing tie-in** — align fields with PPTC/PIVOT and CSTN so the index can
  feed (or annotate) preclinical-testing model selection; a natural partner/steward bridge.
- **Availability-monitoring service** — the `lastConfirmedAvailable` + dead-link engine could run as a
  standing "is this model still obtainable?" watcher across repositories.

---

## 8. Open questions for the maintainer

1. **PDX-MI posture:** will you formally profile PDX-MI (recommended) or maintain an independent
   schema and accept reduced interoperability?
2. **PDCM Finder licensing (now partly answered):** given metadata is under *EMBL-EBI terms of use*
   (not CC0/CC-BY), is the aggregate redistributable as CC-BY-4.0, or do we need the `fact-cite-only`
   posture for it and similar sources? (This reshapes §7 and the output-license claim.)
3. **Consent-at-source policy:** include a model when the source provides *no* documented de-id/consent
   statement, or exclude until attested? Who is qualified to make that call?
4. **Re-id threshold:** what concrete cell-count / attribute-combination rule defines "too
   identifying" for Ewing, and does even coarse primary/relapse/metastatic survive it?
5. **Scope of "model":** include xenografts-of-cell-lines and organoids, or PDX/O-PDX only? Include
   Ewing-like (CIC/BCOR) sarcomas as labeled non-Ewing context, or exclude?
6. **Partner sequencing:** should M2 breadth ingestion be gated on at least one validation signal
   (named partner or ≥3 researcher requests) rather than built speculatively?
7. **Distribution depth:** do we attempt to verify *current* obtainability per model (high-value,
   high-maintenance) or only record the source's stated terms?
8. **Relationship to PDCM Finder:** complement (link out, add Ewing depth + provenance + availability)
   or contribute curation upstream to them? Could co-curation be the partner path?

---

### Key citations
- PDX-MI standard: https://pmc.ncbi.nlm.nih.gov/articles/PMC5738926/
- PDCM Finder (NAR 2023): https://academic.oup.com/nar/article/51/D1/D1360/6833240 · terms: https://www.cancermodels.org/terms-of-use
- PDXNet portal: https://academic.oup.com/narcancer/article/4/2/zcac014/6572305
- EuroPDX Data Portal: https://bmcgenomics.biomedcentral.com/articles/10.1186/s12864-022-08367-1 · https://dataportal.europdx.eu/
- NCI PDMR: https://pdmr.cancer.gov/
- CCMA: https://www.cell.com/cancer-cell/fulltext/S1535-6108(23)00080-6 · https://ccma.shinyapps.io/ccma/
- PPTC/PIVOT: https://preclinicalpivot.org/about-pivot/ · Ewing 2025: https://aacrjournals.org/mct/article-pdf/doi/10.1158/1535-7163.MCT-25-0428/3656065/mct-25-0428.pdf
- CSTN (St. Jude): https://www.stjude.org/research/why-st-jude/data-tools/childhood-solid-tumor-network.html
- ProXe: https://www.proxe.org/ · PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC5177991/
- Cell Model Passports: https://academic.oup.com/nar/article/47/D1/D923/5107576
- Cellosaurus: https://www.cellosaurus.org/
- JAX MTB / PDX: https://tumor.informatics.jax.org/mtbwi/pdxSearch.do
- Ewing Sarcoma Cell Line Atlas (ESCLA): https://www.cell.com/cell-reports/fulltext/S2211-1247(22)01644-8
