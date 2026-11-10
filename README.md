<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/brand/asb-logo-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="assets/brand/asb-logo.svg">
    <img alt="AgenticScienceBuilder" src="assets/brand/asb-logo.svg" width="320">
  </picture>
</p>

<h1 align="center">ASB Skills: Astronomy and astrophysics</h1>

<p align="center"><em>Evidence-grounded skills and tool records for astronomy and astrophysics software: data reduction, catalogues, simulation and statistical inference.</em></p>

<p align="center">
  <img src="https://img.shields.io/badge/skills-1%2C671-00284b" alt="1,671 skills">
  <img src="https://img.shields.io/badge/tools-415-00284b" alt="415 tools">
  <img src="https://img.shields.io/badge/workflows-2-00284b" alt="2 workflows">
  <img src="https://img.shields.io/badge/papers-100-00284b" alt="100 source papers">
  <img src="https://img.shields.io/badge/content-CC--BY--4.0-04919a" alt="content CC-BY-4.0">
  <img src="https://img.shields.io/badge/code-Apache--2.0-04919a" alt="code Apache-2.0">
  <img src="https://img.shields.io/badge/status-released-04919a" alt="status released">
</p>

---

## TL;DR

This repository holds the astronomy skill collection of AgenticScienceBuilder (ASB): 1,671 skills over 415 software tools and 2 composite workflows, distilled from 100 method papers and their code repositories. Each skill is a `SKILL.md` file that an agent can load. It names its source paper; 1,197 of the 1,671 also quote it or its repository README. The collection installs as a Claude Code plugin, and a script that needs only the Python standard library searches it offline.

## Collections

