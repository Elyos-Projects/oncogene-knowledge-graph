# PLAN — oncogene-knowledge-graph

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated · Risk tier: medium

An open, linked, citable knowledge graph connecting **genes → variants → drugs → pathways → cancer
types**, assembled **only** from open-access, aggregate sources (CIViC, Reactome, Open Targets, and
other CC0/CC-BY public knowledgebases), where **every assertion carries machine-readable provenance**
back to a specific source record, evidence level, and (where applicable) publication. It is a
**research and education artifact, not a clinical decision tool**, and it never ingests identifiable
or controlled-access patient data.

## Executive summary

Cancer is a disease of the genome. A clinician or researcher trying to understand a single tumor
alteration — say *BRAF* V600E in melanoma — must today stitch together facts from a dozen
disconnected resources: the variant's clinical interpretation (CIViC), the targeted drugs and their
evidence (Open Targets, CIViC, ChEMBL), the biological pathway the gene sits in (Reactome), the
disease ontology term (Mondo), and the primary literature (PubMed). These resources use different
identifiers, different variant nomenclature, different disease vocabularies, and different evidence
frameworks. The connective tissue — *which variant, in which cancer, confers sensitivity or
resistance to which drug, acting through which pathway, on what strength of evidence* — exists only in
researchers' heads and in proprietary, paywalled databases.

This project builds that connective tissue as an **open knowledge graph (KG)**. It harmonizes
open-access oncology knowledgebases into one queryable graph using the **Biolink Model** as the
schema and the **GA4GH Variation Representation Specification (VRS)** for variant identity, so the
graph is interoperable with the wider open biomedical linked-data ecosystem (Monarch Initiative,
NCATS Biomedical Data Translator, the VICC meta-knowledgebase) from day one. Standard exports (KGX,
RDF/Turtle, JSON-LD, a Neo4j dump) make it usable by bioinformaticians and machine-consumable by
downstream open projects; a lightweight read-only explorer makes it browsable by researchers,
students, and scientifically literate patient advocates.

