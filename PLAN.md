# ewing-pdx-model-index — Open Metadata Catalog of Ewing Sarcoma Preclinical Models

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) ·
> Lane: donated · Risk tier: **medium** (data project; re-identification-aware) · Track: 8a (Ewing Sarcoma)

**Positioning.** A single, trustworthy, openly-licensed *map* of the laboratory models researchers
use to study Ewing sarcoma — patient-derived xenografts (PDX), cell lines, and organoids — assembled
**only** from sources that already describe those models in the open. Not a data warehouse, not a
biobank, not a clinical resource: a **provenance-first metadata catalog** that answers "which Ewing
models exist, what is known about them, where can I get them, and what is the evidence for each
claim?" — so a scientist (or a well-resourced family's advocate) stops losing weeks to fragmented,
paywalled, contradictory model descriptions.

**Why this matters, plainly.** Ewing sarcoma is rare, disproportionately strikes children and young
adults, and has seen little survival improvement in relapsed/metastatic disease for two decades.
Progress depends on good preclinical models, and good models are scattered across repositories,
supplementary tables, and dead links. Every hour a researcher wastes reconciling model records is an
hour not spent on the disease. We take that seriously, and we take the families behind every one of
these models even more seriously — which is exactly why the hardest rules in this plan are about what
we will **never** catalog (see §7).

> In one line: **the open, cited index of Ewing sarcoma models — every model traceable to its source,
> every claim to its evidence, and not one byte of patient-identifiable data.**

---

## 1. Executive summary

Ewing sarcoma research is bottlenecked partly by **model findability**. Useful models — PDX,
cell lines, organoids — are described across PDCM Finder, Cellosaurus, DepMap, the NCI PDMR, the
Childhood Cancer Data Initiative (CCDI), the Pediatric Preclinical Testing Consortium (PPTC), the
Childhood Solid Tumor Network (CSTN), and hundreds of open-access papers — with inconsistent
identifiers, no shared schema, varying access terms, and no single place to see "what is known and
how do we know it."

This project builds an **open, schema-driven, provenance-tracked metadata catalog** of Ewing
sarcoma preclinical models, assembled exclusively from **open-access / aggregate / already-published**
sources, with **per-assertion provenance** and **per-source license verification**. The deliverable
is (a) a normalized, versioned **dataset** (JSON + CSV + a small queryable site), (b) a **Model
Datasheet** standard for describing a Ewing model honestly (including what is *not* known), and
(c) a reproducible, auditable **ingestion + validation pipeline**.

It is a **donated-lane, medium-risk data project.** Medium — not low — because rare-disease model
metadata carries a genuine **re-identification surface** (a relapsed pediatric model from a small
institution can be a near-unique fingerprint), and not high because it produces **no medical advice
and no patient-facing clinical guidance.** The binding gate (see §7) is: open-source-license-clean,
re-identification-reviewed, provenance-complete. **Definition of Shipped:** a researcher can find,
compare, and correctly cite Ewing models from one open catalog, with every record traceable to an
open source and license, and a domain reviewer has signed the re-identification review.

No partner organization or verified institutional need is secured yet — **`verifiedNeed = false`
across the backlog** until a sarcoma-research partner (e.g. a lab, consortium, or foundation) or a
documented researcher request confirms demand and informs scope.

---

## 2. Problem & beneficiaries

**The problem.** A scientist asking "which Ewing models should I use, and what's known about each?"
today must:
- query 6+ repositories with **non-interoperable identifiers** (Cellosaurus `CVCL_*`, DepMap
  `ACH-*`, PDMR/PDCM model IDs, lab-internal names), often for the *same* model under different names;
- read **supplementary tables** and methods sections to learn fusion type, passage, host strain,
  and what omics exist;
- discover that **access terms differ wildly** (open distribution vs. MTA vs. commercial-only) and
  are often undocumented; and
- repeatedly hit **dead links and orphaned accessions.**

There is no single open, cited, license-aware index. This is a tractable, high-leverage gap for
careful AI-assisted curation.

**Who is helped (in order of directness):**
1. **Ewing sarcoma researchers** (academic labs, trainees, consortia) — faster, more reproducible
   model selection; fewer wasted weeks; correct citation and attribution.
2. **Computational/method researchers** — a clean, license-clear cross-reference layer to join
   public model datasets (DepMap, PPTC, GEO) without re-deriving the mapping each time.
3. **Patient advocates and rare-disease foundations** — a legible, sourced view of "what the research
   community has to work with," useful for funding and gap analysis. (Not patient medical guidance.)