| Path | Title | Skills | Tools | Released | DOI |
|---|---|---|---|---|---|
| `collections/astronomy/v1` | ASB Astronomy and astrophysics Skill Collection v1 | 1671 | 415 | 2026-11-09 | [10.5281/zenodo.24093095](https://doi.org/10.5281/zenodo.24093095) |

## Status

Collection `v1`, built from corpus cut `41886c4e1649`, is released on `main` at tag `astronomy-v1.0.0` and archived on Zenodo under the version DOI [10.5281/zenodo.24093095](https://doi.org/10.5281/zenodo.24093095).

| Item | State |
|---|---|
| Release gate | `receipts/41886c4e1649/gate_report.json` reports `pass`: 15 of 16 checks pass and 1 is not applicable (`cut_manifest`). [`receipts/41886c4e1649/VERIFY.md`](receipts/41886c4e1649/VERIFY.md) gives the command that re-verifies it. |
| Plugin manifests | `claude plugin validate` passes for `.claude-plugin/marketplace.json` and for `collections/astronomy/v1` |
| Collection IRI | `https://w3id.org/holobiomicslab/asb-skill/collection/astronomy/v1` |
| IRI resolution (2026-10-30) | Registered at w3id.org. It redirects (HTTP 302) to a path in the asb-skill-collections registry that returns 404: the collection lives in this repository, not in the registry. |
| DOI | [10.5281/zenodo.24093095](https://doi.org/10.5281/zenodo.24093095) (this version). The Zenodo deposit metadata is in [`.zenodo.json`](.zenodo.json). |
| Concept DOI | [10.5281/zenodo.24093094](https://doi.org/10.5281/zenodo.24093094) (all versions) |

## What is inside

| Item | Count | Path under `collections/astronomy/v1/` |
|---|--:|---|
| Leaf skills (`SKILL.md`) | 1,671 | `leaves/<slug>/SKILL.md` |
| Skill router | 1 | `skills/_router/` |
| Composite workflows | 2 | `workflows/<slug>/` |
| Workflow router | 1 | `workflows/_workflow_router/` |
| Software-tool records | 415 | `tools/` |
| Source papers | 100 | `corpus.yaml` |
| Indexes | 4 | `skills_index.json`, `tools_index.json`, `workflows/workflows_index.json`, `kb_bundle.json` |
| Offline search script | 1 | `bin/search_skills.py` |

The build and gate receipts are in `receipts/41886c4e1649/` at the repository root.

Composite workflows chain leaf skills into a typed pipeline:

| Workflow | Use when | Stages |
|---|---|--:|
| `cmb-cosmological-parameter-inference` | observed CMB likelihood data and Lambda-CDM priors must be assembled into a callable CAMB likelihood, sampled for cosmological parameters, cleared of burn-in, diagnosed for autocorrelation, and summarized as physical constraints | 5 |
| `transit-lightcurve-mcmc-comparison` | one or more photometric transit light curves must be detrended, fitted jointly with a transit-and-noise MCMC model, and compared with an external table of published transit parameters | 3 |

## Licence tiers

Each skill carries a `license_tier` that answers one question: what may you do with the underlying tool?
The tiers are defined in the registry's [licence-tier policy](https://github.com/HolobiomicsLab/asb-skill-collections/blob/main/governance/LICENSE_TIERS.md).
The tier is advice about the tool. It does not change the licence of this collection's content.

| Tier | Skills |
|---|--:|
| `open` | 1,092 |
| `noncommercial` | 44 |
| `restricted` | 46 |
| `unknown` | 489 |

## Install

Claude Code, from the directory that holds your clone of this repository:

```bash
claude plugin marketplace add ./asb-skills-astronomy
claude plugin install astronomy@asb-skills-astronomy
```

Other agents: every skill is one Markdown file with YAML front matter. Point the agent at `collections/astronomy/v1/leaves/`, or let it call the search script below.

### Only a few skills

Each leaf directory holds a single `SKILL.md`, which does not refer to other files of the collection. Copy the leaves you need, from the directory that holds your clone. Claude Code reads the skills of a project from `<project>/.claude/skills/`:

```bash
mkdir -p <project>/.claude/skills
cp -R asb-skills-astronomy/collections/astronomy/v1/leaves/mcmc-transit-light-curve-fitting <project>/.claude/skills/
```

The search script below prints the slug of each match.

## Search offline

The search script needs only the Python standard library.

```bash
cd asb-skills-astronomy
python3 collections/astronomy/v1/bin/search_skills.py --query "transit light curve fitting" -k 3
python3 collections/astronomy/v1/bin/search_skills.py --target workflows --query "transit light curve fitting" -k 2
```

Each result prints its rank and slug, then its description, tools, licence tier and the path of the file to read. The ranked slugs:

```text
1. mcmc-transit-light-curve-fitting
2. multi-light-curve-simultaneous-transit-fitting
3. light-curve-gradient-measurement

1. transit-lightcurve-mcmc-comparison
```

## How it was built

The AgenticScienceBuilder pipeline read each paper and its code repository, wrote the skills, and passed the collection through a strict release gate. `receipts/41886c4e1649/source.json` names the pipeline commit `da8abd15`, the collections-tooling commit `b8c22a6bc` and the corpus commit `afa4911`.

The gate checks that every included paper has an allowed access type, caps the length and similarity of verbatim quotes, scans quote spans for personal data and dual-use content, requires provenance and a licence tag on every skill, resolves each composite workflow to real leaves, and binds the receipt to the hashes of the collection files.

97 of the 100 included papers are open access according to `corpus.yaml` (`access.is_oa`). The other 3 are closed access and listed as `link-only`. 2 of the 100 are arXiv preprints.

Each leaf names its source paper in `provenance.source_papers` (1,671 of 1,671 leaves). 1,197 of the 1,671 quote it, or its repository README, in their `## Evidence` section. In the other 474, the gate parsed no quote span (`gate_report.json`, `strip_verbatim_similarity`, `coverage.files_without_spans`). In 464 of them the section is empty or its quotes are withheld because they could not be anchored in the source. In 8 the section holds a quote in a form the gate parser does not read. 2 have no `## Evidence` section.

## Related repositories

- [ASB skill collections](https://github.com/HolobiomicsLab/asb-skill-collections): the registry of every domain, the shared tooling, the governance documents, the contribution loop and the earlier metabolomics v2 tree. The current metabolomics collection (v3) is in the `asb-skills-metabolomics` repository.
- AgenticScienceBuilder: the pipeline that generated this collection. Repository follows.
- `indicium-adapters-astronomy`: the Indicium domain adapter for the same domain. Repository follows.

## Citation

Cite the collection with [`CITATION.cff`](CITATION.cff). Cite also the source paper of each skill you use: its DOI is in `provenance.source_papers` in the skill's front matter. The collection's version DOI is [10.5281/zenodo.24093095](https://doi.org/10.5281/zenodo.24093095); its concept DOI, which covers all versions, is [10.5281/zenodo.24093094](https://doi.org/10.5281/zenodo.24093094).

## Licence

Code (the search script and other scripts): Apache-2.0, see [`LICENSE`](LICENSE). Collection content (`SKILL.md` files, tool records, indexes, metadata): CC-BY-4.0, see [`LICENSING.md`](LICENSING.md). Short verbatim quotes from source papers are reproduced for attribution under quotation right. A rights holder can ask for removal through an issue.
