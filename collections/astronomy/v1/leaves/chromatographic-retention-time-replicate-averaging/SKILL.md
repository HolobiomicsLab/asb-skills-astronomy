---
name: chromatographic-retention-time-replicate-averaging
description: Use when capillary GC retention times for reference/relative vapour pressure
  determination (e.g. tetraorganostannanes on an SE-30 column, 40–120°C) are collected
  as multiple replicate runs and must be averaged and hold-up corrected.
when_to_use_negative:
- When only single-run retention data are available and replicate variability cannot
  be averaged out — the method presumes multiple separate runs per compound per temperature
- When a different stationary phase or temperature programme is used whose hold-up
  behaviour cannot be captured by a single natural-gas retention-time subtraction
- When the analysis goal is peak identification or quantification rather than retention-based
  vapour pressure determination
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0153
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Instrument used for isothermal determination of retention times from organostannane
    mixtures
- name: FID detector
  role: Detection of eluted organostannanes during replicate retention-time runs
- name: Chromatopac C-R6A integrator
  role: Interface recording retention times from the GC-15A FID signal
- name: SE-30 capillary column
  role: 2 m × 0.55 mm ID, 1.2 µm film Alltech column providing the stationary phase
    for retention-time measurement
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:8798083608757f6a36258fb6523d559e2c922b5e4e6774718b9b4906ae4149c2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chromatopac C-R6A integrator
  - FID detector
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Chromatographic retention-time replicate averaging

## Summary

Averages isothermal GC retention times over replicate runs (5 runs per organostannane; 3 for Allyl4Sn) and corrects them by subtracting natural-gas hold-up retention before vapour pressure calculation.

## When to use

Use when capillary GC retention times for reference/relative vapour pressure determination (e.g. tetraorganostannanes on an SE-30 column, 40–120°C) are collected as multiple replicate runs and must be averaged and hold-up corrected.

## When NOT to use

- When only single-run retention data are available and replicate variability cannot be averaged out — the method presumes multiple separate runs per compound per temperature
- When a different stationary phase or temperature programme is used whose hold-up behaviour cannot be captured by a single natural-gas retention-time subtraction
- When the analysis goal is peak identification or quantification rather than retention-based vapour pressure determination

## Inputs

- Raw isothermal GC retention times from replicate runs (5 per compound, 3 for Allyl4Sn) at 40, 60, 80, 100, 120°C
- Retention time of a natural gas sample (hold-up reference) at each temperature
- 1.0% w/v organostannane solutions in octane

## Outputs

- Corrected mean retention times per tetraorganostannane per temperature (Table 1-style table)
- Averaged retention-time dataset suitable for vapour pressure determination via a reference compound

## How to apply

Inject 1.0% w/v solutions of the organostannanes in octane on a Shimadzu GC-15A with FID detector (SE-30 capillary column, 2 m × 0.55 mm ID, 1.2 µm film) isothermally at 40, 60, 80, 100 and 120°C. Record raw retention times for each of 5 separate runs (use 3 runs only for Allyl4Sn) and compute the arithmetic mean per compound per temperature. Correct each mean by subtracting the retention time of a natural gas sample, which removes the column dead-time/hold-up contribution so only analyte–stationary phase interaction remains. Feed the corrected mean retention times into the relative retention vapour-pressure calculation with Me4Sn as the reference compound; replicating the measurement reduces injection and integration (Chromatopac C-R6A) variability before regression.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Instrument used for isothermal determination of retention times from organostannane mixtures)
- **FID detector** (Detection of eluted organostannanes during replicate retention-time runs)
- **Chromatopac C-R6A integrator** (Interface recording retention times from the GC-15A FID signal)
- **SE-30 capillary column** (2 m × 0.55 mm ID, 1.2 µm film Alltech column providing the stationary phase for retention-time measurement)

## Examples

```
means = df.groupby(['compound','temperature_C'])['retention_time_raw'].mean(); df['retention_time_corrected'] = means - natural_gas_rt
```

## Evaluation signals

- All vapour-pressure retention times are means of 5 separate runs (mean of 3 for Allyl4Sn) — check replicate count before averaging
- Each reported retention time equals raw mean minus natural-gas retention time at the same isothermal temperature (hold-up corrected)
- Corrected mean retention times cover the full 40–120°C range in 20°C increments for each tetraorganostannane
- Resulting corrected times (e.g. for Me4Sn) reproduce the published Clausius–Clapeyron reference fit (slope ≈ -1.66e3, intercept ≈ 6.65, r ≈ 1.00, ΔHv ≈ 31.28 kJ/mol)

## Limitations

- A C-87 stationary-phase capillary column (identified as most satisfactory by Bidleman) was unavailable, so SE-30 with relatively low McReynold's constants was used instead
- Allyl4Sn retention times relied on only 3 replicate runs rather than 5, reducing averaging power
- Tetra-n-butylstannane was the largest compound whose vapour pressure could be determined under these experimental conditions

## Evidence