4. **Indirectly, families and patients** — every reduction in research friction is a small push
   toward better therapies. We name this honestly: the benefit to families is **real but indirect**;
   this catalog is a research-infrastructure good deed, not a patient resource.

**Verified need: TO BE SECURED (`verifiedNeed = false`).** The need is well-evidenced in the
literature and from the existence of overlapping-but-incomplete resources, but **no specific partner
has requested or validated this catalog yet.** Before M1 scales, we require either (a) a sarcoma lab,
consortium (e.g. EuroPDX, PPTC), or foundation (e.g. a Ewing-focused nonprofit) as a named
requestor/partner, or (b) ≥3 independent documented researcher requests. Until then we build a thin,
honest M0 and seek validation in parallel.

**Partner org: TO BE SECURED.** Candidate partners listed in §11; outreach is an M0 task.

---

## 3. Goals and non-goals

**Goals.**
- G1. A **normalized, versioned, openly-licensed metadata catalog** of Ewing sarcoma PDX, cell-line,
  and organoid models, each record traceable to ≥1 open source.
- G2. A published **"Model Datasheet" standard** — an honest description template (provenance,
  fusion status, access terms, molecular data availability, and explicit *unknowns*).
- G3. A **reproducible, auditable ingestion + validation pipeline** (re-runnable, diff-able, CI-gated).
- G4. A **cross-reference / identifier-resolution layer** mapping the same model across Cellosaurus,
  DepMap, PDCM Finder, PDMR, and literature.
- G5. **Per-assertion provenance** and **per-source license clearance** on 100% of published records.
- G6. A passing **re-identification review** by a qualified reviewer before any public release.

**Non-goals (constraints create identity — see also §5, §7).**
- N1. **Not a biobank or distributor.** We never host, ship, or broker biological material; we link
  to the providers' own access pages and MTAs.
- N2. **Not a data warehouse.** We do not re-host omics matrices, sequence data, or images; we link
  to the authoritative open accession and record its license.
- N3. **Not a clinical or patient-facing resource.** No diagnosis, prognosis, treatment, or trial
  advice. Any explanatory text is sourced, framed as research context, and out of scope for medical use.
- N4. **Not a curator of controlled-access or patient-level data.** dbGaP/EGA individual-level data,
  identifiable clinical records, and individual biobank entries are **out of scope** (see §7).
- N5. **Not an originator of facts.** We aggregate and cite; we do not assert novel biological claims.
- N6. **Not a license launderer.** We never redistribute non-commercial or restricted source content
  as if it were open; such sources are *referenced*, not *ingested* (see §7).

---

## 4. Success metrics (outcomes)

Outcome-centric, beneficiary-first. Vanity metrics (stars, page views) are explicitly *not* success.

| # | Outcome | Baseline | Target (v1.0) | How measured |
|---|---|---|---|---|
| O1 | Ewing models cataloged with complete, sourced records | 0 | ≥ 60 distinct models (PDX + cell line + organoid), realistic given known counts | Catalog row count w/ all required fields + ≥1 provenance |
| O2 | Records with per-assertion provenance + cleared license | 0 | **100%** of published records | Validation CI report |
| O3 | Cross-references resolved (same model across ≥2 sources) | 0 | ≥ 70% of cell-line models linked to Cellosaurus + DepMap where they exist | Resolver coverage report |
| O4 | Re-identification review pass before each public release | n/a | **100% of releases** | Signed review checklist in repo |
| O5 | Researcher time-to-find (independent task test) | ~hours (anecdotal) | < 10 min to locate + correctly cite a model + its access terms | Moderated task test w/ 3–5 researchers |
| O6 | Independent reuse | 0 | ≥ 3 external citations/uses (papers, tools, or consortium adoption) within 12 months of v1.0 | Citation/issue/fork tracking |
| O7 | Data freshness | n/a | Quarterly re-ingest; < 5% dead links at each release | Link-check CI |
| O8 | Corrections honored | n/a | Median time-to-correction < 14 days for valid reports | Issue tracker SLA |

O5 and O6 are the *real* success signals (a beneficiary saved time and someone reused the work). O1–O4
and O7 are quality preconditions, not the point.

---

## 5. Scope

**In scope.**
- Metadata for Ewing sarcoma **PDX, cell-line, and organoid** models that are **already openly
  described** in open-access literature or open repositories.
