---
name: astrophysical-simulation-result-curation
description: Use when a task requires reproducing the paper's Figure 6 science-results
  summary — i.e., extracting per-problem qualitative claims (ignition regime, Kolmogorov
  turbulence, WD-merger masses, sonic-regime restart) with their figure/section citations
  from the article text.
when_to_use_negative:
- The task requires quantitative simulation data (temperatures, burning rates, grid
  parameters) — the paper supplies only qualitative per-problem claims and no numeric
  thresholds to extract.
- The task is about BoxLib-to-AMReX code migration or AMReX contribution workflow
  — this skill curates science results, not software-engineering procedures.
- Only the AMReX/BoxLib READMEs are available and the article full text is not — the
  four science results cannot be sourced from repository documentation.
edam_operation: http://edamontology.org/operation_0306
tools:
- name: Maestro
  role: Source of the subsonic-regime science results (sub-Chandra convection ignition,
    X-ray-burst convection) and the output that is restarted in Castro
- name: Castro
  role: Source of the compressible-regime results (white-dwarf merger of a 0.9 and
    0.6 solar-mass WD) and the code used for the sonic-regime restart demonstration
- name: AMReX
  role: Block-structured AMR library both codes are built on; its GitHub README provides
    the project documentation context for the suite
  repo: https://github.com/AMReX-Codes/amrex
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:4e4d3981b7f1de79dcd987b21ff2539e12fbbdddbffb5b3f6a1a55405e191b70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AMReX-Codes/amrex
  tools:
  - AMReX
  - Castro
  - Maestro
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# astrophysical-simulation-result-curation

## Summary

Curates qualitative science-result claims from the Maestro/Castro thermonuclear-explosion paper (Zingale et al. 2018, J. Phys. Conf. Ser. 1031, 012024) into a structured four-entry JSON register covering sub-Chandra convection ignition, X-ray-burst convection, the Castro white-dwarf merger, and the Maestro-to-Castro restart demonstration.

## When to use

Use when a task requires reproducing the paper's Figure 6 science-results summary — i.e., extracting per-problem qualitative claims (ignition regime, Kolmogorov turbulence, WD-merger masses, sonic-regime restart) with their figure/section citations from the article text.

## When NOT to use

- The task requires quantitative simulation data (temperatures, burning rates, grid parameters) — the paper supplies only qualitative per-problem claims and no numeric thresholds to extract.
- The task is about BoxLib-to-AMReX code migration or AMReX contribution workflow — this skill curates science results, not software-engineering procedures.
- Only the AMReX/BoxLib READMEs are available and the article full text is not — the four science results cannot be sourced from repository documentation.

## Inputs

- Full text of Zingale et al. 2018, J. Phys. Conf. Ser. 1031, 012024 (DOI 10.1088/1742-6596/1031/1/012024), PDF at https://iopscience.iop.org/article/10.1088/1742-6596/1031/1/012024/pdf
- Figure 6 and the surrounding science-result passages (sub-Chandra, XRB, WD-merger, restart sections)
- Article-enriched index of tools, findings, and section-scoped evidence spans

## Outputs

- JSON array of exactly four objects, each with fields 'result' (qualitative claim paraphrase) and 'citation_location' (figure/section reference)
- Validated record confirming substrings 'Kolmogorov' and 'restart' occur in the correct entries

## How to apply

Read the full article and locate the four science-result passages: sub-Chandra convection ignition (nova-like vs. localized ignition dependence on WD/He-layer mass), the first 3D X-ray-burst convection model with a fully turbulent Kolmogorov spectrum, the Castro merger of a 0.9 solar-mass primary with a 0.6 solar-mass companion to assess burning sensitivity, and the restart of Maestro output in Castro once the flow becomes sonic. For each passage record a verbatim-adjacent paraphrase as 'result' plus the cited figure or section number as 'citation_location'. Build a JSON array of exactly four objects with those two fields, ensuring the substrings 'Kolmogorov' and 'restart' land in the appropriate entries. Rationale: the paper explicitly frames its contributions as piecing together simulations of different explosion phases from Maestro (subsonic convection) and Castro (compressible explosion), so each entry must be tied to the code and phase it describes.

## Related tools

- **Maestro** (Source of the subsonic-regime science results (sub-Chandra convection ignition, X-ray-burst convection) and the output that is restarted in Castro)
- **Castro** (Source of the compressible-regime results (white-dwarf merger of a 0.9 and 0.6 solar-mass WD) and the code used for the sonic-regime restart demonstration)
- **AMReX** (Block-structured AMR library both codes are built on; its GitHub README provides the project documentation context for the suite) — https://github.com/AMReX-Codes/amrex

## Examples

```
python -c "import json; d=json.load(open('results.json')); assert len(d)==4 and all('result' in e and 'citation_location' in e for e in d); assert any('Kolmogorov' in str(e) for e in d) and any('restart' in str(e) for e in d)"
```

## Evaluation signals

- Parsed JSON contains exactly 4 entries — one per science result (sub-Chandra, XRB, WD-merger, restart).
- Every entry has both 'result' and 'citation_location' present and non-empty.
- Substring 'Kolmogorov' appears in the X-ray-burst convection entry.
- Substring 'restart' appears in the Maestro-to-Castro restart entry.
- Each 'citation_location' names a real figure or section number from the article (e.g., Figure 6).

## Limitations

- Claims are qualitative paraphrases ('verbatim-adjacent'), not numeric results — the article index provides no thresholds or parameter values for the four science problems.
- One source document (github.com__AMReX-Codes__amrex) lacks DOI, title, authors, year, or journal metadata, so citation provenance must come from the article metadata (DOI 10.1088/1742-6596/1031/1/012024).
- The science-result passages must be located in the full article text; the extracted entity index alone does not contain the verbatim Figure 6 passages.

## Evidence

- [abstract] Codes model distinct explosion phases: "Maestro is tuned to efficiently model subsonic convective flows while Castro models the highly compressible flows associated with stellar explosions."
- [abstract] Suite enables investigation of the target phenomena: "Together, these codes enable a thorough investigation of stellar phenomena, including Type Ia supernovae and X-ray bursts."
- [intro] Results are pieced together across codes and phases: "No single algorithm meets all of the demands imposed by these events."
- [readme] AMReX README identifies the underlying framework: "A software framework for massively parallel block-structured adaptive mesh refinement applications."
