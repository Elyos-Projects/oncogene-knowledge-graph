# oncogene-knowledge-graph

> Cancer is a disease of the genome. A clinician or researcher trying to understand a single tumor alteration — say *BRAF* V600E in melanoma — must today stitch together facts from a dozen disconnected   ·  **Risk tier:** med  ·  **Status:** planning

Cancer is a disease of the genome. A clinician or researcher trying to understand a single tumor alteration — say *BRAF* V600E in melanoma — must today stitch together facts from a dozen disconnected resources: the variant's clinical interpretation (CIViC), the targeted drugs and their evidence (Open Targets, CIViC, ChEMBL), the biological pathway the gene sits in (Reactome), the disease ontology term (Mondo), and the primary literature (PubMed). These resources use different identifiers, different variant nomenclature, different disease vocabularies, and different evidence frameworks. The connective tissue — *which variant, in which cancer, confers sensitivity or resistance to which drug, acting through which pathway, on what strength of evidence* — exists only in researchers' heads and in proprietary, paywalled databases.

**Definition of shipped:** sources**; **100% per-assertion provenance** (CI-enforced); identifier resolution ≥99% and ≥95% of variants VRS-keyed; independent accuracy audit **≥98%**; standard exports + read-only explorer (with "not medical advice" banner) live; per-export license manifests correct; **a nam

This is an **Elyos** good-deed project. Contributors pull a task, do it with their own coding agent, and open a PR. Platform: https://github.com/jdev1977/elyos

## Plan
- [PLAN.md](./PLAN.md) — robust enterprise plan (vision, architecture, roadmap, risks; includes an applied-improvements appendix + review sign-off)
- [TASKS.md](./TASKS.md) — schema-mapped task backlog
- [tasks/](./tasks/) — ready-to-pull task JSON(s)

## Contribute
```bash
elyos browse
elyos next --repo Elyos-Projects/oncogene-knowledge-graph --no-fork
```

## Licensing & review
- Open license (see PLAN.md).
- Risk tier **med** — deeds are *delivered, not merged*; a domain reviewer (and expert sign-off for any high-stakes content) must approve before merge.

> Planning stage; no adopting partner secured yet (`verifiedNeed: false` on delivery-dependent tasks).