- Model attributes: canonical ID + aliases; model type; diagnosis & fusion-confirmation status;
  fusion partner/type *where openly reported*; broad anatomical context *only at a non-identifying
  granularity*; provider/repository; **access terms** (open / MTA / commercial); molecular-data
  **availability + accession + that accession's access tier**; drug-response data availability;
  references (PMID/PMCID/DOI); and **explicit unknowns**.
- Identifier cross-referencing across open registries.
- A Model Datasheet standard, the ingestion/validation pipeline, and a lightweight browse/query layer.

**Out of scope (hard).**
- ❌ Any **controlled-access** data (dbGaP, EGA individual-level, individual biobank records).
- ❌ Any **patient-identifiable or individual-level clinical** data (age in years, dates, location
  specifics, free-text clinical history, anything re-identifying for a rare pediatric disease).
- ❌ **Re-hosting** sequence/omics/image payloads or biological material.
- ❌ Redistribution of **non-commercial / restricted** source content (COSMIC, OncoKB cell-line
  annotations, vendor catalogs) — referenced only, never ingested.
- ❌ Any **medical advice, prognosis, or treatment/trial guidance.**
- ❌ Non-Ewing tumor models (beyond unavoidable cross-reference stubs).
- ❌ Scraping sources whose terms forbid it.

---

## 6. Solution approach & architecture

A small, boring, auditable **data pipeline + static catalog** — deliberately low-tech so it is
reproducible, reviewable, and cheap to maintain. No live database service to operate; the catalog
*is* version-controlled files.

**Pipeline (stages):**
1. **Source register** — `sources.yaml`: every source with its URL, license, license URL, access
   tier (open/aggregate/controlled), `ingest: true|false` (false = reference-only), and verification
   date + verifier. Nothing is ingested unless its row says so.
2. **Acquire** — fetch *only* `ingest:true` open records (PDCM Finder API, Cellosaurus release files,
   DepMap downloads, PDMR/CCDI open metadata, and structured extraction from PMC-OA full text).
   Raw pulls are cached with checksum + retrieval date.
3. **Extract & normalize** — map heterogeneous fields to the **Model Datasheet schema**; normalize
   identifiers, fusion nomenclature, and access-term vocabulary; record source field → catalog field
   lineage.
4. **Resolve** — link records describing the same model across sources into one canonical entity
   with `aliases[]`; resolution rules are explicit and logged (no silent merges).
5. **Provenance-stamp** — every field-level assertion carries `{value, source, sourceUrl, license,
   retrievedDate, assertedBy}`. No provenance → field is dropped, not guessed.
6. **Validate** — JSON-Schema validation + project rules (license cleared? provenance present?
   re-identification lint? dead-link check?). CI-gated; a failing record cannot publish.
7. **Re-identification review** — automated lint *plus* human reviewer sign-off (see §7, §8).
8. **Publish** — versioned release: `catalog.json`, `catalog.csv`, `datasheets/<modelId>.md`, a
   static browse site, and a machine-readable `LICENSE`/provenance bundle.

**Tech stack (locked decisions).**
- **TypeScript + ESM, pnpm workspaces** (per Elyos conventions); reuse `packages/schema` patterns
  (AJV/JSON-Schema draft-07) for the Model Datasheet schema.
- **Storage = files in Git** (JSON/CSV/Markdown). No server DB at M0–M2. Optional read-only static
  site (e.g. a generated table + client-side search) — no backend, no PII, nothing to breach.
- **Validation = AJV** + custom lint rules run in CI (GitHub Actions).
- **No LLM in the published artifact.** AI sessions assist *curation and extraction during tasks*;
  every AI-extracted assertion is provenance-stamped to its open source and **human/expert-reviewed**
  before publication. The catalog contains facts-with-sources, never model-generated claims.

**Data model (Model Datasheet — canonical fields, abbreviated).**
- `modelId` (canonical), `aliases[]`, `modelType` (pdx | cell-line | organoid | other),
  `diagnosis` (fixed: "Ewing sarcoma"), `fusionConfirmed` (bool), `fusion` (e.g. EWSR1-FLI1 /
  EWSR1-ERG / other; `fusionTypeIfReported`), `derivationContext` (coarse, non-identifying:
  primary | relapse | metastatic — **only if openly published and non-identifying**),
  `provider`, `accessTerms` (open | mta | commercial | unknown), `accessUrl`,
  `molecularData[]` (`{assay, accession, repository, accessTier, license}`),
  `drugResponseDataAvailable` (bool + refs), `references[]` (PMID/PMCID/DOI),
  `provenance[]` (per-assertion), `validationStatus`, `reidReviewStatus`, `lastVerified`, `notes`
  (sourced unknowns explicit).

