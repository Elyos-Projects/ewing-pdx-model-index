# ewing-pdx-model-index — Task backlog

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) ·
> Lane: donated · Risk tier: medium
>
> Companion to `PLAN.md`. Every task here maps to an Elyos **Task JSON**
> (`packages/schema/src/schemas.ts`). All tasks are **donated lane** and **`verifiedNeed = false`**
> until a sarcoma-research partner or ≥3 documented researcher requests are secured.

---

## How these tasks map to Elyos

Each row below becomes a Task JSON validated against `taskSchema`. Field mapping:

- `id` — stable slug `ewing-pdx-<area>-NNN` (also the table ID).
- `title` — the table Title.
- `project` — `"ewing-pdx-model-index"`.
- `type` — one of `code | research | writing | data | design-spec | maintenance` (table "Type").
- `lane` — `"donated"` (no funded tasks in this backlog).
- `priority` — `high | medium | low`.
- `domain` — array, e.g. `["cancer-research","ewing-sarcoma","bioinformatics","open-data"]`.
- `riskTier` — `low | medium | high` (table "Risk").
- `urgent` — boolean (default `false`; nothing here is time-critical).
- `deliverable` — `pr | dataset | document | translation` (table "Deliverable").
- `tokenEstimate` — `small | medium | large` (table "Size").
- `status` — `open` for all (backlog).
- `context`, `objective`, `acceptanceCriteria[]`, `output` — required prose/criteria.
- `resources[]` — source URLs / spec paths.
- `requestor` — omitted/`"TBD"` until secured.
- `verifiedNeed` — **`false`** (no partner yet).
- `outputLicense` — `"MIT"` (code) or `"CC-BY-4.0"` (data/content).
- `fundedBudgetUsd` — n/a (donated lane only).

---

## Milestone M0 — Foundation & cold-start (rules before data)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-pdx-schema-001 | Define Model Datasheet schema v0.1 (JSON-Schema + fields) | design-spec | medium | medium | document | — | Maintainer + Domain reviewer |
| ewing-pdx-licen-002 | Build `sources.yaml` license register (≥10 sources, verified, ingest flags) | research | medium | medium | document | — | Maintainer |
| ewing-pdx-prov-003 | Implement per-assertion provenance model + validator | code | medium | low | pr | ewing-pdx-schema-001 | Maintainer |
| ewing-pdx-reid-004 | Author re-identification lint v0 + reviewer checklist | code | medium | high | pr | ewing-pdx-schema-001 | Re-id reviewer |
| ewing-pdx-seed-005 | Seed 5–10 exemplar Ewing models (fully sourced + re-id reviewed) | data | medium | medium | dataset | 001,002,003,004 | Domain + Re-id reviewer |
| ewing-pdx-part-006 | Partner/need outreach + verified-need decision log | research | small | low | document | — | Steward |

**Acceptance criteria — key M0 tasks**

`ewing-pdx-schema-001` (Model Datasheet schema):
- [ ] JSON-Schema (draft-07, AJV-compatible) covering all canonical fields in PLAN §6, incl.
  `provenance[]`, `accessTerms`, `molecularData[].accessTier`, `reidReviewStatus`, and **required
  `explicitUnknowns`**.
- [ ] Every potentially-identifying field documented with a coarsen/omit rule.
- [ ] Validates the 5–10 seed records; rejects a record with a missing-provenance field.
- [ ] Reuses `packages/schema` patterns; merged with example datasheet + field docs.

`ewing-pdx-licen-002` (license register):
- [ ] ≥10 sources, each with license, license URL, access tier, `ingest:true|false`, verifier +
  date.
- [ ] COSMIC, OncoKB, ATCC/DSMZ marked `ingest:false` (reference-only); dbGaP/EGA marked out-of-scope.
- [ ] Cellosaurus/DepMap confirmed CC BY 4.0; PDCM Finder/PDMR/CCDI/CSTN terms verified or flagged
  "needs confirmation" (not silently assumed).

`ewing-pdx-reid-004` (re-identification gate):
- [ ] Automated lint flags small-cell-count attributes, free-text leakage, identifying combinations.
- [ ] Human reviewer checklist authored; sign-off recorded at `reid-review/<release>.md`.
- [ ] CI blocks release if checklist unsigned. Reviewed by a qualified (sarcoma/clinical-genomics) reviewer.

