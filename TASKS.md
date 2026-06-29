# TASKS — oncogene-knowledge-graph

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated · Risk tier: medium

Itemized backlog for the open gene–variant–drug–pathway cancer knowledge graph. See
[`PLAN.md`](./PLAN.md) for context, the **binding cancer guardrails + license gate**, the schema
decisions (Biolink Model + GA4GH VRS), and the roadmap (M0–M4).

## How these tasks map to Elyos

Each task below becomes an Elyos **Task JSON** validated against `packages/schema/src/schemas.ts`.
Field mapping:

- **id** — stable `oncogene-kg-<area>-NNN` (the table ID).
- **title** — the table Title.
- **project** — `oncogene-knowledge-graph`.
- **type** — `code | research | writing | data | design-spec | maintenance` (table "Type").
- **lane** — `donated` for all tasks here. Any future metered ETL run would be `funded` and must add
  `fundedBudgetUsd` (per schema's `if/then`).
- **priority** — `high | medium | low`.
- **domain** — e.g. `["cancer-research","bioinformatics","open-data","knowledge-graph"]`.
- **riskTier** — `low | medium | high`. License/extraction/explorer tasks are **medium** (license +
  evidence-fidelity + clinical-adjacency); schema/CI/docs scaffolding is **low**. No task here is
  `high` — anything patient-facing is out of scope and routes to a separate high-risk project.
- **urgent** — boolean (default `false`).
- **deliverable** — `pr | dataset | document | translation` (table "Deliverable").
- **tokenEstimate** — `small | medium | large` (table "Size").
- **status** — `open | in-progress | review | delivered | done` (start `open`).
- **context / objective / acceptanceCriteria[] / resources[] / output** — per task.
- **requestor** — `jdev1977` / beneficiary class until a named partner is secured.
- **verifiedNeed** — **`false`** while no committed steward/partner is secured (honest: the *gap* is
  real but the last-mile beneficiary is TO BE SECURED).
- **outputLicense** — `MIT` (code), `CC0-1.0` (CC0-core data/docs), `CC-BY-4.0` (CC-BY overlays/docs),
  or `CC-BY-SA-3.0` (ChEMBL-derived overlay only).

> **Standing guardrail on every data/extraction task:** no source may be touched until its
> `sources/allowlist.yml` entry is `approved` by the License/Compliance reviewer. Any task proposing
> to ingest or redistribute **COSMIC, OncoKB, or DrugBank-NC** (or any non-redistributable / controlled-
> access / identifiable patient data) is **refused and flagged** — out of scope, full stop. No medical
> advice is ever produced. Provenance is mandatory on every assertion.

---

## Milestone M0 — Foundation, licensing spine & schema

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| oncogene-kg-schema-001 | Define KG schema on Biolink Model + GA4GH VRS for variants | design-spec | small | low | document | — | Maintainer + scientific reviewer |
| oncogene-kg-license-001 | Source allow-list schema + license-gate policy (cancer guardrails) | design-spec | small | medium | document | — | License reviewer |
| oncogene-kg-license-002 | Analyze & record first sources (CIViC/Reactome/Open Targets; flag COSMIC/OncoKB/DrugBank) | research | medium | medium | document | oncogene-kg-license-001 | License reviewer |
| oncogene-kg-prov-001 | Provenance model + define the countable "assertion" unit | design-spec | small | low | document | oncogene-kg-schema-001 | Maintainer |
| oncogene-kg-id-001 | Identifier/CURIE normalization policy (Bioregistry: HGNC/Ensembl/VRS/Mondo/ChEMBL) | design-spec | small | low | document | oncogene-kg-schema-001 | Maintainer |
| oncogene-kg-firewall-001 | "Research use / not medical advice" scope-firewall policy | design-spec | small | medium | document | — | Maintainer |
| oncogene-kg-ci-001 | CI scaffold: Biolink/SHACL + provenance-completeness + allow-list linter | code | medium | low | pr | oncogene-kg-schema-001, oncogene-kg-prov-001 | Maintainer |
| oncogene-kg-partner-001 | Steward + CIViC-community outreach; reviewer recruitment | research | small | low | document | — | Maintainer |

**Acceptance criteria (key M0 tasks)**

- **oncogene-kg-schema-001**
  - Core Biolink categories defined (`Gene`, variant/`MolecularProfile`, `Drug`/`ChemicalEntity`,
    `Pathway`, `Disease`) with the predicates linking them (e.g. `affects_response_to`,
    `participates_in`, `gene_associated_with_condition`), datatypes, and cardinality.
  - GA4GH VRS adopted as the canonical variant identity; HGVS + source-native IDs retained as aliases.
  - Each entity reserves a provenance slot (source + evidence level + license) per the provenance model.
  - Conflicting-assertion representation specified (no single forced "truth").
  - Published as a human-readable spec plus a machine artifact (Biolink/SHACL stub).
- **oncogene-kg-license-001**
  - `sources/allowlist.yml` schema captures: source, custodian, version, URL, format, license basis,
    **redistribution + derivatives terms**, attribution/share-alike obligation, and
    `status: approved|rejected|pending`.
  - Policy text states **COSMIC, OncoKB, DrugBank-NC** and any non-redistributable / controlled-access /
    identifiable source are categorically `rejected`; CC-BY-SA/CC-BY sources go to segregated overlays.
  - Per-version re-verification requirement documented (licenses change between releases).
- **oncogene-kg-license-002**
  - ≥3 sources analyzed with a recorded license/redistribution determination; **CIViC marked
    `approved`** (CC0); Reactome + Open Targets analyzed (OT-own vs integrated-source distinction noted).
  - COSMIC/OncoKB/DrugBank recorded as `rejected` with the specific reason (non-commercial/proprietary/NC).
- **oncogene-kg-prov-001**
  - SEPIO-/nanopublication-style per-edge record defined with named fields incl. pipeline commit hash.
  - The countable **"assertion" unit** (one Biolink association instance) is defined so the
    100%-provenance CI gate is mechanically checkable.
- **oncogene-kg-ci-001**
  - CI **fails** on any assertion lacking resolvable provenance (source + evidence level + license).
  - CI runs Biolink/SHACL validation and rejects data referencing a source not `approved` in the allow-list.

**M0 Definition of Done:** schema v0 (Biolink + VRS) published; allow-list schema + license-gate policy
merged with ≥3 sources analyzed and ≥1 (CIViC) `approved`; provenance model ratified **with the
countable assertion unit defined**; CURIE policy + scope-firewall policy merged; CI provenance +
SHACL + allow-list gates live; **License/Compliance reviewer named (hard exit — if the seat is empty
M0 cannot exit; escalate per PLAN.md fallback)**; steward + CIViC-community outreach logged.
`pnpm build && pnpm test && pnpm lint` green.

---

## Milestone M1 — First sourced slice (CIViC, CC0, end-to-end)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| oncogene-kg-extract-civic-001 | CIViC ETL → Biolink edges with full provenance + evidence verbatim | code | medium | medium | pr | oncogene-kg-ci-001, oncogene-kg-license-002 | Maintainer |
| oncogene-kg-data-civic-001 | Produce first CIViC-derived graph slice (gene–variant–drug–disease) | data | large | medium | dataset | oncogene-kg-extract-civic-001 | Scientific reviewer |
| oncogene-kg-qa-001 | Independent stratified accuracy + provenance audit of CIViC slice (≥98%) | research | small | medium | document | oncogene-kg-data-civic-001 | Scientific reviewer |
| oncogene-kg-partner-002 | Engage ≥1 candidate steward for adoption/validation | research | small | low | document | oncogene-kg-partner-001 | Maintainer |

**Acceptance criteria (key M1 tasks)**

- **oncogene-kg-extract-civic-001**
  - Transforms a **pinned, checksummed** CIViC release to Biolink associations; CIVic evidence
    levels (A–E) and AMP/ASCO/CAP tiers are carried **verbatim** — never re-graded.
  - Variants mapped to GA4GH VRS identifiers (source-native IDs retained as aliases).
  - Every edge gets a complete provenance record (source, record ID, license=CC0, version, PMID,
    method, pipeline commit). Flags-on-doubt; never invents an edge.
- **oncogene-kg-data-civic-001**
  - First slice published as KGX + JSON-LD; **100%** of assertions carry resolvable provenance.
  - Passes CI (Biolink/SHACL + provenance + allow-list) before review.
  - Source-conflicting statements retained with separate provenance (not flattened).
- **oncogene-kg-qa-001**
  - A **stratified** sample (≥200 assertions or whole release; strata by edge type + evidence level)
    is checked against the cited CIViC record; **≥98% verify**.
  - The **auditor is independent of the extractor** (no self-grading).
  - Confirms no evidence re-grading occurred; mis-citations block sign-off until fixed.

**M1 Definition of Done:** end-to-end pipeline proven on CIViC (CC0); 100% provenance; independent
stratified audit ≥98%; exports validate; **Scientific reviewer named and signed off**; ≥1 candidate
steward in active conversation.

---

## Milestone M2 — Multi-source integration & normalization

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| oncogene-kg-id-002 | Implement identifier-normalization service (genes/variants/drugs/diseases/pathways) | code | large | medium | pr | oncogene-kg-id-001, oncogene-kg-data-civic-001 | Maintainer |
| oncogene-kg-extract-reactome-001 | Reactome (CC0) pathway + gene/protein-participation ETL | code | medium | low | pr | oncogene-kg-id-002, oncogene-kg-license-002 | Maintainer |
| oncogene-kg-extract-ot-001 | Open Targets ETL (OT-own CC0 data; per-integrated-source license segregation) | code | large | medium | pr | oncogene-kg-id-002, oncogene-kg-license-002 | License + scientific reviewer |
| oncogene-kg-data-clinvar-001 | ClinVar (US public-domain) variant enrichment under VRS | data | medium | medium | dataset | oncogene-kg-id-002 | Scientific reviewer |

**Acceptance criteria (key M2 tasks)**

- **oncogene-kg-id-002**
  - Resolves entities to canonical CURIEs (HGNC/Ensembl genes, VRS+HGVS variants, ChEMBL/RxNorm drugs,
    Mondo/DOID diseases, Reactome pathways) via Bioregistry; **≥99%** resolve to a live record (sampled).
  - **≥95%** of variant nodes carry a canonical GA4GH VRS identifier; merges are precision-tuned and
    human-reviewed (no silent auto-merge); source-native IDs kept as aliases.
- **oncogene-kg-extract-ot-001**
  - Ingests **only Open Targets' own CC0-computed evidence/associations**; any integrated third-party
    source-level record is taken **only after** that upstream source passes the license gate.
  - CC-BY / CC-BY-SA-derived fields land in **segregated overlays** with their own license manifest —
    the CC0 core is never contaminated.
- **oncogene-kg-data-clinvar-001**
  - ClinVar variant–condition assertions reconcile to existing VRS variant nodes where they match.
  - 100% provenance (license = US public domain); evidence/review status carried verbatim.

**M2 Definition of Done:** normalization service live (≥99% ID resolution, ≥95% VRS-keyed variants);
Reactome (CC0) + Open Targets (OT-own CC0) + ClinVar (PD) integrated; CC-BY/CC-BY-SA overlays cleanly
segregated; 100% provenance across all sources; cross-source contradiction report produced.

---

## Milestone M3 — Validation, explorer & exports

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| oncogene-kg-validate-001 | Full Biolink/SHACL validation + cross-source contradiction report | code | medium | medium | pr | oncogene-kg-extract-reactome-001, oncogene-kg-extract-ot-001, oncogene-kg-data-clinvar-001 | Scientific reviewer |
| oncogene-kg-export-001 | KGX + RDF/Turtle + JSON-LD + Neo4j exports w/ per-export license manifests | code | medium | low | pr | oncogene-kg-validate-001 | Maintainer |
| oncogene-kg-explorer-001 | Read-only public explorer (source + evidence on every edge; "not medical advice" banner) | code | medium | medium | pr | oncogene-kg-export-001 | Maintainer + scientific reviewer |
| oncogene-kg-docs-001 | Data dictionary + citation/usage guide for consumers | writing | small | low | document | oncogene-kg-explorer-001 | Maintainer |

**Acceptance criteria (key M3 tasks)**

- **oncogene-kg-validate-001**
  - Whole graph passes Biolink + SHACL validation; identifier-resolution and provenance-completeness
    gates green; a cross-source **contradiction report** lists conflicting assertions with their
    sources (kept, not silently resolved).
- **oncogene-kg-export-001**
  - Produces valid KGX (TSV+JSON), RDF/Turtle, JSON-LD, and a Neo4j dump with stable IRIs/CURIEs.
  - Each export ships a **license manifest** (`LICENSES.md` / `ATTRIBUTION.md`) naming every source,
    version, and required citation; CC0 core vs CC-BY/CC-BY-SA overlays are separately packaged.
  - Exports round-trip (re-import validates against the schema).
- **oncogene-kg-explorer-001**
  - Browse a gene/variant and see linked drugs, pathways, diseases, **and every assertion's source +
    evidence level + "as-of" source version**.
  - Static / no-account / collects no visitor PII; standing **"For research and education only — not
    medical advice"** banner on every view; links to raw exports.

**M3 Definition of Done:** full validation + contradiction report green; all four export formats with
correct license manifests; read-only explorer live with mandatory disclaimer and per-edge provenance;
data dictionary + citation guide published; reuse metrics tracked.

---

## Milestone M4 — Scale & partner adoption (shipped)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| oncogene-kg-data-scale-001 | Scale to ≥4 approved sources + ≥25,000 provenanced edges; refresh pipeline | data | large | medium | dataset | oncogene-kg-validate-001 | Scientific + license reviewers |
| oncogene-kg-partner-003 | Secure steward adoption + ≥1 documented downstream use | research | medium | low | document | oncogene-kg-partner-002 | Maintainer |
| oncogene-kg-license-003 | Conclude & document COSMIC/OncoKB/DrugBank engagement outcome | research | small | medium | document | oncogene-kg-license-002 | License reviewer |
| oncogene-kg-sustain-001 | Sustainability: refresh cadence + reviewer rotation + hosting plan | writing | small | low | document | oncogene-kg-partner-003 | Maintainer |

**Acceptance criteria (key M4 tasks)**

- **oncogene-kg-data-scale-001**
  - ≥4 source collections integrated (each license-gated before extraction); **≥25,000** sourced
    edges published with 100% provenance; a fresh stratified accuracy audit still **≥98%**.
  - A re-runnable refresh re-applies the license gate and re-stamps source versions ("as-of" labels).
- **oncogene-kg-partner-003**
  - A **named** academic/advocacy/community steward commits to adopt/host/cite the graph.
  - **≥1 concrete downstream use** (a tool, paper, or integration) is documented.

**M4 Definition of Done (project "shipped"):** ≥4 approved sources; ≥25,000 provenanced edges; 100%
provenance; ≥99% ID resolution; ≥95% variants VRS-keyed; fresh audit ≥98%; **a named steward has
adopted/cited the graph with ≥1 documented downstream use**; COSMIC/OncoKB/DrugBank engagement
outcome documented (incl. "declined / not pursued"); sustainability + refresh + reviewer-rotation
plan in effect.

---

## Backlog / future (sized, unscheduled)

| ID | Title | Type | Size | Risk | Deliverable | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| oncogene-kg-overlay-mondo-001 | Mondo disease-ontology CC-BY overlay (xrefs + labels) | data | small | low | dataset | CC-BY-4.0 overlay; segregated from CC0 core |
| oncogene-kg-overlay-chembl-001 | ChEMBL drug-normalization CC-BY-SA overlay | data | medium | medium | dataset | CC-BY-SA-3.0 overlay; share-alike isolated |
| oncogene-kg-vicc-001 | Reconcile against VICC meta-knowledgebase representation | data | medium | medium | dataset | Interop with VICC; map evidence models |
| oncogene-kg-sparql-001 | Optional hosted SPARQL / Neo4j query endpoint | code | large | low | pr | Only if a steward offers hosting |
| oncogene-kg-refresh-001 | Automated scheduled refresh + diff review + re-license-gate | maintenance | medium | medium | pr | Combats stale-evidence risk |
| oncogene-kg-metrics-001 | Reuse-metrics tracking (downloads/queries, no PII) | maintenance | small | low | document | Outcome tracking |
| oncogene-kg-quality-001 | Automated anomaly/outlier flagging for reviewer | code | medium | low | pr | Assistive QA, human-confirmed |

---

## Example task JSON

Schema-valid Task JSON for the first M0 task (`oncogene-kg-schema-001`), validated field-by-field
against `packages/schema/src/schemas.ts`:

```json
{
  "id": "oncogene-kg-schema-001",
  "title": "Define KG schema on Biolink Model + GA4GH VRS for variants",
  "project": "oncogene-knowledge-graph",
  "type": "design-spec",
  "lane": "donated",
  "priority": "high",
  "domain": ["cancer-research", "bioinformatics", "open-data", "knowledge-graph", "precision-oncology"],
  "riskTier": "low",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "small",
  "status": "open",
  "context": "An open gene-variant-drug-pathway cancer knowledge graph needs a shared schema before any data is ingested. Sources are open-access, openly-licensed knowledgebases only (CIViC CC0, Reactome CC0, Open Targets own CC0 data, ClinVar public domain) plus open ontologies for normalization. COSMIC, OncoKB, and DrugBank-NC are non-redistributable and OUT OF SCOPE; controlled-access (dbGaP/EGA/biobanks) and any identifiable patient data are forbidden. This is a research/education artifact, never medical advice. See PLAN.md (Data, licensing & compliance leads).",
  "objective": "Define the KG schema using the Biolink Model for entity categories (Gene, variant/MolecularProfile, Drug/ChemicalEntity, Pathway, Disease) and predicates linking them, adopting GA4GH VRS as the canonical variant identity, with a per-edge provenance slot and explicit modeling of source-conflicting assertions (no single forced 'truth').",
  "acceptanceCriteria": [
    "Core Biolink categories and the predicates linking gene/variant/drug/pathway/disease defined with datatypes and cardinality.",
    "GA4GH VRS adopted as canonical variant identity; HGVS and source-native IDs retained as aliases.",
    "Every entity/edge reserves a provenance slot (source IRI + evidence level + license) per the provenance model.",
    "Conflicting-assertion representation specified so contradicting sources are retained, not flattened.",
    "Evidence levels are carried verbatim from sources; the schema does not permit re-grading.",
    "Published as a human-readable spec plus a machine artifact (Biolink/SHACL stub)."
  ],
  "resources": [
    "planning/projects/oncogene-knowledge-graph/PLAN.md",
    "https://biolink.github.io/biolink-model/",
    "https://vrs.ga4gh.org/",
    "https://civicdb.org/",
    "https://reactome.org/"
  ],
  "output": "A schema specification document (schema/README.md) plus a Biolink/SHACL stub defining the entity categories, predicates, variant (VRS) identity model, and provenance slots, committed via PR.",
  "requestor": "jdev1977",
  "verifiedNeed": false,
  "outputLicense": "CC0-1.0"
}
```