**Key decisions.**
- D1. **Provenance is a first-class field, not metadata about metadata.** A record with an unsourced
  claim is invalid by construction.
- D2. **Reference-only vs. ingestible** is a per-source switch enforced in code (§7).
- D3. **Coarsen-by-default** for any potentially-identifying attribute; when in doubt, omit and note.
- D4. **Files over services** to minimize threat surface, maximize reproducibility, and make every
  change reviewable as a diff.

---

## 7. Data, licensing & compliance

> ### CANCER GUARDRAILS (BINDING — this section leads, and overrides anything below it)
> - **Open-access / aggregate / de-identified data ONLY.** Controlled-access sources (**dbGaP, EGA**,
>   individual-level biobanks) and **ANY identifiable patient data** are **OUT OF SCOPE** — they
>   require authorized access + IRB oversight, which is **not** what donated AI tasks provide.
> - **Per-source license verification is mandatory.** TCGA/GDC **open-tier** and **GEO** are open and
>   ingestible. **COSMIC** and **OncoKB** are **non-commercial / custom-licensed → FLAG; reference
>   only, never ingest/redistribute.** Vendor catalogs (ATCC/DSMZ) are proprietary → reference only.
> - **No medical advice.** Any patient-facing or explanatory text is **education only**, fully
>   **sourced**, labeled **"not medical advice"**, and **gated behind oncologist + patient-advocate
>   review (riskTier = high)** before publication. The core catalog avoids such text entirely.
> - **Provenance on every assertion.** No claim is published without `{source, sourceUrl, license,
>   retrievedDate}`. Unsourced = invalid.
> - **Re-identification is the live risk for a rare pediatric cancer.** Even "de-identified" model
>   metadata can re-identify when tumor counts are tiny. We coarsen by default and require human
>   re-identification review before every release (see below).

**Source register (initial — each verified before ingest; `ingest` column is binding):**

| Source | Content | License (to verify) | Access tier | Ingest? |
|---|---|---|---|---|
| **PDCM Finder / PDX Finder** (EMBL-EBI) | PDX/organoid/cell-line model metadata | Open (verify CC0/CC-BY + terms) | open metadata | ✅ (after verify) |
| **Cellosaurus** (SIB) | Cell-line identity, RRIDs, cross-refs | **CC BY 4.0** | open | ✅ |
| **DepMap** (Broad) | Cell-line models + dependencies | CC BY 4.0 (verify per-file) | open | ✅ (metadata) |
| **NCI PDMR** | Patient-derived models repository metadata | US Gov / public (verify) | open metadata | ✅ (after verify) |
| **CCDI** (NCI Childhood Cancer Data Initiative) | Pediatric model/data metadata | Mixed (verify per resource) | open + controlled | ✅ open only |
| **PPTC / PPTP** | Pediatric preclinical models + molecular | Per-publication (verify) | open (published) | ✅ metadata/refs |
| **CSTN** (St. Jude Childhood Solid Tumor Network) | Orthotopic PDX, openly described | Per-resource (verify) | open metadata | ✅ (after verify) |
| **GEO / SRA** (open records only) | Omics accessions for models | Open (NCBI) | open | ✅ link only |
| **PMC Open Access subset** | Model descriptions in papers | CC-BY/CC0 per article (verify each) | open | ✅ per-article |
| **EuroPDX / ProXe / Cell Model Passports** | PDX/leukemia/organoid metadata | Verify each | open metadata | ✅ (after verify) |
| **COSMIC** | Cell-line mutation annotations | **Non-commercial / custom → FLAG** | restricted | ❌ reference only |
| **OncoKB** | Variant/therapy annotations | **Non-commercial / custom → FLAG** | restricted | ❌ reference only |
| **dbGaP / EGA (individual-level)** | Controlled genomic/clinical | Controlled | controlled | ❌ OUT OF SCOPE |
| **ATCC / DSMZ / vendors** | Commercial catalog entries | Proprietary | commercial | ❌ reference only |

**Provenance model.** Each field-level assertion stores `{value, source, sourceUrl, license,
retrievedDate, assertedBy, confidence}`. A `provenance.json` accompanies each release and is the
auditable record of *what we claim and why*.

**PII / patient-data stance.** **Zero patient-level data.** The catalog describes *models*, not
people. We never record patient age-in-years, dates, geographic specifics, sex when combined with
rare features, free-text clinical narrative, or any combination that could re-identify. Where a
source's openly-published model name encodes such detail, we use a neutral canonical ID and do not
propagate the encoding.

