---
name: thermonuclear-explosion-modeling-terminology
description: 'Use when a task asks to reproduce the science-results summary (e.g.
  Figure 6) of the Castro/Maestro/AMReX paper: four qualitative claims about sub-Chandra
  ignition, XRB convection, WD merger, and the Maestro→Castro restart.'
when_to_use_negative:
- The input is repository documentation (e.g. the AMReX README) rather than the article's
  science-results passages — the README contains no simulation findings.
- The task requires quantitative extraction (numerical tables, code parameters, benchmark
  timings) rather than the four qualitative science claims.
- The target paper describes only one phase of the evolution (e.g. only the Castro
  explosion) — the skill's claims span both subsonic Maestro and compressible Castro
  phases.
edam_operation: http://edamontology.org/operation_0306
tools:
- name: Maestro
  role: Source of the sub-Chandra and X-ray-burst convection results and the output
    that is restarted in Castro once the flow becomes sonic
- name: Castro
  role: Code used for the white-dwarf-merger simulation (0.9 + 0.6 solar-mass companion)
    and for continuing Maestro restarts into the sonic regime
- name: AMReX
  role: Shared block-structured AMR library underlying both codes; relevant for locating
    code repositories cited alongside the results
  repo: https://github.com/AMReX-Codes/amrex
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:3f94c35b88b8593449d38bc5b34573342cff4799bdee1e05fcb054dd1661b4ff
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
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# thermonuclear-explosion-modeling-terminology

## Summary

Extracts and structures the qualitative science-result claims of Zingale et al. 2018 (J. Phys. Conf. Ser. 1031, 012024) — sub-Chandra convection ignition, 3D X-ray-burst convection, the Castro white-dwarf-merger simulation, and the Maestro-to-Castro restart demonstration — into validated JSON records keyed to figure/section citations.

## When to use

Use when a task asks to reproduce the science-results summary (e.g. Figure 6) of the Castro/Maestro/AMReX paper: four qualitative claims about sub-Chandra ignition, XRB convection, WD merger, and the Maestro→Castro restart.

## When NOT to use

- The input is repository documentation (e.g. the AMReX README) rather than the article's science-results passages — the README contains no simulation findings.
- The task requires quantitative extraction (numerical tables, code parameters, benchmark timings) rather than the four qualitative science claims.
- The target paper describes only one phase of the evolution (e.g. only the Castro explosion) — the skill's claims span both subsonic Maestro and compressible Castro phases.

## Inputs

- Full text of Zingale et al. 2018, J. Phys. Conf. Ser. 1031, 012024 (DOI 10.1088/1742-6596/1031/1/012024), OA PDF at https://iopscience.iop.org/article/10.1088/1742-6596/1031/1/012024/pdf
- Figure 6 and the surrounding science-results section text

## Outputs

- JSON array of exactly 4 objects, each with fields 'result' (qualitative claim paraphrase) and 'citation_location' (figure or section number)
- Records containing the substrings 'Kolmogorov' (XRB convection entry) and 'restart' (Maestro-to-Castro entry)

## How to apply

Read the full article and locate the four science-result passages: (1) sub-Chandra convection ignition and its dependence on nova-like vs localized ignition as a function of WD/He-layer mass, (2) the first 3D X-ray-burst convection model exhibiting a fully turbulent Kolmogorov spectrum, (3) the Castro merger of a 0.9 solar-mass primary and 0.6 solar-mass companion to assess burning sensitivity, and (4) the restart of Maestro output in Castro once the flow becomes sonic for the Chandra SNe Ia model. For each result, record a verbatim-adjacent paraphrase together with its cited figure or section number. Build a JSON array of exactly four objects, each with the fields 'result' and 'citation_location', ensuring the substrings 'Kolmogorov' and 'restart' appear in the appropriate entries. Rationale: the paper states that no single algorithm covers all phases, so each claim is tied to the phase (subsonic Maestro vs compressible Castro) in which it was obtained.

## Related tools

- **Maestro** (Source of the sub-Chandra and X-ray-burst convection results and the output that is restarted in Castro once the flow becomes sonic)
- **Castro** (Code used for the white-dwarf-merger simulation (0.9 + 0.6 solar-mass companion) and for continuing Maestro restarts into the sonic regime)
- **AMReX** (Shared block-structured AMR library underlying both codes; relevant for locating code repositories cited alongside the results) — https://github.com/AMReX-Codes/amrex

## Examples

```
python -c "import json; d=json.load(open('fig6_results.json')); assert len(d)==4 and all('result' in e and 'citation_location' in e for e in d); assert any('Kolmogorov' in e['result'] for e in d) and any('restart' in e['result'] for e in d)"
```

## Evaluation signals

- The output file parses as valid JSON.
- json.load + schema check confirms exactly 4 entries, each containing non-empty 'result' and 'citation_location' keys.
- The substring 'Kolmogorov' occurs in the X-ray-burst convection entry.
- The substring 'restart' occurs in the Maestro-to-Castro entry.
- Each 'citation_location' resolves to a figure or section number actually present in the article.

## Limitations

- The article's science results are qualitative claims; no numerical thresholds or data tables are provided for these four results.
- The restart claim is specific to the Chandra model SNe Ia pathway (Maestro subsonic → Castro sonic); it does not generalize to all models in the paper.
- Repository context (AMReX README, BoxLib deprecation notice) is separate from the article and must not be conflated with the paper's findings.

## Evidence

- [abstract] Maestro subsonic vs Castro compressible division of the science results: "Maestro is tuned to efficiently model subsonic convective flows while Castro models the highly compressible flows associated with stellar explosions."
- [abstract] Application domain of the four claims (Type Ia SNe and X-ray bursts): "Together, these codes enable a thorough investigation of stellar phenomena, including Type Ia supernovae and X-ray bursts."
- [intro] Rationale for why the four results come from different codes/phases: "No single algorithm meets all of the demands imposed by these events. Instead, we advance our understanding of these events by piecing together simulations of different phases"
- [intro] Division of labor: Maestro early subsonic runaway, Castro the explosion: "designed to perform three-dimensional models of the early subsonic evolution leading to runaway and the subsequent explosion, respectively"
