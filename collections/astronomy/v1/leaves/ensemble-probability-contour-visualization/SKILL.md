---
name: ensemble-probability-contour-visualization
description: Use when standardized OSPREI output files exist from a single run or
  ensemble and need to be converted into profile figures, histograms, and ensemble
  probability contour plots with optional OMNI data overlays.
when_to_use_negative:
- When only quick-look figures are needed during testing — the contour and ensemble
  correlation computations should be commented out to save time
- When raw model inputs (e.g., magnetograms or PFSS backgrounds) have not yet been
  processed into a completed OSPREI run — there are no standardized output files to
  process
- When fine-tuning inputs to achieve a best fit is the goal — this skill is designed
  around forecasting proof-of-concept runs using real-time-analogous information rather
  than optimized parameter exploration
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: OSPREI
  role: Parent suite whose standardized output files are processed into results objects
    and figures
  repo: https://github.com/ckay314/OSPREI
- name: processOSPREI.py
  role: Reads standardized output files, creates a results object, and generates in
    situ profiles, histograms, and ensemble probability contour and correlation plots
  repo: https://github.com/ckay314/OSPREI
- name: FIDO
  role: Produces per-member in situ magnetic field, velocity, and Kp profiles that
    feed the visualization step; includePUP=True activates the FIDO-SIT sheath
  repo: https://github.com/ckay314/OSPREI
- name: ANTEA (ANTEATR/PARADE)
  role: Supplies the evolved CME ensemble members (speed, size, shape, arrival time)
    that FIDO flies over the synthetic spacecraft
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Provides CME final latitude, longitude, and tilt used when flying each ensemble
    member past the synthetic spacecraft
  repo: https://github.com/ckay314/OSPREI
- name: OMNI data
  role: Observational solar wind and Kp reference data overlaid on synthetic profiles
    via ObsDataFile
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:e92e1e0b02c6a82711e2f7208b7ae5f74cc1a2c2dacfad1c0bfe1022934c2ea6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ANTEA
  - ANTEA (ANTEATR/PARADE)
  - FIDO
  - ForeCAT
  - OMNI data
  - OSPREI
  - processOSPREI.py
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ensemble-probability-contour-visualization

## Summary

Generate user-friendly ensemble figures (in situ profiles with OMNI overlays and ensemble probability contour/correlation plots) from standardized OSPREI output files using processOSPREI.py, turning per-member ForeCAT/ANTEATR/FIDO results into accessible forecasting visualizations.

## When to use

Use when standardized OSPREI output files exist from a single run or ensemble and need to be converted into profile figures, histograms, and ensemble probability contour plots with optional OMNI data overlays.

## When NOT to use

- When only quick-look figures are needed during testing — the contour and ensemble correlation computations should be commented out to save time
- When raw model inputs (e.g., magnetograms or PFSS backgrounds) have not yet been processed into a completed OSPREI run — there are no standardized output files to process
- When fine-tuning inputs to achieve a best fit is the goal — this skill is designed around forecasting proof-of-concept runs using real-time-analogous information rather than optimized parameter exploration

## Inputs

- Standardized OSPREI output files from a single simulation or full ensemble (ForeCAT, ANTEATR/PARADE, FIDO outputs)
- OSPREI input text file (date, time, suffix, models, nRuns) matching the run
- Optional .ens file specifying varied ensemble parameters and ranges (required when nRuns > 1)
- OMNI observational data supplied via ObsDataFile for overlaying in situ profiles

## Outputs

- In situ profile figures (magnetic field, velocity, Kp) with OMNI observations overlaid
- Ensemble probability contour plots
- Ensemble correlation plots
- Histograms and value-versus-distance profile figures
- A results object containing all simulation data in an easily accessible format for custom figures

## How to apply

After running OSPREI.py on an input text file (e.g., with date, time, suffix, models, nRuns>1 plus a matching .ens file for ensembles), invoke processOSPREI.py with the same input file; it reads the standardized output files, builds a results object holding all simulation data, and auto-generates figures (value-versus-distance profiles, histograms, in situ profiles, and ensemble probability contours). For a full FIDO workflow, set includePUP=True to activate the FIDO-SIT sheath, fly the ANTEATR-evolved CME ensemble members over the synthetic spacecraft, and supply observations via ObsDataFile so OMNI data overlay the synthetic B, velocity, and Kp profiles. If only quick-look figures are needed while testing, comment out the contour and ensemble correlation plot blocks, since they require significant computation. Validation compares predicted versus observed: ensemble-mean shock arrival within 3 hr, maximum B of 16±3 nT (CME 1) and 11±1 nT (CME 2), most negative Bz of −11±2 nT and −2±1 nT, and maximum Kp occurring in the sheath.

## Related tools

- **OSPREI** (Parent suite whose standardized output files are processed into results objects and figures) — https://github.com/ckay314/OSPREI
- **processOSPREI.py** (Reads standardized output files, creates a results object, and generates in situ profiles, histograms, and ensemble probability contour and correlation plots) — https://github.com/ckay314/OSPREI
- **FIDO** (Produces per-member in situ magnetic field, velocity, and Kp profiles that feed the visualization step; includePUP=True activates the FIDO-SIT sheath) — https://github.com/ckay314/OSPREI
- **ANTEA (ANTEATR/PARADE)** (Supplies the evolved CME ensemble members (speed, size, shape, arrival time) that FIDO flies over the synthetic spacecraft) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Provides CME final latitude, longitude, and tilt used when flying each ensemble member past the synthetic spacecraft) — https://github.com/ckay314/OSPREI
- **OMNI data** (Observational solar wind and Kp reference data overlaid on synthetic profiles via ObsDataFile)

## Examples

```
python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- Ensemble-mean predicted shock arrival time falls within 3 hr of the observed value for both CMEs
- Maximum B matches 16±3 nT for CME 1 (09 May 2021) and 11±1 nT for CME 2 (22 April 2021)
- Most negative Bz matches −11±2 nT (CME 1) and −2±1 nT (CME 2) within the ensemble profiles
- Maximum Kp occurs in the sheath region for both events
- Contours only appear for full ensemble runs; single-run processing should yield profiles/histograms without ensemble contour blocks

## Limitations

- Contour and ensemble correlation plots require significant computation and may need to be skipped for quick-look figures
- FIDO-SIT cannot predict in situ magnetic field components, only assumes a Parker Spiral orientation for the upstream direction
- Only two CME cases considered; unclear if the limited in situ B agreement (better for the more flux-rope-like CME) is a persistent limitation
- Inputs are not fine-tuned, so these results do not represent the best attainable from the model — parameter space optimization is left as future work

## Evidence

- [readme] processOSPREI reads standardized output files into results and generates ensemble contour/correlation figures: "After running OSPREI, the results can be automatically processed using `processOSPREI.py`. This takes the standardized output files, processes them into a results object"
- [readme] Contour plots are computationally heavy and can be skipped for quick looks: "Others require more significant computation, particularly for the contour plots and ensemble correltation plots, and should potentially be commented out and not run if you want to make quick figures"
- [other] Validation thresholds for arrival time and in situ B/Bz/Kp: [evidence span withheld — anchor status: not_found]
- [intro] Standardized visualizations generated automatically for forecasting accessibility: "Standardized visualizations are automatically generated, providing easily accessible, essential information for space weather forecasting."
- [discussion] FIDO produces the in situ profiles that feed the visualization step: [evidence span withheld — anchor status: not_found]
