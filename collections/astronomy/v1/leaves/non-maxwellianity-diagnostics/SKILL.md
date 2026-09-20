---
name: non-maxwellianity-diagnostics
description: 'Use when judging whether Vlasiator simulation output (VLSV files) contains
  kinetic ion physics: compute a non-Maxwellianity metric on downstream shock ion
  distributions and compare magnetic compression ratios across grid resolutions.'
when_to_use_negative:
- 'The simulation output is purely fluid (MHD) rather than hybrid-Vlasov: ion velocity
  distribution functions are not available, so a non-Maxwellianity metric cannot be
  defined'
- The analysis target is an equilibrium, collisional, or thermally Maxwellian plasma
  region with no shock — departures from fluid theory are not the question under study
- You only need global magnetosphere morphology rather than resolved ion-kinetic structure
  at bow-shock-like scales
tools:
- name: Vlasiator
  role: Hybrid-Vlasov solver used to run the 1D collisionless-shock benchmark at 1000
    km and 200 km resolutions and write the VLSV output files analysed here
  repo: https://github.com/fmihpc/vlasiator
- name: Analysator
  role: Python toolset used to read and analyse the VLSV output files, including VlsvReader
    access and velocity distribution function plotting
  repo: https://github.com/fmihpc/analysator
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:6372afe3be14cad87f68919f7a636631cdcdb7369f9b3285cd1cd6a0041291b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/vlasiator
  tools:
  - Analysator
  - Vlasiator
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# non-maxwellianity-diagnostics

## Summary

Quantifies departures of ion velocity distribution functions from Maxwellian equilibrium in Vlasiator hybrid-Vlasov shock simulations, demonstrating that the model captures kinetic (non-fluid) physics rather than behaving like MHD forerunners.

## When to use

Use when judging whether Vlasiator simulation output (VLSV files) contains kinetic ion physics: compute a non-Maxwellianity metric on downstream shock ion distributions and compare magnetic compression ratios across grid resolutions.

## When NOT to use

- The simulation output is purely fluid (MHD) rather than hybrid-Vlasov: ion velocity distribution functions are not available, so a non-Maxwellianity metric cannot be defined
- The analysis target is an equilibrium, collisional, or thermally Maxwellian plasma region with no shock — departures from fluid theory are not the question under study
- You only need global magnetosphere morphology rather than resolved ion-kinetic structure at bow-shock-like scales

## Inputs

- Vlasiator VLSV output files from the 1D collisionless-shock run at 1000 km grid resolution
- Vlasiator VLSV output files from the identical shock setup re-run at 200 km resolution
- Simulation configuration for the Pfau-Kempf et al. (2018) bow-shock-like plasma conditions

## Outputs

- Magnetic compression ratio of the downstream shock region for each resolution
- Non-Maxwellianity metric computed on the downstream ion velocity distribution function
- shock_comparison_table.csv assembling the two-resolution results

## How to apply

Configure the 1D collisionless-shock benchmark with terrestrial bow-shock-like plasma conditions following the Pfau-Kempf et al. (2018) setup, run it in Vlasiator at a coarse 1000 km grid resolution, then re-run the identical setup at 200 km. Load the VLSV output files with Analysator's VlsvReader and analyse the downstream shock region, extracting the magnetic compression ratio and computing a non-Maxwellianity metric for the downstream ion distribution (ions are stored as velocity distribution functions, electrons as MHD fluid). Assemble both runs into shock_comparison_table.csv: the compression ratio should lie near the ~4x intensification reported for the terrestrial bow shock, and the 200 km run should show markedly improved physical detail, with distribution departures from fluid theory consistent with a kinetic description.

## Related tools

- **Vlasiator** (Hybrid-Vlasov solver used to run the 1D collisionless-shock benchmark at 1000 km and 200 km resolutions and write the VLSV output files analysed here) — https://github.com/fmihpc/vlasiator
- **Analysator** (Python toolset used to read and analyse the VLSV output files, including VlsvReader access and velocity distribution function plotting) — https://github.com/fmihpc/analysator

## Examples

```
export PTNONINTERACTIVE=1; pip install --editable ./analysator; python -c "import analysator as pt; r=pt.vlsvfile.VlsvReader('bulk.0001000.vlsv')"
```

## Evaluation signals

- Magnetic compression ratio extracted from the downstream region falls near the ~4x intensification reported for the terrestrial bow shock
- All required fields are present in shock_comparison_table.csv and the analysis script runs to completion
- The 200 km run shows improved physical detail relative to the 1000 km run
- Downstream ion distributions depart from fluid theory in a way that expert review confirms is consistent with a kinetic description
- Results depart from fluid (MHD) behaviour even at the coarse 1000 km resolution — if they match fluid theory, the metric or setup is wrong

## Limitations

- Vlasiator's multi-dimensional ion-scale approach implies immense computational challenges; the 200 km refinement that improves physical detail is correspondingly more expensive than the 1000 km run
- Coarse 1000 km resolution captures the kinetic departure from fluid theory but with limited physical detail compared with 200 km
- Ions are kinetic distribution functions while electrons remain a magnetohydrodynamic fluid, so electron-scale non-Maxwellian features are outside the model's scope

## Evidence

- [readme] Hybrid-Vlasov model captures ion kinetics, not fluid physics: [evidence span withheld — anchor status: external_doc]
- [readme] Analysator is the designated reader for VLSV output: [evidence span withheld — anchor status: not_found]
- [abstract] Vlasiator is the target simulation platform for this diagnostic: "A particular application of the Vlasov system is Vlasiator, the world’s first global hybrid-Vlasov simulation for the Earth’s magnetic domain, the magnetosphere."
- [intro] Kinetic results are validated against observations/theory rather than assumed: "When tested against observations, numerical models can strengthen or invalidate existing theories and quantify the degree to which the theories have to be improved."
- [results] The non-Maxwellianity metric distinguishes Vlasov results from MHD forerunners: [evidence span withheld — anchor status: not_found]