`ewing-pdx-seed-005` (seed set):
- [ ] 5–10 Ewing models from unambiguously-open sources, every field provenance-stamped + license-cleared.
- [ ] Second-curator review + domain spot-check + re-id sign-off complete.

**M0 Definition of Done.** Schema v0.1 + validator merged; license register with ingest flags;
provenance model implemented; re-id lint + checklist live; 5–10 seed models published & all gates
passed; partner outreach logged (≥3 contacts) with a documented `verifiedNeed` decision.

---

## Milestone M1 — First open catalog (breadth from open registries)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-pdx-ingest-007 | Cellosaurus + DepMap ingest → normalize to schema | code | large | medium | pr | ewing-pdx-schema-001, ewing-pdx-prov-003 | Maintainer |
| ewing-pdx-ingest-008 | PDCM Finder + NCI PDMR open-metadata ingest | code | large | medium | pr | ewing-pdx-licen-002, ewing-pdx-ingest-007 | Maintainer + Domain |
| ewing-pdx-resolve-009 | Identifier cross-reference resolver (logged, reversible) | code | medium | medium | pr | ewing-pdx-ingest-007 | Curators |
| ewing-pdx-ci-010 | CI gates: schema + license + provenance + dead-link checks | code | medium | low | pr | ewing-pdx-prov-003 | Maintainer |
| ewing-pdx-rel-011 | Publish v0.1 catalog release (CC BY 4.0) | data | medium | medium | dataset | 007,008,009,010, ewing-pdx-reid-004 | Maintainer + Re-id reviewer |

**Acceptance criteria — key M1 tasks**

`ewing-pdx-ingest-007` (Cellosaurus + DepMap):
- [ ] Re-runnable, idempotent fetch with cached checksums + retrieval dates.
- [ ] Each record provenance-stamped, license = CC BY 4.0, normalized identifiers + `aliases[]`.
- [ ] No controlled/non-open fields ingested; only metadata, no omics payloads re-hosted.

`ewing-pdx-resolve-009` (resolver):
- [ ] Same model across sources merged into one canonical entity; rules explicit + logged; no silent
  merges; merges reversible.
- [ ] Conflicting field values surfaced with per-source provenance, not overwritten.

`ewing-pdx-rel-011` (v0.1 release):
- [ ] ≥30 models, 100% provenance + license-cleared, CI green, re-id review signed.
- [ ] `catalog.json` + `catalog.csv` + `provenance.json` + CHANGELOG tagged under CC BY 4.0.

**M1 Definition of Done.** ≥30 models published with full provenance/license; resolver linking
cell-line models to Cellosaurus + DepMap where they exist; all CI gates green; re-id review signed;
v0.1 release tagged.

---

## Milestone M2 — Enrichment & molecular-availability index

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-pdx-fusion-012 | Add fusion detail (partner/type) where openly reported | data | medium | medium | dataset | ewing-pdx-rel-011 | Domain reviewer |
| ewing-pdx-omics-013 | Molecular-data availability index (accession + access-tier label) | data | large | medium | dataset | ewing-pdx-rel-011 | Maintainer + Domain |
| ewing-pdx-lit-014 | PMC-OA literature extraction of model descriptions (reviewed) | research | large | medium | dataset | ewing-pdx-schema-001 | Domain + Curators |
| ewing-pdx-drug-015 | Drug-response data availability flags + references | data | medium | medium | dataset | ewing-pdx-rel-011 | Domain reviewer |
| ewing-pdx-rel-016 | Publish v0.5 catalog release | data | medium | medium | dataset | 012,013,014,015 | Maintainer + Re-id reviewer |

**Acceptance criteria — key M2 tasks**

`ewing-pdx-omics-013` (molecular availability):
- [ ] Each molecular entry = `{assay, accession, repository, accessTier, license}`; **access tier
  labeled** so an open catalog never implies open access to a controlled accession.
- [ ] Links only; no sequence/matrix/image payloads re-hosted; dead-link checked.

`ewing-pdx-lit-014` (PMC-OA extraction):
- [ ] Only PMC Open Access subset articles (license verified per article).
- [ ] Every AI-extracted assertion provenance-stamped to the article + human-reviewed before publish;
  no unsourced or model-originated claims.

**M2 Definition of Done.** ≥60 models (O1); ≥70% cross-ref coverage (O3); molecular-availability index
with access-tier labels; PMC-OA extraction reviewed; domain spot-check passed; v0.5 released.

