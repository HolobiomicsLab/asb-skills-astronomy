---
name: cross-backend-consistency-comparison
description: Use when a GAMBIT observable can be computed by multiple interchangeable
  backends and you need to verify that different external codes agree on the same
  pinned model point, rather than relying on a single backend's output.
when_to_use_negative:
- Only one backend implementing the observable is available or buildable — no cross-check
  is possible
- The goal is a global statistical scan over parameter space rather than a fixed benchmark-point
  code comparison
- The observable is computed natively by a GAMBIT module (e.g. DarkBit's own gamma-ray
  yield) with no external backend alternative
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: GAMBIT
  role: Framework whose backend system routes the pinned spectrum and couplings to
    each external code
  repo: github.com/GambitBSM/gambit
- name: DarkBit
  role: Module computing the dark matter observables (relic density) requested in
    the YAML scan
  repo: github.com/GambitBSM/gambit_1.5
- name: DarkSUSY
  role: First backend used to compute the relic density for the shared benchmark point
- name: micrOMEGAs
  role: Second backend (v4.1) evaluating the identical benchmark point for comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:2c20fd0debbd4a05d24b50e17a10aa9c4ca8ccb29d024ff80eb547d26a8cae54
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DarkBit
  - DarkSUSY
  - GAMBIT
  - micrOMEGAs
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cross-backend-consistency-comparison

## Summary

Compute the same observable (e.g. the dark matter relic density for a shared MSSM7 benchmark point) through two alternative GAMBIT backends — DarkSUSY and micrOMEGAs 4.1 — and quantify their relative difference as a cross-code consistency check, mirroring DarkBit's quantitative comparison of the two main dark matter codes.

## When to use

Use when a GAMBIT observable can be computed by multiple interchangeable backends and you need to verify that different external codes agree on the same pinned model point, rather than relying on a single backend's output.

## When NOT to use

- Only one backend implementing the observable is available or buildable — no cross-check is possible
- The goal is a global statistical scan over parameter space rather than a fixed benchmark-point code comparison
- The observable is computed natively by a GAMBIT module (e.g. DarkBit's own gamma-ray yield) with no external backend alternative

## Inputs

- GAMBIT source (git repository GambitBSM/gambit_1.5 or tarball)
- YAML scan input file pinning the shared MSSM7 benchmark parameter point and requesting the relic density via DarkBit
- Compiled external backends: DarkSUSY and micrOMEGAs 4.1

## Outputs

- relic_density_comparison.csv with one row per backend and a quantified relative difference
- Per-backend relic density values for the same benchmark point

## How to apply

Build GAMBIT from the retrieved source with CMake, then compile both external dark matter backends with 'make -jn backends' and confirm installation via 'gambit backends'. Write a YAML input file (e.g. modeled on yaml_files/MSSM7.yaml) that pins a shared benchmark parameter point — here MSSM7 — and requests the relic density observable through DarkBit, using separate condition/routing entries so the identical spectrum and couplings are passed first to the DarkSUSY backend and then to micrOMEGAs 4.1. Run 'gambit -f <yaml>' once per backend configuration, recording the relic density value each time. Tabulate the two results side by side, compute the relative difference between them, and write the comparison table to relic_density_comparison.csv. The rationale is GAMBIT's backend system exists precisely to interface external codes, so the same model point can be routed through either code and checked against the paper's reported quantitative DarkSUSY-vs-micrOMEGAs comparison.

## Related tools

- **GAMBIT** (Framework whose backend system routes the pinned spectrum and couplings to each external code) — github.com/GambitBSM/gambit
- **DarkBit** (Module computing the dark matter observables (relic density) requested in the YAML scan) — github.com/GambitBSM/gambit_1.5
- **DarkSUSY** (First backend used to compute the relic density for the shared benchmark point)
- **micrOMEGAs** (Second backend (v4.1) evaluating the identical benchmark point for comparison)

## Examples

```
make -j4 backends && gambit backends && gambit -f yaml_files/MSSM7.yaml
```

## Evaluation signals

- Both backends return finite, positive relic density values for the identical benchmark point
- relic_density_comparison.csv contains exactly one row per backend with the relative difference quantified
- 'gambit backends' lists both DarkSUSY and micrOMEGAs as installed before running
- Relative difference between the two codes is of a magnitude consistent with the paper's reported quantitative DarkSUSY vs micrOMEGAs comparison
- Both runs consume the same pinned spectrum and couplings (same MSSM7 point), not independently generated spectra

## Limitations

- Agreement between the two codes is a consistency check, not validation against data — both could share systematic biases
- Backend versions matter (micrOMEGAs 4.1 is the version referenced); different versions may change results
- cmake may fail to find some dependencies on some systems without guidance, per the README build notes
- Building the entirety of GAMBIT is resource-intensive (the README notes the full build requires substantial resources)

## Evidence

- [abstract] a quantitative comparison between two of the main dark matter codes ( DarkSUSY and micrOMEGAs ): "a quantitative comparison between two of the main dark matter codes ( DarkSUSY and micrOMEGAs )"
- [abstract] along with interfaces to DarkSUSY, micrOMEGAs, and the neutrino telescope likelihood package nulike: "along with interfaces to DarkSUSY , micrOMEGAs , and the neutrino telescope likelihood package nulike"
- [intro] a flexible system for interfacing external codes (the "backend" system): "a flexible system for interfacing external codes (the "backend" system)"
- [methods] Verify installed backends and scanners: "gambit backends
  gambit scanners"
- [methods] Run GAMBIT scan via yaml input file: "gambit -f yaml_files/MSSM7.yaml"
- [readme] MicrOmegas citation pinning version 4.1: "G. Belanger, F. Boudjema, A. Pukhov, A. Semenov, micrOMEGAs4.1: Two dark matter candidates, Comp. Phys. Comm. 192 (2015) 322, arXiv:1407.6129"
