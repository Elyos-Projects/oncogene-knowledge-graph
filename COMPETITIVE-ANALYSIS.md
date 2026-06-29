# Competitive + Improvement Analysis — `oncogene-knowledge-graph`

> Analyst review of `PLAN.md` (v0.1.0, 2026-06-28) and `TASKS.md` for the Elyos cancer good-deed
> project: an open, provenance-complete gene → variant → drug → pathway → cancer-type knowledge graph
> built from open-licensed sources (CIViC, Reactome, Open Targets-own, ClinVar) on the Biolink Model +
> GA4GH VRS, with COSMIC / OncoKB / DrugBank-NC flagged-and-excluded. Cancer-wide sibling of the
> Ewing-specific `ewsr1-fli1-knowledge-graph`.
> Web-grounded; sources cited inline.

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually mature: it already adopts the right standards (Biolink, VRS, KGX, SEPIO,
Bioregistry), leads with the binding license gate, defines a countable assertion unit for the 100%
provenance gate, segregates CC0 core from CC-BY/CC-BY-SA overlays, and makes steward adoption a
Definition-of-Shipped gate. The 25 "improvements applied" appendix shows real self-correction. The
findings below are the remaining gaps, in rough priority order.

**1.1 The aggregator-backdoor (license-laundering) risk is under-specified and the single most
dangerous gap.** The plan treats "Open Targets-own computed evidence" as clean CC0. But Open Targets
is itself an aggregator that *integrates COSMIC's Cancer Gene Census (CGC) and COSMIC-curated cancer
hallmarks, plus IntOGen, ClinVar-somatic via EVA, and ~20 other sources* into its target–disease
evidence and association scores ([Open Targets 22.04 release notes](https://blog.opentargets.org/open-targets-platform-22-04-release/);
[Open Targets evidence docs](https://platform-docs.opentargets.org/evidence)). CGC *is* COSMIC
([Open Targets two-years-on, PMC6324073](https://pmc.ncbi.nlm.nih.gov/articles/PMC6324073/)). So:
  - OT's "own" association *score* is a derivative computed partly *over* COSMIC/IntOGen evidence. Is
    the score CC0-clean, or is it a derivative of non-redistributable inputs? The plan does not answer
    this, and "OT-own = CC0" is too coarse a rule. A score column that is a deterministic function of
    a COSMIC-derived datasource may carry the upstream restriction.
  - The same backdoor exists for **CIViC**: CIViC curators read primary literature, but CIViC
    evidence items can *cite and paraphrase* assertions that also appear in OncoKB/COSMIC. CC0 covers
    CIViC's *expression*, but the plan's "no COSMIC/OncoKB content" rule needs to be re-scoped to
    "no COSMIC/OncoKB *records or bulk data*; independently-curated CC0 restatements of the same
    biological fact are permitted" — otherwise the rule is either unenforceable or accidentally bans
    CIViC. This distinction (facts are not copyrightable; curated expression is) is missing and is the
    crux of the whole license posture.
  - **Concrete fix:** add a *per-datasource provenance filter* inside the OT adapter (the plan's
    "per-integrated-source check" is named but has no mechanism). Maintain an explicit `excluded`
    datasource list (cancer_gene_census, cosmic, intogen-if-restricted, oncokb…) and *drop OT
    evidence rows whose `datasourceId` is on it*, keeping only EVA/ClinVar/genetic/known-drug streams
    whose own licenses pass. Record per-row `datasourceId` in provenance so the exclusion is auditable.
    This belongs in `oncogene-kg-extract-ot-001` acceptance criteria, not just prose.

**1.2 Citation-presence vs citation-faithfulness is not gated.** The 100%-provenance metric proves
every edge *has* a PMID/source-record pointer; it does **not** prove the edge's assertion is *actually
supported by* that source. A pipeline (or a donated agent) can attach a real, resolvable PMID to a
wrong claim and still pass CI green. The ≥98% accuracy audit is the only check, and it is a *sampled,
human, post-hoc* gate — not mechanical and not 100%. There is an emerging benchmark exactly for this
("biomedical fact-checking… cancer variant interpretation verification",
[bioRxiv 2025.09.10.675443](https://www.biorxiv.org/content/10.1101/2025.09.10.675443.full.pdf)) that
should be cited and used. **Add a distinct metric: "citation faithfulness" (sampled entailment check
that the cited source supports the asserted predicate+direction), separate from "provenance
completeness."** Right now the two are conflated.

**1.3 Conflicting-claims model is named but under-designed.** The plan says contradictions are
"retained with separate provenance" and a "contradiction report" is produced (M2/M3). But it never
defines *what counts as a contradiction* on a Biolink graph. CIViC vs ClinVar can disagree on
clinical significance (sensitivity vs resistance vs no-effect) for the *same VRS variant + same Mondo
disease + same drug*. Detecting that requires: (a) canonicalized variant+disease+drug keys (fine),
and (b) a predicate/direction normalization so `affects_response_to{sensitivity}` and
`{resistance}` are comparable. Direction/polarity is the hard part and is unmentioned. **Specify a
contradiction as: same (variant, disease, therapy) triple with opposing response-direction or
opposing clinical-significance, each edge keeping its own evidence level.** Without this the
"contradiction report" is undefined work.

**1.4 Variant identity via VRS is correct but the ≥95% VRS-keyed target is optimistic and the failure
mode is under-planned.** VRS computed identifiers work cleanly for simple SNVs/indels but are
genuinely hard for: fusions (the *defining* lesion class for the sibling EWSR1-FLI1 project!),
copy-number, structural variants, and the **"category" / bucket variants CIViC uses heavily** (e.g.
"BRAF Mutation", "EGFR Exon 19 deletions", "KRAS oncogenic mutation"), which are *not* single
sequence variants and have **no canonical VRS allele**. CIViC models these as Molecular Profiles, not
alleles. The plan name-drops `MolecularProfile` but then sets a ≥95% *VRS-keyed* target that these
category variants structurally cannot meet. **Fix:** split the metric — ≥95% of *precise sequence
variants* get VRS IDs; *category/bucket profiles* get GA4GH **categorical variation / VRSATILE /
CatVRS** treatment or a documented Molecular-Profile node with member-variant links, and are excluded
from the VRS-keyed denominator. As written the metric will either fail or pressure the pipeline to
mis-coerce buckets into single alleles (a correctness hazard).

**1.5 Ontology choices are good but two gaps remain.** Biolink + Mondo + ChEBI/ChEMBL + HGNC + Mondo
are the right ecosystem picks (matches Monarch/Translator;
[Biolink universal schema](https://arxiv.org/pdf/2203.13906); [KG-Hub](https://academic.oup.com/bioinformatics/article/39/7/btad418/7211646)).
But: (a) **Sequence Ontology (SO)** is never mentioned — variant *consequence/type* (missense,
frameshift, fusion) needs SO terms to be interoperable; the plan jumps straight to VRS identity and
skips variant *typing*. (b) **Drug normalization is split confusingly:** ChEMBL is CC-BY-SA (overlay
only), yet drugs must be normalized to *something* in the CC0 core. RxNorm is mentioned but is
US-centric and lacks investigational oncology agents; ChEBI (CC-BY, the Biolink-preferred chemical
prefix) is never named. **Decide the CC0-core canonical drug identifier** (likely ChEBI or a
CIVic-native drug ID with NCIt xref) so the core isn't forced to depend on the CC-BY-SA ChEMBL
overlay for basic drug identity.

**1.6 LLM-extraction hallucination is acknowledged only obliquely.** Risk table row "donated-agent
ETL introduces silent extraction errors" + "deterministic, reviewable transforms; flag-on-doubt;
never invent." But the plan's stated pipeline is *deterministic ETL from structured releases*
(CIViC/Reactome/OT/ClinVar all ship structured bulk data — no NLP needed for the core graph). That is
the **right** call and should be stated as a principle: *the CC0 core is built by deterministic
structured-data mapping, not LLM extraction.* Any LLM use (e.g. mining free-text evidence summaries,
literature relation extraction) is then an *explicitly separate, human-curated overlay* — never in the
provenance-gated core. The plan blurs this; make it a hard architectural line (see §5).

**1.7 Weak / missing metrics.**
  - "≥25,000 edges" is a vanity-adjacent enabling metric (the plan even admits this). For calibration:
    CIViC alone has ~6,300+ evidence items ([CIViCdb 2022, NAR](https://academic.oup.com/nar/article/51/D1/D1230/6827107));
    Open Targets has 7.8M target–disease associations over 17M evidence pieces
    ([OT search result / platform 24.x]). 25k is trivially clearable from OT alone and says nothing
    about quality — **risk: the scale gate incentivizes dumping low-value OT association rows.** Tie
    scale to *clinically-meaningful edge types* (variant↔drug response with evidence level), not raw
    count.
  - No **freshness/staleness SLA** as a metric despite "stale data" being the only *High-likelihood*
    risk. Add: max source-version age at release; % edges with as-of label (should be 100%).
  - No **precision/recall on variant merges** metric, despite mis-merge being a High-impact risk.
  - "Downstream adoption ≥1 named use" is the right headline but has no *leading indicator* — add a
    pre-adoption proxy (steward LOI, or a reconciliation PR accepted by VICC/Monarch).

**1.8 Scope vs the EWSR1-FLI1 sibling — only lightly differentiated.** PLAN.md references the sibling
pattern obliquely (revolutionary-patriots-kg companion) but does **not** explicitly resolve the
overlap with `ewsr1-fli1-knowledge-graph`: a cancer-WIDE KG that ingests CIViC *necessarily contains
the EWSR1-FLI1 / Ewing content* the fusion-specific project curates. The plan should state the
relationship explicitly: this project is the **broad substrate** (structured-data, all cancers,
breadth-first); the sibling is the **deep vertical** (fusion-specific, literature-curated,
mechanism-deep, possibly LLM-assisted relation extraction the wide KG forbids in its core). Without
this, the two projects look redundant to a reviewer/steward. **Recommend: declare a shared schema +
shared BioCypher-style adapter template, and position EWSR1-FLI1 as a depth overlay that *reconciles
against* this graph's VRS/Mondo nodes.** (See §4, §7.)

**1.9 Smaller items.**
  - **Reactome nuance:** Reactome *data* is CC0 but its *illustrations/icons/art are CC-BY 4.0*
    ([Reactome license](https://reactome.org/license/)). If the explorer reuses Reactome pathway
    diagrams, that's a CC-BY obligation the "CC0 core" framing misses. Flag explicitly.
  - **HGNC/Ensembl "no restriction"** is asserted but EBI/EMBL terms should be the verified citation;
    keep as an open item (the plan already re-verifies per release — good).
  - **No data-model for "evidence level" heterogeneity:** CIViC A–E, AMP/ASCO/CAP I–IV, ClinVar
    review-status stars, OT scores are *non-comparable scales*. The plan correctly says "carry
    verbatim, never re-grade," but the explorer/consumers need a documented *non-mapping* statement so
    users don't naively compare a CIViC "B" to a ClinVar "2-star." Add to data dictionary.
  - **No mention of ClinVar's somatic vs germline split**; an oncogene KG cares about somatic
    interpretations primarily — specify which ClinVar subset and how germline cancer-predisposition
    variants are labeled.

---

## 2. Competitive landscape

No existing resource is simultaneously (a) cancer-wide variant→drug→pathway, (b) fully open-licensed +
redistributable, (c) 100%-per-assertion provenance, and (d) faithfulness-audited. Each competitor
nails some axes and misses others.

**Open Targets Platform** — the 800-lb gorilla of open target–disease evidence. Harmonizes ~20+
sources into 7.8M target–disease associations with scoring, GraphQL API, ontology mapping (EFO/Mondo).
*Strengths:* scale, funding (EMBL-EBI/pharma consortium), maintained, genuinely open *code* (Apache-2.0)
and largely open data. *Weaknesses for our angle:* it is **target–disease**-centric, not
**variant→drug-response**-centric (it is not a clinical variant interpretation KB); its association
scores are *derived/opaque* (not per-assertion verbatim evidence); and it **launders restricted
sources** (CGC/COSMIC, IntOGen) into its evidence — the exact backdoor we must filter.
([platform.opentargets.org](https://platform.opentargets.org/); [evidence docs](https://platform-docs.opentargets.org/evidence);
[22.04 notes](https://blog.opentargets.org/open-targets-platform-22-04-release/))

**CIViC** — our CC0 seed and closest *content* peer. Expert-crowdsourced clinical interpretation of
cancer variants; ~6,300+ evidence items; gene→variant→evidence→assertion hierarchy; CC0; great API/
CIViCpy. *Strengths:* clean license, verbatim evidence levels, community alignment (WashU). *Weaknesses:*
it is a *knowledgebase, not a harmonized graph* — no VRS canonicalization across sources, no pathway
layer, no cross-source contradiction view, no Biolink export. **We are a superset integrator over it,
not a competitor.** ([CIViCdb 2022, NAR](https://academic.oup.com/nar/article/51/D1/D1230/6827107);
[Nature Genetics 2017](https://www.nature.com/articles/ng.3774))

**OncoKB** (MSK) — the precision-oncology leader, FDA partially-recognized somatic variant database.
*Strengths:* clinical authority, FDA recognition, therapy levels. *Weaknesses for us:* **proprietary /
custom license** — free for academic *use* but redistribution is barred, commercial/hospital use is
paid, and **"OncoKB cannot be used to train AI/ML models"** ([OncoKB licensing FAQ](https://faq.oncokb.org/licensing);
[terms](https://www.oncokb.org/terms)). Correctly flagged-and-excluded. It is the *gap-creating*
incumbent: it exists, it's good, and it's enclosed — which is precisely why an open alternative has
public value.

**VICC meta-knowledgebase (metakb)** — the *most direct conceptual competitor*: it already harmonizes
six cancer variant KBs (CGI, CIViC, JAX-CKB, MolecularMatch, OncoKB, PMKB) into one searchable
meta-KB, defining genes/variants/diseases/drugs/evidence and linking to unambiguous references; built
by the same GA4GH/VICC community whose standards we adopt
([Nature Genetics 2020, metakb](https://www.nature.com/articles/s41588-020-0603-8);
[docs.cancervariants.org](https://docs.cancervariants.org/)). *Strengths:* harmonization framework,
VRS/GA4GH lineage, authoritative community. *Weaknesses / our opening:* it **includes OncoKB and other
restricted KBs**, so the *harmonized whole cannot be redistributed CC0*; it is variant-interpretation
only (no pathway substrate, thinner drug/pathway connective tissue); and it is more a research
artifact/search tool than a maintained, exported, license-segregated open graph. **Our differentiator
is the CC0-clean subset done rigorously** — and we should *reconcile against* metakb (already in the
backlog, `oncogene-kg-vicc-001`), not duplicate it.

**Monarch Initiative KG** — the reference open Biolink KG: 1.12M nodes / 8.91M edges across 40+
sources, Biolink schema, KGX/Koza ETL ([why-we-need-all-organisms, arXiv 2509.18050](https://arxiv.org/pdf/2509.18050);
[KG-Hub, Bioinformatics](https://academic.oup.com/bioinformatics/article/39/7/btad418/7211646)).
*Strengths:* the gold standard for *how* to build an open Biolink KG; ideal reconciliation partner /
steward. *Weakness for our niche:* gene–phenotype–disease centric, **not deep on cancer
variant→drug-response**. We are a cancer-specialized contributor to that ecosystem, not a rival.

**Hetionet / Project Rephetio** (Himmelstein) — 47k nodes / 2.25M edges, 11 node types from 29
resources, famously open and reproducible, drug-repurposing focused
([eLife 2017](https://elifesciences.org/articles/26726); [GitHub dhimmel/integrate](https://github.com/dhimmel/integrate)).
*Strengths:* gold-standard reproducibility/openness ethos; great prior art for license discipline.
*Weaknesses:* a 2017 *static snapshot*, not variant-resolved (gene-level, not variant-level), not
maintained, no evidence-level provenance. Inspirational, not competitive.

**PrimeKG** (Harvard / Zitnik Lab) — precision-medicine KG, 20 resources, 17,080 diseases, ~4M
relationships, strong on drug indication/contraindication/off-label edges; Harvard Dataverse
([Nature Scientific Data 2023](https://www.nature.com/articles/s41597-023-01960-3);
[GitHub mims-harvard/PrimeKG](https://github.com/mims-harvard/PrimeKG)). *Strengths:* ML-ready,
multimodal, disease-centric. *Weaknesses for us:* disease-level not variant-level; provenance is
source-attribution not per-assertion verbatim-evidence; license is a *mix* inherited from 20 sources
(no clean CC0 redistribution guarantee); not cancer-specialized. A strong reuse/benchmark target.

**BioCypher** (Saez-Rodriguez et al.) — *not a KG but the framework to build one*: FAIR, ontology-
mapped, provenance-preserving Python KG-builder ([Nature Biotech 2023](https://www.nature.com/articles/s41587-023-01848-y);
[GitHub biocypher](https://github.com/biocypher/biocypher)). **Strategically important: we should
likely build *on* or *interoperate with* BioCypher adapters rather than hand-roll ETL** — and ship a
reusable BioCypher adapter as a spin-off (§7). Tension: BioCypher is Python; Elyos core is TS — fits
the plan's "Python only inside isolated adapters" rule.

**INDRA / INDRA DB** (Gyori/Bachman) — automated assembly of *causal molecular mechanisms* from text
mining + curated DBs into standardized INDRA Statements with belief scores
([Mol Syst Biol 2023, PMC10167483](https://pmc.ncbi.nlm.nih.gov/articles/PMC10167483/);
[indra.bio](http://www.indra.bio/)). *Strengths:* the leading system for *machine-read* mechanistic
relations with evidence aggregation + belief models — exactly the literature-extraction layer the
sibling project might want. *Weaknesses for our core:* text-mined statements are probabilistic, not
the verbatim-curated provenance our CC0 core demands. **Best used as an optional, clearly-segregated
mechanism overlay (and a model for belief/contradiction handling), never in the gated core.**

**DGIdb** — drug–gene interaction aggregator, 41 sources, open, API/TSV
([DGIdb 4.0, PMC7778926](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7778926/)). Strong for the
drug↔gene edge layer; not variant-resolved, not cancer-specific. A candidate *source* (license-gate
each upstream) more than a competitor.

**Pharos / TCRD** (IDG/NIH) — druggable-genome target portal, 70+ resources, Target Development Levels
([Pharos 2023, NAR](https://academic.oup.com/nar/article/51/D1/D1405/6851109)). Target-centric, not
variant→drug-response; useful for druggability annotation overlays.

**Reactome** — our CC0 pathway substrate (data CC0; art CC-BY 4.0
[license](https://reactome.org/license/)). Authoritative pathways; *not* a variant/drug-response KB —
complementary, ingested, not competitive.

**SIGNOR** — manually-curated causal signaling interactions (activation/inhibition, mechanism)
([SIGNOR 3.0, NAR](https://academic.oup.com/nar/article/51/D1/D631/6761728)). Excellent *directional
causal* layer (helps the contradiction/polarity model in §1.3) — candidate overlay source; verify
license per release.

**Clinical Knowledge Graph (CKG)** (Mann Lab) — 16M nodes / 220M relationships integrating proteomics
+ clinical + literature ([Nature Biotech 2022](https://www.nature.com/articles/s41587-021-01145-6);
[GitHub MannLabs/CKG](https://github.com/MannLabs/CKG)). Proteomics-analysis-centric, clinical-decision
framing; different niche, useful architecture reference.

**Net read:** the white space is real. The closest direct competitor is **VICC metakb** (but it's
license-contaminated by OncoKB and pathway-thin); the biggest *data* gravity wells are **Open Targets**
(target-disease, not variant-response, and laundering COSMIC) and **CIViC** (our seed, not a graph).
Nobody owns "**CC0-clean, redistributable, variant-resolved, pathway-connected, faithfulness-audited
cancer KG.**"

---

## 3. Gaps we can fill

1. **A genuinely redistributable CC0-core cancer KG.** metakb/OT/PrimeKG all carry mixed or restricted
   licenses; none can be handed to a downstream tool builder as *"take this whole graph, CC0, no
   strings."* The license-layer segregation (CC0 core vs CC-BY/CC-BY-SA overlays) is a real,
   under-served deliverable.
2. **Per-assertion verbatim-evidence provenance at scale.** OT gives derived scores; PrimeKG gives
   source attribution; Hetionet gives edge metadata. None give *"this exact CIViC evidence item, level
   B, PMID X, license CC0, as-of version Y, extracted by pipeline commit Z"* on every edge as a
   machine-checkable gate.
3. **Faithfulness, not just presence, of citations** (§1.2) — an audited entailment guarantee is
   absent across all competitors and is the trust differentiator.
4. **Variant-resolved + VRS-canonical + cross-source reconciled.** metakb harmonizes but isn't a
   maintained Biolink/KGX export; OT/PrimeKG are gene/disease-level. A VRS-keyed graph where CIViC and
   ClinVar variants collapse to one node, with contradictions preserved, is the connective tissue
   nobody ships openly.
5. **An explicit, auditable anti-laundering filter** (drop CGC/COSMIC/IntOGen/OncoKB-derived rows from
   aggregator sources, recorded in provenance) — a compliance artifact other open KGs simply don't do,
   and a publishable methods contribution.
6. **A reconciliation contribution back to VICC/Monarch** rather than a walled garden — fills the
   "open subset done rigorously + given back" gap.

---

## 4. Differentiators to win

1. **CC0-clean redistribution as a first-class, segregated product** — the core can be reused with
   zero license friction; overlays are opt-in. No major competitor offers this guarantee.
2. **Provenance-completeness + citation-faithfulness as enforced gates**, not aspirations — CI fails on
   missing provenance; sampled entailment audit on faithfulness. "Trust because traceable."
3. **Anti-aggregator-laundering discipline** (§1.1) — turning the hardest compliance risk into a
   visible feature ("we provably exclude COSMIC/OncoKB-derived rows") that institutions can defend.
4. **Verbatim evidence, contradictions preserved** — we never re-grade or flatten; the contradiction
   report is a feature OT/PrimeKG lack.
5. **Ecosystem-native interoperability** — Biolink + VRS + KGX + Bioregistry means day-one
   reconciliation with Monarch/Translator/VICC; we *join* the open-bio graph rather than fork it.
6. **vs the EWSR1-FLI1 sibling — complementary, not competing.** This is the **broad, structured-data,
   all-cancers substrate**; the sibling is the **deep, fusion-specific, literature-curated vertical.**
   Win condition: a **shared schema + shared BioCypher-style adapter template + shared provenance/
   license-gate machinery**, with EWSR1-FLI1 published as a *depth overlay that reconciles against this
   graph's VRS/Mondo/ChEBI nodes.* One graph cites the other; neither re-curates the other's lane. This
   is also the strongest argument to a steward: fund the substrate once, get N fusion/subtype verticals
   cheaply.
7. **Good-citizen governance** (named reviewers as hard gates, steward-adoption as Definition-of-
   Shipped) — most academic KGs are "publish and abandon"; our delivered-not-merged bar is a
   sustainability differentiator.

---

## 5. Claude API leverage

**Where Claude clearly helps (assistive, human-gated, never authoritative):**

1. **License/ToS triage & datasource-flagging drafts.** Claude can read a source's license + an
   aggregator's datasource manifest and *draft* the allow-list entry (basis, redistribution,
   share-alike, and — critically — *flag suspected laundering*, e.g. "OT row datasourceId=cancer_gene_
   census → COSMIC-derived → recommend exclude"). **Human License reviewer must ratify; Claude never
   self-approves a source** (matches CLAUDE.md and the plan's hard gate).
2. **Citation-faithfulness / entailment checking (highest-value).** For each extracted edge, Claude
   judges whether the cited source text (CIViC evidence statement, abstract) *entails* the asserted
   predicate + direction — an automated first pass that surfaces the ~2% mis-citations *before* the
   human ≥98% audit, dramatically improving audit efficiency. Use the existing cancer-variant
   verification benchmark to calibrate ([bioRxiv 2025.09.10.675443](https://www.biorxiv.org/content/10.1101/2025.09.10.675443.full.pdf)).
   **Claude flags; it does not pass/fail the gate alone.**
3. **Entity normalization assistance.** Propose CURIE mappings for messy drug strings / disease names /
   gene aliases (ChEBI/Mondo/HGNC candidates with confidence) to feed the normalization service. **Every
   *new* grounding is human-confirmed; merges are never silent** (plan already requires this).
4. **Contradiction detection / triage.** Cluster same-(variant,disease,drug) edges and have Claude
   *characterize* the disagreement (direction conflict vs evidence-strength difference vs disease-
   granularity mismatch) to populate the contradiction report — *as a draft for the Scientific
   reviewer*, not a verdict.
5. **Relation extraction from open literature — overlay only.** For the *optional* literature overlay
   (and for the sibling EWSR1-FLI1 vertical), Claude can extract gene→variant→drug→effect relations
   with span-level evidence. This is INDRA-like work and **must live in a segregated, human-curated
   overlay with its own provenance and lower trust tier — never in the CC0 deterministic core.**
6. **Data-dictionary, citation-guide, and "non-comparable evidence scales" explainer drafting** (§1.9)
   — low-risk writing leverage.

**Where Claude must NOT decide (hard lines):**
- **Asserted biology is never Claude-originated.** Every biological edge traces to a source record +
  curator; Claude may *restate/normalize/check*, never *assert* a variant→drug relationship that no
  source contains. No "Claude says BRAF V600E is sensitive to X" without provenance.
- **No unverified claims ship.** Claude's entailment/faithfulness judgments are *signals that route to
  human review*, not green-lights; a Claude "looks entailed" does not satisfy the ≥98% audit.
- **Entity grounding is human-confirmed** for novel mappings; auto-accept only exact, deterministic ID
  matches.
- **License calls are human-verified.** Claude drafts; the License reviewer signs. A Claude license
  reading never authorizes ingestion.
- **No re-grading of evidence.** Claude must carry source evidence levels verbatim; it may not
  "normalize" CIViC B vs ClinVar 2-star into a unified score (§1.9).
- **Claude does not run the core ETL non-deterministically.** The CC0 core is deterministic structured-
  data mapping (reproducible from pinned releases + commit hash); LLM steps are assistive/overlay only,
  so the core stays byte-reproducible.

(For exact model IDs / pricing / caching if a funded-lane run is ever scoped, see the `claude-api`
skill — out of scope for this donated plan.)

---

## 6. Ten concrete optimizations

1. **Add a per-datasource exclusion filter to the OT (and any aggregator) adapter** with an explicit
   `excluded_datasources` list (cancer_gene_census/COSMIC/IntOGen/OncoKB…), drop matching rows, and
   record `datasourceId` in provenance. Promote from prose to `oncogene-kg-extract-ot-001` acceptance
   criteria. (§1.1 — the top priority.)
2. **Split provenance into two metrics:** *completeness* (has resolvable source — CI-gated, 100%) **and**
   *faithfulness* (sampled entailment that source supports the assertion+direction — ≥98%, Claude-
   assisted, human-signed). Add a faithfulness task to M1 QA. (§1.2)
3. **Define "contradiction" formally** as same (VRS variant / Molecular Profile, Mondo disease, drug)
   with opposing response-direction or clinical-significance; specify predicate-polarity normalization
   so the contradiction report is buildable. (§1.3)
4. **Re-split the VRS metric:** ≥95% of *precise sequence variants* VRS-keyed; *category/bucket
   profiles* modeled as CatVRS / Molecular-Profile nodes and excluded from that denominator. Prevents
   the metric forcing bucket-into-allele mis-coercion. (§1.4)
5. **Name the CC0-core canonical drug identifier (ChEBI or CIViC-native + NCIt xref)** so basic drug
   identity doesn't depend on the CC-BY-SA ChEMBL overlay; add **Sequence Ontology** for variant
   consequence typing. (§1.5)
6. **State the "deterministic-core / LLM-overlay" firewall as an architectural principle** and add it to
   the schema + CI policy: core edges must be reproducible from structured releases; any LLM-extracted
   edge carries an `extraction_method=llm` flag, a lower trust tier, and lives in an overlay. (§1.6, §5)
7. **Replace/augment the raw "≥25,000 edges" gate** with a *clinically-meaningful-edge* target (e.g. ≥N
   variant↔therapy response edges with evidence level), so scale can't be gamed by dumping low-value OT
   rows. (§1.7)
8. **Add freshness + merge-precision metrics:** max source-version age at release; 100% as-of labeling;
   sampled variant-merge precision/recall (both are named risks with no metric). (§1.7)
9. **Adopt or interoperate with BioCypher/KGX-Koza for the ETL substrate** instead of hand-rolling, and
   reuse Monarch's KG-Hub patterns — reduces build cost and buys instant ecosystem compatibility; ship
   the adapters as a reusable template (§7). (§2)
10. **Make the EWSR1-FLI1 relationship explicit in PLAN + TASKS:** shared schema/template, sibling as a
    depth overlay reconciling against this graph's nodes; add a `oncogene-kg-sibling-recon-001` backlog
    task. Removes the redundancy objection and strengthens the steward pitch. (§1.8, §4.6)

---

## 7. Parallel & perpendicular spin-offs

- **Generalized open biomedical KG template / BioCypher adapter pack.** The license-gate + provenance-
  stamp + CC0/overlay-segregation + faithfulness-audit machinery is domain-agnostic. Package it as a
  reusable Elyos "provenance-first open-KG" template (with a published BioCypher adapter + KGX export),
  so the next vertical — cardiac, rare-disease, neuro — is a config, not a rebuild. Directly powers the
  revolutionary-patriots-kg sibling pattern the plan already cites.
- **EWSR1-FLI1 (and other fusion/subtype) depth overlays.** Position the sibling project as the first
  *depth vertical* on this substrate: it reuses the schema, reconciles to these VRS/Mondo nodes, and
  adds literature-curated mechanism depth (where Claude relation-extraction + INDRA-style assembly are
  appropriate, in an overlay). Template generalizes to KRAS, EGFR, ALK-fusion verticals.
- **`biomarker-extraction` service.** A standalone, human-gated Claude pipeline that turns open
  literature / trial abstracts into candidate (variant, disease, drug, effect, evidence) tuples with
  span-level provenance — feeding *both* this KG's overlay and the sibling verticals. Clear trust-tier
  separation from the core.
- **`drug-target-evidence-cards` / `ewing-drug-target-evidence`.** Auto-generated, fully-cited
  one-page "evidence cards" for a (gene/variant, drug) pair, assembled *only* from the provenanced
  graph — a human-readable, shareable surface for researchers/advocates that inherits the
  not-medical-advice frame. Natural first *downstream-adoption* artifact to satisfy the M4 gate.
- **VICC/Monarch reconciliation contribution** (already `oncogene-kg-vicc-001`) as its own
  publishable artifact: an SSSOM mapping set + a "CC0-clean subset of cancer variant interpretations"
  that VICC/metakb can ingest where their license allows.
- **MCP server over the graph.** A read-only Model Context Protocol server exposing query +
  provenance-fetch tools so *other* agents (including donated-lane Claude runs) can ground cancer-
  genomics answers in the cited graph — every answer carries source + evidence level + as-of, with the
  not-medical-advice frame enforced server-side. High-leverage, low-risk, and a strong adoption hook.
- **Staleness/refresh monitor as a service** — scheduled diff-review + re-license-gate (backlog
  `oncogene-kg-refresh-001`) generalized into a reusable "open-data freshness watchdog."

---

## 8. Open questions for the maintainer

1. **Aggregator laundering policy (blocking):** For Open Targets, do we ingest *only* rows whose
   `datasourceId` is itself open-licensed and *drop* CGC/COSMIC/IntOGen/OncoKB-derived rows — and is
   OT's *association score* (a derivative computed partly over those) admissible to the CC0 core, or
   does it move to an overlay? This single decision gates M2.
2. **Facts vs expression:** Do we adopt the principle that *biological facts are not copyrightable, only
   curated expression is* — so independently-curated CC0 restatements (CIViC) of a fact also in
   COSMIC/OncoKB are permitted? Without this the exclusion rule is either unenforceable or bans CIViC.
3. **Category/bucket variants:** How do we model CIViC Molecular Profiles / "BRAF Mutation"-style
   buckets that have no canonical VRS allele, and how does that reshape the ≥95% VRS-keyed metric?
4. **Faithfulness gate:** Will we add a citation-faithfulness (entailment) audit distinct from
   provenance-completeness, and is a Claude-assisted first pass acceptable to the Scientific reviewer?
5. **Build substrate:** Build on BioCypher/KGX-Koza (Python, isolated adapters) or hand-roll TS ETL?
   Affects cost, ecosystem fit, and the reusable-template spin-off.
6. **Sibling boundary:** Formalize the relationship to `ewsr1-fli1-knowledge-graph` — shared schema +
   template, sibling as depth overlay? Who owns the shared substrate?
7. **CC0-core drug identity:** ChEBI vs RxNorm vs CIViC-native+NCIt as the canonical drug node in the
   core (so it doesn't depend on the CC-BY-SA ChEMBL overlay)?
8. **Scale metric:** Replace raw "≥25,000 edges" with a clinically-meaningful-edge target? What counts?
9. **Reactome art:** Will the explorer reuse any CC-BY Reactome illustrations (attribution obligation),
   or text/structure only?
10. Existing plan open-Qs (steward, reviewer recruitment, Reactome/OT per-release license, ID-only
    cross-refs to excluded sources, overlay boundary, hosting, funded-lane) remain valid and unchanged.

---

### Source list (key citations)
- Open Targets: [platform](https://platform.opentargets.org/) · [evidence docs](https://platform-docs.opentargets.org/evidence) · [22.04 notes](https://blog.opentargets.org/open-targets-platform-22-04-release/) · [two-years-on PMC6324073](https://pmc.ncbi.nlm.nih.gov/articles/PMC6324073/)
- CIViC: [CIViCdb 2022 NAR](https://academic.oup.com/nar/article/51/D1/D1230/6827107) · [Nat Genet 2017](https://www.nature.com/articles/ng.3774)
- OncoKB: [licensing FAQ](https://faq.oncokb.org/licensing) · [terms](https://www.oncokb.org/terms)
- VICC metakb: [Nat Genet 2020](https://www.nature.com/articles/s41588-020-0603-8) · [docs](https://docs.cancervariants.org/)
- Monarch / Biolink / KG-Hub: [arXiv 2509.18050](https://arxiv.org/pdf/2509.18050) · [Biolink arXiv 2203.13906](https://arxiv.org/pdf/2203.13906) · [KG-Hub Bioinformatics](https://academic.oup.com/bioinformatics/article/39/7/btad418/7211646)
- Hetionet: [eLife 2017](https://elifesciences.org/articles/26726) · [GitHub](https://github.com/dhimmel/integrate)
- PrimeKG: [Sci Data 2023](https://www.nature.com/articles/s41597-023-01960-3) · [GitHub](https://github.com/mims-harvard/PrimeKG)
- BioCypher: [Nat Biotech 2023](https://www.nature.com/articles/s41587-023-01848-y) · [GitHub](https://github.com/biocypher/biocypher)
- INDRA: [Mol Syst Biol 2023 PMC10167483](https://pmc.ncbi.nlm.nih.gov/articles/PMC10167483/) · [indra.bio](http://www.indra.bio/)
- DGIdb: [DGIdb 4.0 PMC7778926](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7778926/)
- Pharos/TCRD: [Pharos 2023 NAR](https://academic.oup.com/nar/article/51/D1/D1405/6851109)
- SIGNOR: [SIGNOR 3.0 NAR](https://academic.oup.com/nar/article/51/D1/D631/6761728)
- CKG: [Nat Biotech 2022](https://www.nature.com/articles/s41587-021-01145-6) · [GitHub](https://github.com/MannLabs/CKG)
- Reactome license: [reactome.org/license](https://reactome.org/license/) · ChEMBL: [chembl-licensing](http://chembl.github.io/chembl-licensing/)
- Faithfulness benchmark: [bioRxiv 2025.09.10.675443](https://www.biorxiv.org/content/10.1101/2025.09.10.675443.full.pdf)
