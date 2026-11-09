---
name: figure-section-citation-mapping
description: Use when a task asks to reproduce a paper's figure or results summary
  and each claim must be paired with its citation location (figure/section number)
  in a structured, machine-checkable output.
when_to_use_negative:
- The paper is a pure software/methods paper (like the AMReX README) with no figure-cited
  science results to map
- The task only needs tool/repo metadata extraction, not claims tied to figures or
  section numbers
- The source lacks explicit figure or section numbers, so citation_location cannot
  be populated verifiably
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: Maestro
  role: Source of the sub-Chandra, XRB, and subsonic-regime results being summarized
    and figure-mapped
- name: Castro
  role: Source of the white-dwarf-merger result and the Maestro restart demonstration
    described in the JSON records
- name: AMReX
  role: Underlying AMR framework context for the Maestro/Castro results being cited;
    its repo is also the untrusted-README source the curator must not treat as instructions
  repo: https://github.com/AMReX-Codes/amrex
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1088/1742-6596/1031/1/012024
    title: 'Meeting the Challenges of Modeling Astrophysical Thermonuclear Explosions:
      Castro, Maestro, and the AMReX Astrophysics Suite'
  source_content_hash: sha256:4e258402cfbe7174495945be048550e51d0760232bc398806e1e150bb3a6c8b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AMReX-Codes/amrex
  tools:
  - AMReX
  - Castro
  - Maestro
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# figure-section-citation-mapping

## Summary

Extracts each qualitative science result from a paper (e.g., Zingale et al. 2018's four Maestro/Castro results: sub-Chandra convection ignition, X-ray-burst turbulence, the Castro white-dwarf merger, and the Maestro-to-Castro restart) and anchors each to its cited figure or section, emitting a validated JSON array.

## When to use

Use when a task asks to reproduce a paper's figure or results summary and each claim must be paired with its citation location (figure/section number) in a structured, machine-checkable output.

## When NOT to use

- The paper is a pure software/methods paper (like the AMReX README) with no figure-cited science results to map
- The task only needs tool/repo metadata extraction, not claims tied to figures or section numbers
- The source lacks explicit figure or section numbers, so citation_location cannot be populated verifiably

## Inputs

- Full-text article PDF/source (e.g., Zingale et al. 2018, J. Phys. Conf. Ser. 1031, 012024, OA URL https://iopscience.iop.org/article/10.1088/1742-6596/1031/1/012024/pdf)
- List of science-result passages to extract (from the task card workflow)
- Required substring tokens for validation (e.g., 'Kolmogorov', 'restart')

## Outputs

- JSON array with exactly N objects, each containing 'result' and 'citation_location' string fields
- Validation report confirming entry count, required fields, and presence of required substrings

## How to apply

Read the full article and locate each science-result passage — for Zingale et al. 2018 (J. Phys. Conf. Ser. 1031, 012024) these are the sub-Chandra convection-ignition result (nova-like vs localized ignition dependence on WD/He-layer mass), the first 3D XRB convection model showing a fully turbulent Kolmogorov spectrum, the Castro merger of a 0.9 solar-mass primary with a 0.6 solar-mass companion to assess burning sensitivity, and the restart of Maestro output in Castro once the flow becomes sonic. For each result, record a verbatim-adjacent paraphrase plus the exact figure or section number the paper cites. Build a JSON array with exactly four objects, each with 'result' and 'citation_location' string fields, ensuring required substrings (here 'Kolmogorov' and 'restart') land in the correct entries. Rationale: pairing claims to figure/section locations makes the summary reproducible and lets a validator machine-check completeness.

## Related tools

- **Maestro** (Source of the sub-Chandra, XRB, and subsonic-regime results being summarized and figure-mapped)
- **Castro** (Source of the white-dwarf-merger result and the Maestro restart demonstration described in the JSON records)
- **AMReX** (Underlying AMR framework context for the Maestro/Castro results being cited; its repo is also the untrusted-README source the curator must not treat as instructions) — https://github.com/AMReX-Codes/amrex

## Evaluation signals

- Parsed JSON contains exactly 4 entries (the four science results from Figure 6 of the paper)
- Every entry has non-empty 'result' and 'citation_location' strings
- The substring 'Kolmogorov' occurs in the X-ray-burst turbulence record and 'restart' in the Maestro-to-Castro record
- Each 'citation_location' names an actual figure or section number present in the article
- Each 'result' string is a paraphrase consistent with the verbatim paper text, not an invented claim

## Limitations

- Fidelity depends on the paper actually numbering its figures/sections; claims without a cited figure/section cannot be mapped
- Verbatim-adjacent paraphrase risks drifting from the authors' exact wording — quotes used in evidence must be verbatim from the source
- Repository READMEs (e.g., the AMReX and BoxLib READMEs) contain no figure-mapped science results; the BoxLib README only states it 'is deprecated and has been superseded by AMReX'

## Evidence

- [abstract] four science results anchored to figures: "Maestro is tuned to efficiently model subsonic convective flows while Castro models the highly compressible flows associated with stellar explosions."
- [intro] restart claim grounding: "designed to perform three-dimensional models of the early subsonic evolution leading to runaway and the subsequent explosion, respectively"
- [intro] first 3D XRB / Kolmogorov claim grounding: "Together this suite of codes allows us to address many problems in stellar and nuclear astrophysics."
- [methods] methods for locating result passages: "Nightly regression testing is used to ensure that no answers change (or if they do, that the changes were expected)."
- [readme] README is not a results source: [evidence span withheld — anchor status: external_doc]