---

## Milestone M3 — Validation, community & sustainability

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| ewing-pdx-test-017 | Moderated researcher task-test (O5 <10-min find+cite) | research | medium | low | document | ewing-pdx-rel-016 | Steward |
| ewing-pdx-contrib-018 | Contribution + correction workflow with SLA (O8) | writing | medium | low | document | ewing-pdx-rel-016 | Maintainer |
| ewing-pdx-fresh-019 | Quarterly re-ingest automation + drift detection | maintenance | medium | low | pr | ewing-pdx-ci-010 | Maintainer |
| ewing-pdx-site-020 | Optional static browse/search site (no backend, no PII) | code | medium | low | pr | ewing-pdx-rel-016 | Maintainer |
| ewing-pdx-rel-021 | Publish v1.0 + Definition of Shipped handoff | data | small | medium | dataset | 017,018,019 | Maintainer + Steward |

**Acceptance criteria — key M3 tasks**

`ewing-pdx-test-017` (researcher task-test):
- [ ] 3–5 independent researchers find, compare, and correctly cite a model + its access terms in
  <10 min; results + friction logged; fixes filed.

`ewing-pdx-rel-021` (v1.0 + handoff):
- [ ] All Definition of Shipped criteria met (PLAN §8); steward/partner named; runbook documented.

**M3 Definition of Done.** O5 task-test passed; contribution + correction process live with SLA;
quarterly re-ingest CI running; steward/partner named; v1.0 released; outcomes-tracking ledger live.

---

## Backlog / future (sized, unscheduled)

| ID | Title | Type | Size | Risk | Notes |
|---|---|---|---|---|---|
| ewing-pdx-europdx-101 | EuroPDX / ProXe / Cell Model Passports ingest | code | large | medium | After license verification per source |
| ewing-pdx-orphan-102 | Orphaned-accession reconciliation tooling | code | medium | low | Reduce dead links below 2% |
| ewing-pdx-gaps-103 | "What's missing" coverage-gap ledger per release | writing | small | low | Feeds funding/gap analysis (O6) |
| ewing-pdx-rrid-104 | RRID/Cellosaurus round-trip validation | code | small | low | Identifier integrity |
| ewing-pdx-edu-201 | (HIGH-RISK, gated) sourced research-context explainer | writing | medium | **high** | Only if §7 track opens; oncologist + patient-advocate sign-off + "not medical advice" |
| ewing-pdx-i18n-202 | Translate catalog field labels/docs (not patient content) | writing | small | low | UI/docs only |

---

## Generated task index

Every table row above now has an executable, schema-valid `tasks/<id>.json` (validated against
`taskSchema`), so `elyos browse`/`pull` can process the full backlog — not just the seed. All tasks
are **donated lane**, `status: open`, `verifiedNeed: false`, `requestor: TBD` until a partner or ≥3
researcher requests are secured.

**Fan-out:** none. Each `TASKS.md` row maps to exactly one task JSON (representative tasks only). The
backlog has no plan-enumerated fan-out dimension — the seed set (`-seed-005`) is sized 5–10 but the
specific models are not yet named, the breadth ingests are already separate per-source rows
(`-ingest-007/008`, `-europdx-101`), and no target-language set is enumerated for `-i18n-202`. Those
items expand into concrete per-item tasks only on partner/scope confirmation; no models, datasets,
languages, or beneficiaries were fabricated.

**Guardrail notes:** `ewing-pdx-edu-201` is HIGH-RISK and gated — kept as a sourced,
non-advice research-context explainer requiring oncologist + patient-advocate sign-off and a "not
medical advice" disclaimer, opening only if the PLAN §7 track opens; it never authors medical advice,
prognosis, eligibility, or trial/treatment rankings (those remain refused). `ewing-pdx-reid-004` is
the binding re-identification release gate. All data/ingest tasks carry the PLAN §7 cancer guardrails
(open/aggregate/de-identified metadata only; controlled-access out of scope; COSMIC/OncoKB
reference-only; links-only, no payloads re-hosted; per-assertion provenance).

Generated ids (26, excluding the pre-existing seed `ewing-pdx-schema-001`):