**Re-identification review (the binding gate, beyond standard provenance).** Before every public
release, (1) an automated **re-id lint** flags small-cell-count attributes, free-text leakage, and
identifying combinations; (2) a qualified human reviewer (sarcoma-domain or clinical-genomics
literacy) signs a checklist confirming no record is plausibly re-identifying. **No sign-off → no
release.** Recorded in-repo as `reid-review/<release>.md`.

**Attribution & output license.** Catalog **data/content: CC BY 4.0**; pipeline **code: MIT**. Every
record carries upstream attribution; sources requiring share-alike or non-commercial terms are
handled per their license (referenced, not ingested). Where any ingested source is share-alike, that
subset is segregated and labeled to avoid contaminating the CC-BY whole.

**Refusal triggers (Elyos guardrails applied).** Stop and flag if a task would: ingest
controlled-access/identifiable data; redistribute non-commercial content as open; assert an unsourced
biological claim; produce medical advice; or primarily serve a for-profit (e.g. a vendor's catalog).

---

## 8. Quality, review & risk gates

**Risk tier: medium.** Rationale: domain-accuracy-sensitive data project with a real
re-identification surface, but **no patient-facing medical advice** (which would force high). Any
optional explanatory/patient-facing content is split into a **separate high-risk track** gated behind
**oncologist + patient-advocate** review — and is *not* part of v1.0.

**Required reviews before a deed is "done":**
- **Curation/data review** (every record): a second curator verifies each assertion against its cited
  source; license cleared; identifiers correct.
- **License/provenance gate** (CI + human): 100% records sourced + cleared; `ingest` flags respected.
- **Re-identification review** (human sign-off): mandatory before each public release.
- **Domain accuracy review** (medium-risk requirement): a reviewer with sarcoma/cancer-genomics
  literacy spot-checks fusion calls, model-type classification, and cross-references.
- **High-risk track only** (if/when explanatory text is added): credentialed **oncologist +
  patient-advocate** sign-off, "not medical advice" framing, before merge.

**Definition of Shipped.** A release ships when: schema-valid; 100% provenance + license-cleared;
re-id review signed; domain spot-check passed; dead-link rate < 5%; CHANGELOG + versioned release
published under CC BY 4.0 (data) / MIT (code). **Delivered, not merged:** a real researcher can use it
to find and cite models (O5), and the work is handed to a steward/partner for upkeep.

---

## 9. Roadmap & milestones

Phased, with measurable **exit criteria**. M0 is a deliberately thin, honest cold-start that proves
the rules before scaling.

**M0 — Foundation & cold-start (rules before data).**
Goal: lock the schema, license register, provenance + re-id model, and seed a tiny exemplar set from
unambiguously-open sources; begin partner outreach.
- Exit: Model Datasheet schema v0.1 + validator merged; `sources.yaml` with ≥10 sources each
  license-verified and `ingest` flagged; provenance model implemented; re-id lint v0 + review
  checklist authored; **5–10 seed models** fully sourced, validated, and re-id-reviewed; partner
  outreach started (≥3 contacts logged); `verifiedNeed` status documented.

**M1 — First open catalog (breadth from open registries).**
Goal: ingest the clearly-open registries (Cellosaurus, DepMap, PDCM Finder, PDMR open metadata),
normalize, resolve cross-references, publish v0.1.
- Exit: ≥ 30 models published with full provenance + license; resolver links cell-line models to
  Cellosaurus + DepMap where they exist; CI green; re-id review signed; v0.1 release tagged CC BY 4.0.

**M2 — Enrichment & molecular-availability index.**
Goal: add fusion detail (where openly reported), molecular-data availability + accessions (link-only,
access-tier labeled), drug-response availability, and literature-extracted models from PMC-OA.
- Exit: ≥ 60 models (O1 target); ≥ 70% cross-ref coverage (O3); molecular-availability index with
  access-tier labels; PMC-OA extraction reviewed; domain accuracy spot-check passed; v0.5 release.

**M3 — Validation, community & sustainability.**
Goal: contribution workflow, correction SLA, quarterly re-ingest automation, researcher task-test
(O5), partner handoff, optional static browse site.
- Exit: O5 task-test passed (< 10 min) with 3–5 researchers; contribution + correction process live
  with SLA (O8); quarterly re-ingest CI; steward/partner named; v1.0 release + Definition of Shipped
  met. (Optional high-risk explanatory track only opens here, gated per §7/§8.)

Dependencies: M1 depends on M0 schema + license register; M2 depends on M1 canonical entities; M3
depends on M2 coverage + a secured partner/steward (or O5 self-validation if partner still pending).

---

## 10. Work breakdown

The itemized, schema-mapped backlog lives in **`TASKS.md`** — milestone tables (`ID | Title | Type |
Size | Risk | Deliverable | Depends on | Reviewer`), acceptance criteria for the key tasks per
milestone, each milestone's Definition of Done, a backlog of sized-but-unscheduled work, and a
complete schema-valid example Task JSON for the first M0 task. All tasks are **donated lane** and
**`verifiedNeed = false`** until a partner/requestor is secured.

---

## 11. Governance, roles & stakeholders

- **Maintainer (Owner): TBD** — accountable for schema, releases, license register integrity.
- **Curators (rotation, ≥2):** ingest/normalize records; every record needs a second-curator review.
- **Domain reviewer (medium-risk):** sarcoma / cancer-genomics literacy; verifies fusion calls,
  model classification, cross-references. **TO BE SECURED.**
- **Re-identification reviewer:** signs the re-id checklist before each release (may be the domain
  reviewer if qualified). **TO BE SECURED.**
- **Steward (last-mile owner):** ensures the catalog actually reaches and is used by researchers;
  owns the O5 task-test and partner handoff. **TBD.**
- **Partner / requestor:** sarcoma lab, consortium, or foundation validating need + scope.
  **TO BE SECURED.** Candidates: EuroPDX, PPTC, CSTN/St. Jude, a Ewing-focused foundation.
- **High-risk expert reviewers (only if explanatory track opens):** credentialed **oncologist** +
  **patient-advocate**. **TO BE SECURED.**
- **Conflict-of-interest rule:** no contributor may shape the catalog to favor a vendor or their own
  unpublished models; vendor catalogs are reference-only by policy (§7).

---

## 12. Dependencies & integrations

- **External data sources** (per §7 register): PDCM Finder API, Cellosaurus releases, DepMap
  downloads, NCI PDMR, CCDI (open), PPTC publications, CSTN, GEO/SRA (links), PMC-OA, EuroPDX /
  ProXe / Cell Model Passports.
- **Identifier authorities:** Cellosaurus RRIDs, DepMap `ACH-*`, repository model IDs, PubMed/PMC,
  DOIs.
- **Elyos platform:** `packages/schema` (schema patterns/validation), CLI task-prep flow, registry
  entry, CI/governance workflows, the good-deed definition + refusal guardrails.
- **Tooling:** Node/TypeScript, pnpm, AJV, GitHub Actions (validation, link-check, scheduled
  re-ingest), static-site generator (M3, optional).
- **Upstream risk:** sources may change schemas, licenses, or availability; the `sources.yaml`
  verification dates + cached checksums make drift detectable.

---

## 13. Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|
| **Re-identification** of a patient via rare-model metadata | Medium | **High** | Coarsen-by-default; re-id lint + mandatory human sign-off per release; no individual-level fields; omit-when-in-doubt | Re-id reviewer |
| Ingesting controlled-access data by mistake | Low | **High** | `ingest:false` enforced in code for controlled sources; CI blocks; dbGaP/EGA hard-excluded | Maintainer |
| License violation (COSMIC/OncoKB/vendor redistribution) | Medium | High | Per-source license register; reference-only switch; CI license-clearance check | Maintainer |
| Inaccurate fusion/model classification | Medium | Medium | Domain reviewer spot-check; provenance enables audit; confidence field | Domain reviewer |
| Unsourced/AI-hallucinated assertion leaks in | Medium | High | Provenance-required-by-construction; second-curator review; AI output never published unreviewed | Curators |
| No partner / unvalidated need → wasted effort | **Medium** | Medium | Thin M0; outreach early; `verifiedNeed=false` honest; O5 self-validation fallback | Steward |
| Source drift / dead links | High | Low–Med | Cached checksums + retrieval dates; quarterly re-ingest; link-check CI; <5% target | Maintainer |
| Duplicate/wrong cross-reference merges | Medium | Medium | Explicit logged resolution rules; no silent merges; reversible aliases | Curators |
| Scope creep into clinical/patient guidance | Medium | High | Hard non-goals; high-risk track firewalled + expert-gated | Maintainer |
| Maintainer bus factor | Medium | Medium | ≥2 curators; files-in-Git (no bespoke infra); documented runbook | Maintainer |

---

## 14. Security & privacy

- **Threat surface is intentionally minimal:** static files in Git, no server DB, no user accounts,
  no PII collected. The main risk is *publishing the wrong data*, not a breach — hence the §7/§8 gates.
- **No secrets in repo.** API keys for source fetches (if any) live in CI secrets/env, never
  committed, never logged in receipts (per Elyos rule).
- **PII:** none collected or stored by design; re-identification is the residual risk and is gated.
- **Abuse/misuse prevention:** catalog is research metadata; explicit "not medical advice"; no
  material brokering; vendor-neutral; refusal triggers (§7) stop harmful or license-violating tasks.
- **Supply-chain:** pin dependencies; checksum cached source pulls; CI runs on least privilege.

---

## 15. Sustainability & maintenance

- **After delivery:** maintainer + curator rotation keep the catalog fresh via **quarterly re-ingest**
  CI; corrections handled with a published **SLA (<14-day median, O8)**.
- **Low operating cost by design:** no server to run; releases are Git tags; the static site (if any)
  is hosted free (GitHub Pages or similar).
- **Outcome tracking:** O5 task-tests repeated at major releases; O6 reuse tracked via citations,
  forks, and consortium adoption; a public "coverage / what's missing" ledger each release.
- **Handoff:** ideal end-state is adoption/co-maintenance by a sarcoma consortium or foundation;
  until then Elyos stewards it. A documented runbook ensures continuity.

---

## 16. Open questions

1. **Partner/requestor:** who validates need and scope? (EuroPDX / PPTC / CSTN / a Ewing foundation?)
2. **Re-id threshold:** what cell-count / attribute-combination rules define "too identifying" for a
   rare pediatric cancer? Needs a qualified reviewer to set the policy.
3. **PDCM Finder license:** confirm exact license/terms for ingest vs. link-only.
4. **Derivation context granularity:** is even coarse primary/relapse/metastatic safe to publish per
   model, or should it be omitted absent explicit open publication? (Default: omit when uncertain.)
5. **PMC-OA extraction scope:** how aggressively to mine papers vs. registries first? (Default:
   registries first; literature in M2.)
6. **Does the high-risk explanatory track ever open**, or do we stay strictly research-infrastructure?
7. **Cross-reference to non-Ewing models** for context — allowed as stubs, or strictly excluded?

---

## 17. References

- Elyos: `CLAUDE.md` (work rules, lanes, guardrails); `docs/good-deed-definition.md` (criteria + risk
  tiers); `packages/schema/src/schemas.ts` (Task schema); `planning/ROADMAP.md` (Track 8 guardrails).
- Sources (to be license-verified in M0): PDCM Finder / PDX Finder (EMBL-EBI); Cellosaurus (SIB,
  CC BY 4.0); DepMap (Broad, CC BY 4.0); NCI PDMR; NCI CCDI; PPTC/PPTP; CSTN (St. Jude); GEO/SRA
  (NCBI); PMC Open Access subset; EuroPDX; ProXe; Cell Model Passports (Sanger).
- Flagged non-open (reference only): COSMIC, OncoKB (non-commercial/custom); ATCC/DSMZ (commercial);
  dbGaP/EGA (controlled — out of scope).
- Standards influence: "Datasheets for Datasets" (Gebru et al.); FAIR data principles; RRID/
  Cellosaurus identifier practice.

---

## Appendix A — Improvements applied

The following 25 specific improvements were identified against the first draft and **applied** in the
text above.

1. **Led §7 with the binding cancer guardrails** as a quoted block that explicitly *overrides* the
   rest of the section, per the prompt's hard requirement.
2. **Made `ingest: true|false` a code-enforced, per-source switch** in `sources.yaml` and the §7
   table, so non-open sources cannot be ingested by construction (not just by policy).
3. **Promoted re-identification from a generic risk to a named binding release gate** (§7, §8, §13)
   with an automated lint *plus* mandatory human sign-off — appropriate for a rare pediatric cancer.
4. **Justified the medium (not low, not high) risk tier explicitly** and firewalled any patient-facing
   text into a separate high-risk, oncologist+advocate-gated track excluded from v1.0.
5. **Added per-source license verification dates + verifier** to the source register so license drift
   is detectable and auditable.
6. **Flagged COSMIC and OncoKB by name as non-commercial → reference-only**, exactly as the guardrails
   demand, and added vendor catalogs (ATCC/DSMZ) and dbGaP/EGA to the exclusion list.
7. **Made provenance invalid-by-construction** (D1): a record with an unsourced field fails CI rather
   than being published with a gap.
8. **Set `verifiedNeed = false` everywhere and stated it in the executive summary**, with a concrete
   bar (named partner *or* ≥3 documented researcher requests) to flip it.
9. **Replaced vanity metrics with outcome metrics** (§4), explicitly calling out O5 time-to-find and
   O6 reuse as the *real* success and the rest as preconditions.
10. **Added a coarsen-by-default rule (D3)** for any potentially-identifying attribute, with
    "omit when in doubt" as the operating default.
11. **Separated "ingest" from "re-host"** — clarified we link to omics accessions and record their
    access tier, never re-hosting payloads (N2), keeping the threat surface tiny.
12. **Added an access-tier label to every molecular-data link** so an open catalog never implies open
    access to a controlled accession behind it.
13. **Specified the cross-reference resolver as explicit, logged, reversible** (no silent merges) to
    prevent wrong-model conflation (a real curation hazard).
14. **Chose files-in-Git over a database** (D4) and justified it by reproducibility + minimal security
    surface (§6, §14), avoiding running PII-bearing infrastructure.
15. **Stated the catalog contains no LLM-generated claims** — AI assists curation, but every assertion
    is provenance-stamped to an open source and human-reviewed before publishing.
16. **Named concrete candidate partners** (EuroPDX, PPTC, CSTN/St. Jude, a Ewing foundation) instead
    of an abstract "TO BE SECURED," and made outreach an M0 exit criterion.
17. **Added an O5 moderated researcher task-test** as the human-centered validation of the whole
    project, with a concrete <10-minute target.
18. **Added a corrections SLA (O8, <14-day median)** and a quarterly re-ingest cadence so the catalog
    stays alive and trustworthy after delivery.
19. **Added a dead-link metric and link-check CI (O7)** because orphaned accessions are the exact pain
    this project exists to fix.
20. **Made the honest benefit-to-families statement explicit** (§2): real but indirect; this is
    research infrastructure, not a patient resource — no overclaiming to vulnerable readers.
21. **Added "explicit unknowns" as a required Model Datasheet element**, so the catalog states what is
    *not* known rather than implying false completeness.
22. **Segregated any share-alike-licensed subset** so it cannot contaminate the CC-BY-licensed whole
    (§7 attribution paragraph).
23. **Added a conflict-of-interest governance rule** (§11) forbidding shaping the catalog to favor a
    vendor or a contributor's own models; vendor entries are reference-only.
24. **Tied refusal triggers directly to the Elyos guardrails** (§7) so curators have a concrete
    stop-and-flag list (controlled data, license violation, unsourced claim, medical advice, for-profit).
25. **Sequenced M0 as "rules before data"** — schema, license register, provenance, and re-id review
    are built and proven on 5–10 seed models *before* any breadth ingestion, de-risking the scale-up.

---

## Review sign-off

**Reviewed against:** PLAN_SPEC (17 required H2 sections, order, metadata header, honest tone),
CLAUDE.md (agent-neutral core, donated-lane rules, refusal guardrails, no secrets, quality bar),
good-deed-definition.md (5 criteria + risk tiers), and the binding cancer guardrails.

**Completeness:** All 17 required sections present and in order (Executive summary → References),
plus the metadata header. §7 leads with the cancer guardrails. Risks table uses the required
columns. TASKS.md carries the schema-mapped backlog.

**Correctness checks performed and fixed:**
- ✅ Risk tier (medium) justified and consistent across §1/§8/§13; high-risk explanatory track
  firewalled and expert-gated — consistent with the good-deed risk-tier rules.
- ✅ `verifiedNeed = false` stated in §1, §2, §10, §11 and reflected throughout TASKS.md.
- ✅ Controlled-access (dbGaP/EGA) and non-commercial (COSMIC/OncoKB) sources excluded/flagged in
  §3, §5, §7, §13 — no contradictions.
- ✅ Donated lane only; no funded tasks, so no `fundedBudgetUsd` obligations triggered.
- ✅ Output licensing (CC BY 4.0 data / MIT code) stated in §7 and §8 and carried into TASKS.md.
- ✅ No secrets-in-repo and no medical advice — consistent with §14, §7, and CLAUDE.md.

**Residual items requiring a human decision** (tracked in §16): secure a partner/requestor; set the
re-identification threshold policy; confirm PDCM Finder ingest license; decide whether the high-risk
explanatory track ever opens.

**Sign-off:** Plan is internally consistent, guardrail-compliant, and ready for maintainer review.
Status remains **Draft v0.1.0** pending partner validation and the M0 license verifications.