**The two constraints that dominate this plan are licensing/compliance and clinical safety.** On
licensing: the proprietary leaders in this space — **COSMIC** and **OncoKB** — carry non-commercial
or custom licenses that forbid the open redistribution this project requires; **they are out of scope
for redistribution and are flagged, not ingested** (see [Data, licensing & compliance](#data-licensing--compliance)).
The graph is seeded exclusively from sources whose licenses permit open reuse and derivatives —
**CIViC (CC0)**, **Reactome (CC0)**, **Open Targets' own computed evidence (CC0)**, **ClinVar (US
public domain)**, and openly-licensed ontologies — with CC-BY / CC-BY-SA enrichments held in clearly
segregated, separately-licensed layers to prevent license contamination of the CC0 core. On safety:
this is a **medium-risk research artifact**; it presents *sourced assertions with their original
evidence levels*, never clinical recommendations. Any patient-facing interpretation layer would be a
**high-risk** undertaking requiring oncologist + patient-advocate sign-off and is explicitly **out of
scope** for this project — a firewall we hold deliberately.

Risk tier: **medium** — driven primarily by (1) per-source license/redistribution compliance and (2)
scientific-accuracy and evidence-fidelity review, with a hard scope boundary against medical advice.

## Problem & beneficiaries

**The problem.** Actionable cancer-genomics knowledge is real but fragmented and partly enclosed.
The richest curated interpretation databases (COSMIC, OncoKB) are access-controlled or
non-commercially licensed. The open resources that *do* exist (CIViC, Reactome, Open Targets,
ClinVar) are excellent but siloed: they don't share identifiers, variant representations, disease
vocabularies, or evidence models, so connecting "this variant ↔ this drug ↔ this pathway ↔ this
cancer, at this evidence level, with this citation" requires bespoke, repeated, error-prone
integration work that every lab redoes. There is **no openly-licensed, provenance-complete, queryable
graph** that unifies the open oncology knowledgebases under shared standards.

**Beneficiaries.**
- **Cancer researchers and computational biologists** who need a queryable, citable, source-linked
  starting point instead of re-integrating the same sources by hand.
- **Bioinformatics tool builders & downstream open projects** (Monarch, NCATS Translator, VICC,
  pharmacogenomics pipelines) that can reconcile against and reuse a standards-based open graph.
- **Molecular tumor board support staff and clinical-genomics curators** — as a *research reference
  that surfaces sourced evidence with its level*, never as a decision system.
- **Students and trainees** in cancer genomics / bioinformatics learning how variant–drug–pathway
  evidence is structured and cited.
- **Scientifically literate patient advocates** who want to trace the evidence behind a claim to its
  source — explicitly *for understanding the science*, with a standing "not medical advice" frame.

**Verified need.** The *gap* (no open, standards-based, provenance-complete oncology KG unifying these
sources) is real and demonstrable, and the open sources to fill it exist and are appropriately
licensed. However, a **named partner organization or steward who will adopt, host, cite, and help
validate the output is TO BE SECURED.** `verifiedNeed` is therefore **`false`** in the task backlog
until a committed last-mile beneficiary signs on. Per the Elyos quality bar ("delivered, not
merged"), securing this partner and recruiting the required scientific reviewer are **first-class
M0 objectives and preconditions for declaring the project *shipped*.**

**Partner org.** TO BE SECURED. Candidate stewards: an academic cancer-genomics or
bioinformatics center, a patient-advocacy research foundation, the VICC / GA4GH community, the
Monarch Initiative, or a university molecular-tumor-board informatics group. The **CIViC community
(WashU)** is the natural first collaborator since CIViC is the CC0 seed source and shares this
project's open-curation ethos; engagement is pursued in parallel from M0.

## Goals and non-goals

**Goals.**
- Produce an open, standards-based KG linking genes, variants, drug/therapy associations, pathways,
  and cancer types, with **100% per-assertion provenance** (source record + evidence level + citation
  + license + retrieval date + extraction method).
- Use **Biolink Model** as the schema and **GA4GH VRS** for canonical, computable variant identity, so
  the same variant from different sources reconciles to one node.
- Normalize all entities to stable **CURIEs** via a registry (Bioregistry / identifiers.org):
  genes (HGNC/Ensembl), variants (VRS + HGVS), drugs (ChEMBL/RxNorm), diseases (Mondo/DOID/NCIt),
  pathways (Reactome).
- Preserve and expose each source's **original evidence level / assertion verbatim** (CIViC A–E,
  AMP/ASCO/CAP tiers, etc.) — never re-grade or editorialize evidence.
- Ship standard machine exports (KGX, RDF/Turtle, JSON-LD, Neo4j) + a read-only explorer + a clear
  data dictionary and citation guide.
- Keep a hard, auditable boundary: open-access/aggregate data only; no medical advice.

**Non-goals (this project will NOT):**
- Provide clinical decision support, treatment recommendations, or any patient-specific
  interpretation. (That would be high-risk and is out of scope; see Scope.)
- Ingest, redistribute, or scrape **COSMIC, OncoKB, DrugBank's non-commercial data**, or any source
  whose license forbids open redistribution/derivatives. (Cross-referencing public stable IDs is
  evaluated separately and conservatively; *no bulk content* is taken.)
- Touch any **controlled-access** resource (dbGaP, EGA, individual-level biobanks) or any
  **identifiable patient data**, ever.
- Perform original variant calling, primary sequencing analysis, or de-novo clinical curation.
- Re-grade, "harmonize away," or assert a single "truth" where sources legitimately disagree —
  conflicts are retained with separate provenance.

## Success metrics (outcomes)

Outcome-centric (beneficiary value), not vanity counts. Baselines are 0 (greenfield) unless noted.

| Metric | Baseline | Target (at "shipped") |
|---|---|---|
| **Provenance completeness** — assertions with a resolvable source + evidence level + license | n/a | **100%** (CI-enforced; non-negotiable gate) |
| **Identifier resolution** — entity CURIEs that resolve to a live, correct registry record | n/a | **≥99%** (sampled audit) |
| **Accuracy audit** — sampled assertions verified against the cited source by an independent reviewer | n/a | **≥98%** (medium-risk bar; higher than the 95% used for low-stakes history) |
| **Source coverage** — open knowledgebases integrated through the license gate | 0 | **≥4** (CIViC, Reactome, Open Targets, ClinVar) |
| **Graph scale** — sourced variant↔(drug\|pathway\|disease) edges published | 0 | **≥25,000** edges, each fully provenanced |
| **Variant reconciliation** — variants assigned a canonical GA4GH VRS identifier | 0 | **≥95%** of variant nodes |
| **Downstream adoption** — documented external reuse (a tool, paper, or steward integration) | 0 | **≥1** concrete, named, documented use |
| **Reviewer coverage** — high-impact (drug-association) edges seen by the scientific reviewer | 0 | **100%** of a stratified review sample |

*Note:* scale and coverage are *enabling* metrics; the headline outcome is **a named beneficiary
actually using a trustworthy, fully-sourced graph** (the adoption metric). A large graph nobody
adopts is not "delivered."

## Scope

**In scope.**
- ETL from open-access, openly-licensed oncology knowledgebases into a Biolink/VRS-modeled graph.
- Entity & variant normalization to stable CURIEs / VRS IDs.
- Per-assertion provenance, evidence-level preservation, and contradiction tracking.
- Standard exports, a read-only explorer (with a standing "research use / not medical advice"
  banner), data dictionary, and citation guidance.
- License-gating infrastructure and a per-source rights ledger.

**Out of scope (explicit).**
- **Any clinical or patient-facing interpretation, recommendation, triage, or advice.** A
  patient-facing layer would be a *separate, high-risk project* gated behind oncologist +
  patient-advocate sign-off and "not medical advice" framing.
- **COSMIC, OncoKB, DrugBank-NC** and any non-redistributable source content.
- **Controlled-access / individual-level / identifiable patient data** (dbGaP, EGA, biobanks, raw
  TCGA controlled tier). Only open-tier, aggregate, de-identified data is ever considered.
- Original sequencing, variant calling, or de-novo clinical curation.
- Hosting a production clinical service or any system that ingests user-submitted patient data.

## Solution approach & architecture

A deterministic, reproducible **ETL → harmonize → validate → export** pipeline, governed by a
license gate at the front and a provenance/validation gate at the back. Content-only; no live
patient data flows through any component.

**Pipeline (per source):**
1. **License gate** — a source may not be touched until its `sources/allowlist.yml` entry is
   `approved` by the License/Compliance reviewer (license basis, redistribution + derivatives terms,
   attribution/share-alike obligations, version pinned).
2. **Acquire** — pull a *pinned, versioned* release (snapshot date + checksum recorded). Prefer bulk
   open releases over live APIs for reproducibility.
3. **Extract & map** — transform source records to **Biolink Model** entities/associations; map
   variants to **GA4GH VRS** computed identifiers; resolve entity IDs to canonical CURIEs.
4. **Provenance stamp** — attach to every edge: source + source-record ID, source license, snapshot
   version/date, original evidence level/assertion text, supporting publication (PMID/DOI),
   extraction method, and the deriving pipeline commit hash (a nanopublication-/SEPIO-style record).
5. **Validate** — Biolink + SHACL shape validation, identifier-resolution check, provenance
   completeness, allow-list conformance, and a cross-source contradiction report.
6. **Merge & export** — assemble the harmonized graph; emit KGX (TSV+JSON), RDF/Turtle, JSON-LD, and
   a Neo4j dump, each accompanied by a per-export **license manifest** (see compliance section).

**Tech stack.** TypeScript + ESM, pnpm workspaces (per Elyos conventions) for pipeline orchestration,
CLI, validators, and the explorer; Python is permitted *only* inside isolated ETL adapters where a
canonical library is mandatory (e.g., the reference VRS implementation) and is wrapped behind a
typed interface — the agent-neutral core stays TS. Storage/interchange is **file-based and
git-versioned** (KGX / RDF / JSON-LD artifacts), with Neo4j as an *optional* downstream load target,
not a source of truth. The explorer is a static, no-account, no-PII reader over the exported graph.

**Schema & standards (key decisions).**
- **Schema:** Biolink Model (categories: `Gene`, `SequenceVariant`/`MolecularProfile`, `Drug`/
  `ChemicalEntity`, `Pathway`, `Disease`; predicates: `affects_response_to`, `participates_in`,
  `gene_associated_with_condition`, etc.). Chosen for ecosystem interoperability (Monarch,
  Translator) over a bespoke ontology.
- **Variant identity:** GA4GH **VRS** computed identifiers as the canonical variant key, with HGVS and
  source-native IDs retained as aliases. This is what lets the *same* variant from CIViC and ClinVar
  collapse to one node.
- **Evidence/provenance:** SEPIO-aligned, nanopublication-style per-edge provenance; the source's
  evidence framework is **carried verbatim**, never re-scored.
- **Identifiers:** Bioregistry-backed CURIEs; HGNC + Ensembl (genes), Mondo (+ DOID/NCIt xrefs)
  (diseases), ChEMBL (+ RxNorm) (drugs), Reactome (pathways).

**Key architectural decision — license-layer segregation.** The **CC0 core graph** is built only
from CC0 / US-public-domain sources (CIViC, Reactome, Open Targets' own computed data, ClinVar). Any
**CC-BY** (e.g., Mondo, GO) or **CC-BY-SA** (e.g., ChEMBL) enrichment lives in *separately packaged,
separately licensed overlay layers* with their own manifests, so the core stays cleanly CC0 and no
share-alike obligation contaminates it. Consumers can take the CC0 core alone or opt into overlays.

## Data, licensing & compliance

> **THIS SECTION IS BINDING AND LEADS THE PLAN. The cancer-domain guardrails below override any
> convenience, deadline, or scope ambition. When in doubt, stop and flag.**

**Cancer-domain guardrails (non-negotiable):**
1. **Open-access / aggregate / de-identified data ONLY.** Controlled-access resources (**dbGaP, EGA,
   individual-level biobanks**) and **any identifiable patient data** are categorically **out of
   scope** — they require authorized access + IRB oversight this project does not have and will not
   seek. TCGA/GDC: **open tier only**; the controlled tier is forbidden. GEO: open records only.
2. **Per-source license verification before any access.** No source is read, fetched, parsed, or
   redistributed until its allow-list entry is `approved`. **COSMIC** (non-commercial/custom) and
   **OncoKB** (proprietary/custom, API-licensed) **fail the gate for redistribution → flagged, not
   ingested.** **DrugBank** open data is **CC-BY-NC** (non-commercial) → the NC clause is
   incompatible with Elyos's open-reuse requirement → flagged/excluded from redistribution.
3. **No medical advice.** This project ships a *research/education* graph. It presents sourced
   assertions with their original evidence levels and never generates recommendations, triage, or
   patient-specific interpretation. Every human-facing surface carries a standing **"For research and
   education only — not medical advice"** notice. A patient-facing interpretation layer is a
   *separate high-risk project* requiring oncologist + patient-advocate sign-off; it is **out of
   scope here**.
4. **Provenance on every assertion.** An assertion without resolvable provenance (source + evidence
   level + license) **does not ship** — CI fails the build. No exceptions.

**Source rights ledger (initial determination — to be confirmed per-version by the License reviewer;
licenses change, so each release re-verifies):**

| Source | Use | License (as understood) | Gate decision |
|---|---|---|---|
| **CIViC** | Variant clinical interpretations, drug/disease evidence | **CC0 1.0** (public domain) | **Approve — CC0 core seed** |
| **Reactome** | Pathways, gene/protein participation | **CC0 1.0** (verify current version) | **Approve — CC0 core** |
| **Open Targets** | Computed target–disease & drug evidence/associations | **CC0 1.0** for OT's *own* data; integrates 3rd-party sources w/ their own terms | **Approve OT-own data; per-integrated-source check before taking source-level records** |
| **ClinVar** | Variant–condition assertions | **US Government public domain** (NCBI) | **Approve — CC0-equivalent core** |
| **Mondo / DOID / NCIt** | Disease ontology terms & xrefs | Mondo **CC-BY 4.0**; DOID **CC0**; NCIt open | **Approve — DOID in core; Mondo as CC-BY overlay** |
| **HGNC / Ensembl** | Gene identifiers/nomenclature | Open, no-restriction (verify EBI/EMBL terms) | **Approve — identifiers only** |
| **ChEMBL** | Drug/compound normalization | **CC-BY-SA 3.0** (attribution + share-alike) | **Approve as CC-BY-SA overlay only — segregated from CC0 core** |
| **GA4GH VRS** | Variant representation standard | Open spec | **Approve — standard, not data** |
| **COSMIC** | Mutation catalogue | **Non-commercial / custom license** | **FLAG — out of scope for redistribution; not ingested** |
| **OncoKB** | Precision-oncology annotations | **Proprietary / custom, API-licensed** | **FLAG — out of scope for redistribution; not ingested** |
| **DrugBank** | Drug data | **CC-BY-NC 4.0** (non-commercial) | **FLAG — NC incompatible; excluded from redistribution** |

**Provenance model.** Each edge is a SEPIO-/nanopublication-style record: `{assertion, source,
source_record_id, source_license, source_version, retrieval_date, original_evidence_level,
publication (PMID/DOI), extraction_method, pipeline_commit}`. The **countable "assertion" unit** is
defined precisely (one Biolink association instance) so the 100%-provenance CI gate is mechanically
checkable.

**Attribution & output licensing.** **Code: MIT.** **CC0 core data: CC0 1.0.** **CC-BY overlays:
CC-BY-4.0; ChEMBL-derived overlay: CC-BY-SA-3.0** — each shipped with a `LICENSES.md` /
`ATTRIBUTION.md` manifest naming every source, its version, and its required citation. CIViC,
Reactome, Open Targets, and ClinVar are credited per their requested citation even where CC0 does not
legally compel it (good-citizen norm).

**Privacy / PII stance.** The graph contains **no individual-level or identifiable data** — only
gene/variant/drug/pathway/disease aggregate knowledge. The explorer collects **no visitor PII**, has
no accounts, and runs no analytics that profile users. There is no path by which patient data enters
the system.

## Quality, review & risk gates

**Risk tier: medium.** Rationale: the deliverable is a research artifact (not patient-facing advice),
but it requires (a) strict license/redistribution compliance and (b) scientific-accuracy +
evidence-fidelity review. Any drift toward patient-facing interpretation escalates the affected work
to **high** and triggers oncologist + patient-advocate sign-off — at which point it leaves this
project's scope.

**Required reviews before a deed is "done":**
- **License/Compliance reviewer** — signs off every source's allow-list entry *before* extraction and
  re-verifies on every version bump. A hard gate; if the seat is empty, license/data tasks cannot
  proceed (escalation fallback documented in Governance).
- **Scientific/domain reviewer** (cancer-genomics-literate: bioinformatician, curator, or clinician
  scientist) — audits accuracy and evidence fidelity; 100% of a stratified sample of drug-association
  edges; confirms no re-grading of evidence occurred.
- **Maintainer** — code, schema, CI, and provenance-mechanism review.
- **Oncologist + patient-advocate** — **only if/when** any patient-facing surface is proposed; that
  proposal is rejected into a separate high-risk project rather than merged here.

**Definition of Shipped (project).** Open, Biolink/VRS-modeled KG built from **≥4 approved open
sources**; **100% per-assertion provenance** (CI-enforced); identifier resolution ≥99% and ≥95% of
variants VRS-keyed; independent accuracy audit **≥98%**; standard exports + read-only explorer (with
"not medical advice" banner) live; per-export license manifests correct; **a named steward has
adopted/cited the graph and ≥1 downstream reuse is documented**; License and Scientific reviewers
named and active; sustainability/refresh plan in effect.

## Roadmap & milestones

Phased, dependency-ordered. M0 is a thin cold-start that stands up the *gates* before any data moves.

- **M0 — Foundation, licensing spine & schema.**
  *Goal:* make it impossible to ingest data unsafely or unlicensed.
  *Exit criteria:* schema v0 (Biolink + VRS) published; allow-list schema + license-gate policy
  merged with ≥3 sources analyzed and ≥1 (CIViC) `approved`; provenance model ratified *with the
  countable assertion unit defined*; identifier/CURIE policy committed; "research-use / not-medical-
  advice" scope-firewall policy merged; CI gates (Biolink/SHACL + provenance + allow-list) live;
  License/Compliance reviewer **named** (hard exit); steward + CIViC-community outreach initiated.
  `pnpm build && pnpm test && pnpm lint` green.

- **M1 — First sourced slice (CIViC, CC0, end-to-end).**
  *Goal:* prove the whole pipeline on the cleanest source.
  *Exit criteria:* CIViC ETL → Biolink edges with **100% provenance**; first graph slice published;
  independent accuracy audit (stratified, ≥200 assertions or whole release) **≥98%**; exports
  validate; Scientific reviewer named and signed off; ≥1 candidate steward in active conversation.

- **M2 — Multi-source integration & normalization.**
  *Goal:* add pathways, computed associations, and variant enrichment under shared IDs.
  *Exit criteria:* identifier-normalization service live (genes/variants/drugs/diseases/pathways);
  Reactome (CC0) and Open Targets (OT-own CC0) integrated; ClinVar (PD) variant enrichment added;
  ≥95% of variants VRS-keyed; CC-BY/CC-BY-SA overlays cleanly segregated from the CC0 core; 100%
  provenance maintained across all sources; cross-source contradiction report produced.

- **M3 — Validation, explorer & exports.**
  *Goal:* make the graph usable and trustworthy by humans and machines.
  *Exit criteria:* full Biolink/SHACL validation + contradiction report green; KGX + RDF/Turtle +
  JSON-LD + Neo4j exports with correct per-export license manifests; read-only explorer live (every
  edge shows source + evidence level; standing "not medical advice" banner); data dictionary +
  citation/usage guide published; reuse metrics tracked.

- **M4 — Scale & partner adoption (shipped).**
  *Goal:* reach a useful scale and a real beneficiary.
  *Exit criteria:* ≥4 approved sources integrated and refreshable; **≥25,000 provenanced edges**;
  fresh audit still ≥98%; **a named steward has adopted/cited the graph with ≥1 documented downstream
  use**; COSMIC/OncoKB/DrugBank engagement outcome documented (incl. "declined / not pursued");
  sustainability + refresh-cadence + reviewer-rotation plan in effect. → **Definition of Shipped met.**

## Work breakdown

The itemized, schema-mapped backlog lives in [`TASKS.md`](./TASKS.md), organized by the M0–M4
milestones above. Each task maps to an Elyos **Task JSON** validated against
`packages/schema/src/schemas.ts`, carries a stable `oncogene-kg-<area>-NNN` ID, a reviewer, and (for
data/extraction tasks) the standing license-gate precondition. ~22 tasks span foundation → first
slice → multi-source → validation/explorer → scale/adoption, with a sized, unscheduled backlog for
later enrichment (additional sources, SPARQL endpoint, periodic-refresh automation).

## Governance, roles & stakeholders

- **Maintainer (Owner): TBD** (`jdev1977` bootstrapping) — schema, code, CI, releases, final
  integration sign-off; accountable for the scope firewall.
- **License/Compliance reviewer: TO BE SECURED (hard requirement).** Approves every source before
  access and on each version bump. *Fallback if unfilled:* no data/extraction task may start; M0
  cannot exit; escalate to the Elyos board for a named volunteer or paused project. License/ToS
  questions are never self-approved by the implementing agent.
- **Scientific / domain reviewer: TO BE SECURED (hard requirement for M1 exit).** Cancer-genomics-
  literate (bioinformatician, clinical-genomics curator, or clinician scientist). Audits accuracy +
  evidence fidelity.
- **Steward (last-mile owner): TO BE SECURED.** Adopts, hosts/cites, and helps validate the graph
  (academic center, advocacy-research foundation, VICC/Monarch/GA4GH community, or MTB informatics
  group).
- **Requestor / beneficiary class:** `jdev1977` on behalf of the cancer-research + open-bio-data
  community until a named partner is secured (`verifiedNeed=false` until then).
- **Oncologist + patient-advocate reviewers:** engaged **only** if a patient-facing surface is ever
  proposed — which routes to a separate high-risk project, not this one.
- **Expert reviewers / advisors (advisory):** CIViC community (WashU), GA4GH/VICC, Monarch
  Initiative — pursued as collaborators and validation partners from M0.

## Dependencies & integrations

- **Data sources:** CIViC, Reactome, Open Targets, ClinVar (core); Mondo/DOID/NCIt, HGNC/Ensembl,
  ChEMBL (normalization/overlay). COSMIC/OncoKB/DrugBank explicitly **excluded**.
- **Standards & vocabularies:** Biolink Model, GA4GH VRS, SEPIO/nanopublications (provenance),
  Bioregistry/identifiers.org (CURIEs), KGX (KG exchange), SHACL (validation), SSSOM (mappings).
- **Tooling:** TS/pnpm pipeline + CLI + validators; reference VRS implementation (isolated adapter);
  optional Neo4j load target; static explorer host.
- **Elyos pieces:** the Task schema (`packages/schema`), the donated-lane workflow (CLI prepares
  workspace + opens PRs; humans run their own agent), governance/review process, and the registry.
- **External communities:** CIViC/WashU, VICC, GA4GH, Monarch — for validation, reconciliation, and
  potential stewardship.

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|
| A source's license is misread; non-redistributable content (e.g., COSMIC/OncoKB) gets ingested | Medium | **Critical** | Hard allow-list gate; no access before `approved`; per-version re-verification; categorical exclusion list; CI rejects data from non-approved sources | License reviewer |
| License contamination — a CC-BY-SA source (ChEMBL) infects the CC0 core | Medium | High | Layer segregation (CC0 core vs CC-BY/CC-BY-SA overlays), separate packages + manifests, CI provenance carries per-source license | Maintainer |
| Project drifts toward patient-facing advice / clinical use | Medium | **Critical** | Scope-firewall policy; standing "not medical advice" banner; any patient-facing proposal rejected into a separate high-risk project (oncologist+advocate gated) | Maintainer |
| Misrepresenting/re-grading evidence levels → false confidence in a claim | Medium | High | Carry source evidence verbatim; never re-score; Scientific reviewer audits fidelity; contradiction report keeps disagreements visible | Scientific reviewer |
| Variant reconciliation errors merge distinct variants (VRS edge cases: indels, structural, normalization) | Medium | High | Use reference VRS implementation; precision-over-recall matching; human-reviewed merges; keep source-native IDs as aliases; never auto-merge silently | Maintainer |
| Identifier rot — CURIEs/source IDs change between versions | Medium | Medium | Pin + checksum versions; Bioregistry resolution checks in CI; scheduled refresh with diff review | Maintainer |
| No steward/partner secured → output unused ("merged, not delivered") | Medium | High | Outreach from M0 (CIViC community first); adoption is a Definition-of-Shipped gate, not optional | Maintainer |
| Required reviewer seats stay empty | Medium | High | Hard gates (M0/M1 cannot exit); documented escalation to Elyos board; pause rather than proceed unreviewed | Maintainer |
| Stale data presented as current (oncology evidence moves fast) | High | Medium | Snapshot date + source version on every edge and in the explorer banner; documented refresh cadence; "as-of" labeling | Maintainer |
| Donated-agent ETL introduces silent extraction errors at scale | Medium | High | Deterministic, reviewable transforms; flag-on-doubt (never invent); independent ≥98% audit; CI validation before review | Scientific reviewer |

## Security & privacy

- **Threat surface is small by design:** content-only pipeline over public data; no user accounts, no
  patient data, no PII ingestion, no write-back from the public explorer.
- **Secrets handling:** the few source APIs that need keys (read-only) use environment variables, are
  never committed, and never appear in logs, receipts, or provenance records (per CLAUDE.md). Prefer
  keyless bulk downloads where possible.
- **Supply-chain / integrity:** pinned source versions + checksums; reproducible builds; pipeline
  commit hash recorded in every assertion's provenance so any edge is traceable to the code that made
  it.
- **Abuse/misuse prevention:** the standing "research/education — not medical advice" frame and the
  scope firewall guard against the graph being repackaged as clinical guidance; evidence levels are
  always shown so weak/contested evidence cannot masquerade as established fact; the refusal
  guardrails (no surveillance, no for-profit primary benefit, no license/privacy violation) are
  applied to every task.
- **No controlled-access path:** there is deliberately no mechanism by which dbGaP/EGA/biobank or
  identifiable data could enter — the absence is the control.

## Sustainability & maintenance

- **Post-delivery ownership:** the steward (once secured) hosts/cites; the maintainer + reviewer
  rotation keeps the pipeline runnable. If no steward is secured, the graph remains a versioned,
  self-describing open artifact (exports + manifests) that any group can fork and refresh — the
  build is reproducible from pinned sources.
- **Refresh cadence:** sources update (CIViC continuously, Reactome/ClinVar/Open Targets on
  releases). The plan defines a scheduled, diff-reviewed refresh; every release re-runs the license
  gate (licenses can change) and re-stamps versions. Each edge is labeled "as-of <source version>".
- **Outcome tracking:** documented downstream reuses (tools, papers, integrations), export
  downloads/queries (no user PII), and the accuracy-audit trend over releases — reported against the
  success metrics above.
- **Bus-factor:** schema, provenance model, license ledger, and pipeline are all documented and
  git-versioned so a new maintainer can take over from the repo alone.

## Open questions

1. **Steward.** Who is the committed adopter/host/citer? (CIViC community is the lead candidate.)
2. **Reviewer recruitment.** Who fills the License/Compliance and Scientific reviewer seats, and what
   is the credential bar for the scientific reviewer?
3. **Reactome / Open Targets license version.** Confirm the *current* license of the exact release
   pulled (Reactome historically shifted licensing; Open Targets mixes upstream terms) — re-verify
   per release, do not assume.
4. **Cross-referencing excluded sources.** Is publishing a *stable public ID* that points to COSMIC/
   OncoKB (no content taken) acceptable, or excluded entirely? (Default: conservative — exclude
   unless the License reviewer explicitly approves ID-only cross-refs.)
5. **Overlay packaging.** Final boundary between the CC0 core and CC-BY / CC-BY-SA overlays — which
   enrichments justify the share-alike obligation?
6. **Neo4j hosting.** Is a hosted query endpoint in scope post-ship, or do we ship dumps only and let
   stewards host? (Default: dumps only until a steward offers hosting.)
7. **Funded lane.** If donated curation is too slow, is a budget-capped funded ETL run justified, and
   what is the per-task cap? (Default: donated-only unless the board approves.)

## References

- Elyos: `CLAUDE.md` (work rules, lanes, guardrails), `docs/good-deed-definition.md` (criteria +
  risk tiers), `packages/schema/src/schemas.ts` (Task schema), `planning/ROADMAP.md` (Track 8b —
  `oncogene-knowledge-graph`, medium risk).
- Companion: `planning/projects/revolutionary-patriots-kg/` (sibling open-KG plan; shared
  provenance/license-gate patterns).
- Sources: CIViC (civicdb.org, CC0), Reactome (reactome.org), Open Targets (platform.opentargets.org),
  ClinVar (ncbi.nlm.nih.gov/clinvar), Mondo/DOID, HGNC, Ensembl, ChEMBL.
- Standards: Biolink Model, GA4GH Variation Representation Specification (VRS), VICC
  meta-knowledgebase, KGX, SEPIO, nanopublications, Bioregistry/identifiers.org, SHACL, SSSOM.
- Excluded (flagged): COSMIC (non-commercial), OncoKB (proprietary), DrugBank (CC-BY-NC).

---

## Appendix A — Improvements applied

The following 25 specific improvements were identified during planning and have been **applied** to
the PLAN and TASKS above (not left as suggestions):

1. **Adopted Biolink Model as the schema** (vs a bespoke ontology) so the graph is interoperable with
   Monarch/Translator from day one — applied in Solution approach & schema decisions.
2. **Adopted GA4GH VRS for canonical variant identity**, so the same variant across CIViC/ClinVar
   collapses to one node — applied in architecture, metrics (≥95% VRS-keyed), and M2.
3. **Raised the accuracy bar to ≥98%** (vs the 95% used for low-stakes history) to reflect medium
   risk + clinical adjacency — applied in Success metrics and M1/M4 exit criteria.
4. **Made COSMIC/OncoKB/DrugBank explicit, named exclusions** with the *reason* (non-commercial/
   proprietary/NC) rather than a vague "verify licenses" — applied in the rights ledger and Scope.
5. **Introduced license-layer segregation** (CC0 core vs CC-BY/CC-BY-SA overlays) to prevent
   share-alike contamination of the CC0 core — applied as a key architectural decision + M2 exit.
6. **Built a hard scope firewall against patient-facing advice**, routing any such proposal into a
   separate high-risk project — applied across Goals/Non-goals, Scope, Quality gates, Risks.
7. **Defined the countable "assertion" unit** (one Biolink association) so the 100%-provenance CI
   gate is mechanically checkable — applied in provenance model + M0.
8. **Specified a SEPIO-/nanopublication-style provenance record** with named fields (incl. pipeline
   commit hash) rather than "provenance somehow" — applied in architecture + compliance.
9. **Required evidence levels carried verbatim, never re-graded** — added as a goal, a non-goal, a
   metric (reviewer fidelity check), and a risk row.
10. **Added a cross-source contradiction report** so legitimate disagreements stay visible instead of
    being flattened — applied in pipeline step 5 and M2/M3.
11. **Pinned + checksummed source versions** and stamped "as-of" labels for reproducibility and to
    combat stale-evidence risk — applied in pipeline, sustainability, risks.
12. **Named CIViC (CC0) as the M1 single-source proof** — cleanest license + community alignment —
    sequencing the riskiest integration last.
13. **Made the License/Compliance reviewer a hard gate** with a documented empty-seat fallback (M0
    cannot exit) — applied in Governance + Quality gates + Risks.
14. **Made steward adoption a Definition-of-Shipped gate** (not optional), with M0 outreach, so the
    project is "delivered, not merged."
15. **Added an identifier-normalization service** with Bioregistry-backed CURIEs and a ≥99%
    resolution metric — applied in architecture, metrics, M2.
16. **Standardized exports to KGX + RDF/Turtle + JSON-LD + Neo4j**, each with a per-export license
    manifest — applied in architecture + M3.
17. **Kept the explorer no-account, no-PII, read-only** with a standing "not medical advice" banner —
    applied in Security/privacy + M3.
18. **Confined Python to isolated ETL adapters behind a typed interface** so the agent-neutral core
    stays TS (CLAUDE.md compliance) — applied in tech stack.
19. **Added a per-version license re-verification rule** (licenses change) — applied in compliance,
    sustainability, open questions.
20. **Treated Open Targets as mixed-license** (OT-own CC0 vs integrated third-party terms) rather
    than uniformly CC0 — applied in the rights ledger + M2 acceptance.
21. **Added a conservative default on excluded-source cross-references** (ID-only refs excluded unless
    License reviewer approves) — applied in open questions + non-goals.
22. **Made provenance failure a build-breaking CI gate** ("no provenance → does not ship") — applied
    in compliance + M0 CI scaffold.
23. **Recorded the deriving pipeline commit hash in every assertion** for full traceability — applied
    in provenance model + security.
24. **Added a stratified, independent accuracy audit** (auditor ≠ extractor; strata by source +
    edge type) — applied in M1 QA task + metrics.
25. **Distinguished enabling metrics from the headline outcome** (scale/coverage enable; *named
    adoption* is the real "delivered") — applied in Success metrics note + M4.

---

## Review sign-off

**Reviewer pass (self-review for completeness + correctness):**

- **Spec conformance:** all 17 required H2 sections present, in order; metadata header present;
  honest "TO BE SECURED" used for partner + reviewer seats; `verifiedNeed=false` reflected. ✔
- **Cancer guardrails lead compliance:** Data/licensing & compliance opens with the binding
  guardrails (open-access/aggregate/de-identified only; controlled-access out of scope; per-source
  license verification; no medical advice; provenance on every assertion). ✔
- **License correctness:** CIViC/Reactome/ClinVar/OT-own as CC0/PD core; ChEMBL CC-BY-SA and Mondo
  CC-BY held as segregated overlays; **COSMIC/OncoKB/DrugBank-NC flagged and excluded with reasons.**
  Output licensing (MIT/CC0/CC-BY/CC-BY-SA) matched to inputs. ✔
- **Risk-tier correctness:** medium for the research graph; explicit firewall escalating any
  patient-facing surface to a separate high-risk (oncologist+advocate-gated) project. ✔
- **Schema validity:** TASKS.md example Task JSON validated field-by-field against
  `schemas.ts` (all `required` present; enums in range; `verifiedNeed=false`; donated lane needs no
  `fundedBudgetUsd`). ✔
- **Correction applied during review:** ensured every milestone's exit criteria are *measurable*; the
  100%-provenance gate is defined against a *countable* assertion unit; reviewer empty-seat fallbacks
  are explicit; adoption is a shipped-gate, not a nice-to-have. ✔
- **Residual human decisions (flagged, not resolved):** steward identity, reviewer recruitment,
  Reactome/OT current-version license confirmation, excluded-source ID cross-ref policy, overlay
  boundaries, hosting/funded-lane choices (see Open questions). ✔

**Sign-off:** Plan is internally consistent, guardrail-compliant, and schema-mapped. Ready for
maintainer + (once named) License and Scientific reviewer ratification. Outstanding items are
human/governance decisions, not plan gaps.