- **M0:** `ewing-pdx-licen-002`, `ewing-pdx-prov-003`, `ewing-pdx-reid-004`, `ewing-pdx-seed-005`, `ewing-pdx-part-006`
- **M1:** `ewing-pdx-ingest-007`, `ewing-pdx-ingest-008`, `ewing-pdx-resolve-009`, `ewing-pdx-ci-010`, `ewing-pdx-rel-011`
- **M2:** `ewing-pdx-fusion-012`, `ewing-pdx-omics-013`, `ewing-pdx-lit-014`, `ewing-pdx-drug-015`, `ewing-pdx-rel-016`
- **M3:** `ewing-pdx-test-017`, `ewing-pdx-contrib-018`, `ewing-pdx-fresh-019`, `ewing-pdx-site-020`, `ewing-pdx-rel-021`
- **Backlog:** `ewing-pdx-europdx-101`, `ewing-pdx-orphan-102`, `ewing-pdx-gaps-103`, `ewing-pdx-rrid-104`, `ewing-pdx-edu-201`, `ewing-pdx-i18n-202`

Total task files in `tasks/`: **27** (1 seed + 26 generated).

---

## Example task JSON (first M0 task, schema-valid)

```json
{
  "id": "ewing-pdx-schema-001",
  "title": "Define Ewing Model Datasheet schema v0.1 (JSON-Schema + field docs)",
  "project": "ewing-pdx-model-index",
  "type": "design-spec",
  "lane": "donated",
  "priority": "high",
  "domain": ["cancer-research", "ewing-sarcoma", "open-data", "bioinformatics", "metadata-schema"],
  "riskTier": "medium",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "medium",
  "status": "open",
  "context": "Ewing sarcoma model metadata is scattered across PDCM Finder, Cellosaurus, DepMap, NCI PDMR, CCDI, PPTC, CSTN, and open-access papers with non-interoperable identifiers and no shared, provenance-aware schema. Before any breadth ingestion, the project must lock an honest 'Model Datasheet' standard so every later record is schema-valid, fully sourced, and re-identification-safe. This is the 'rules before data' foundation task (PLAN M0). No partner/requestor is secured yet, so verifiedNeed is false. Binding cancer guardrails apply: open-access/aggregate/de-identified data only; no controlled-access (dbGaP/EGA) or patient-identifiable data; COSMIC/OncoKB reference-only; provenance on every assertion; no medical advice.",
  "objective": "Produce a draft-07 JSON-Schema (AJV-compatible) plus field documentation defining the canonical Ewing Model Datasheet, including per-assertion provenance, access-terms vocabulary, molecular-data access-tier labeling, a required explicitUnknowns field, and coarsen/omit rules for every potentially-identifying attribute.",
  "acceptanceCriteria": [
    "JSON-Schema covers all PLAN §6 canonical fields: modelId, aliases[], modelType, diagnosis, fusionConfirmed, fusion, derivationContext, provider, accessTerms, accessUrl, molecularData[] (assay, accession, repository, accessTier, license), drugResponseDataAvailable, references[], provenance[] (value, source, sourceUrl, license, retrievedDate, assertedBy, confidence), validationStatus, reidReviewStatus, lastVerified, explicitUnknowns.",
    "Schema rejects any record containing a field assertion that lacks a provenance entry.",
    "Every potentially-identifying field documents an explicit coarsen-or-omit rule (omit when in doubt).",
    "Schema validates the 5-10 seed records (ewing-pdx-seed-005) and is reviewed by the maintainer and a domain reviewer.",
    "Reuses packages/schema AJV/draft-07 patterns; merged with an example datasheet and field-level docs."
  ],
  "resources": [
    "C:/code/elyos/packages/schema/src/schemas.ts",
    "C:/code/elyos/docs/good-deed-definition.md",
    "https://www.cellosaurus.org/",
    "https://depmap.org/portal/",
    "https://www.cancermodels.org/",
    "Gebru et al., Datasheets for Datasets"
  ],
  "output": "A merged PR adding ewing-model-datasheet.schema.json (draft-07), field documentation, and an example datasheet, validating the seed records and rejecting unsourced assertions.",
  "requestor": "TBD",
  "verifiedNeed": false,
  "outputLicense": "MIT"
}
```

> Note: data/dataset deliverables in this backlog publish under **CC-BY-4.0**; code/schema/pipeline
> deliverables publish under **MIT**. No funded tasks exist here, so `fundedBudgetUsd` is never
> required; all tasks carry `verifiedNeed: false` until a partner/requestor is secured.
