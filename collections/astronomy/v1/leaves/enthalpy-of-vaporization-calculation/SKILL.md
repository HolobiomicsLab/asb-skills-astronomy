---
name: enthalpy-of-vaporization-calculation
description: Use when you have corrected isothermal GC retention times for a compound
  series over a temperature range plus one reference compound with known vapour pressure-temperature
  data, and need ΔHv/ΔHv-Tb values.
when_to_use_negative:
- For tetraalkenylstannanes, where current hydrocarbon vapour pressure estimation
  methods proved less successful than for tetraalkylstannanes
- When no boiling point data are available for the compounds and entropy of vaporization
  is required — lack of appropriate Tb data is an explicit limitation in this study
- For triorganostannanes, where published vapour pressure data are sparse or vary
  considerably, so applicability of the relationship is uncertain
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Instrument used for isothermal measurement of retention times from which vapour
    pressures and ΔHv are derived
- name: FID detector
  role: Flame ionization detector coupled to the GC-15A for detecting organostannane
    peaks
- name: Chromatopac C-R6A integrator
  role: Integrator interfaced to the GC for recording raw retention times
- name: SE-30 capillary column
  role: 2 m × 0.55 mm ID, 1.2 µm film column (Alltech) used because the C-87 stationary-phase
    column was unavailable; SE-30 has relatively low McReynold's constants
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:a58f864a58285bdc56964e5a988f51deec4b74ea1da9dd47064ee35ec1758438
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chromatopac C-R6A integrator
  - FID detector
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# enthalpy-of-vaporization-calculation

## Summary

Derives enthalpies (and, where boiling points exist, entropies) of vaporization for tetraorganostannanes by regressing log retention ratios against 1/T from isothermal GC data, anchored to a reference compound (Me4Sn, ΔHv = 31.28 kJ/mol).

## When to use

Use when you have corrected isothermal GC retention times for a compound series over a temperature range plus one reference compound with known vapour pressure-temperature data, and need ΔHv/ΔHv-Tb values.

## When NOT to use

- For tetraalkenylstannanes, where current hydrocarbon vapour pressure estimation methods proved less successful than for tetraalkylstannanes
- When no boiling point data are available for the compounds and entropy of vaporization is required — lack of appropriate Tb data is an explicit limitation in this study
- For triorganostannanes, where published vapour pressure data are sparse or vary considerably, so applicability of the relationship is uncertain

## Inputs

- Corrected mean retention times for tetraorganostannanes at 40–120°C in 20°C increments (Table 1)
- Natural-gas dead-time retention times for dead-time correction
- Reference compound identity and vapour pressure-temperature data (Me4Sn, ΔHv = 31.28 kJ/mol)
- Boiling point (Tb) values where available

## Outputs

- Per-compound enthalpy of vaporization (ΔHv, kJ/mol) values spanning 31.3–61.3 kJ/mol (Table 4)
- Per-compound entropy of vaporization (ΔHv/Tb, J K-1 mol-1) populated only where Tb is available (89.1–93.4 J K-1 mol-1)
- Ordered CSV of ΔHv and ΔHv/Tb values (vaporization_enthalpies.csv)

## How to apply

Start from corrected mean retention times (raw minus natural-gas dead time, averaged over 5 runs; 3 for Allyl4Sn) measured isothermally from 40°C to 120°C in 20°C increments on the SE-30 capillary column. For each compound and temperature, compute the retention ratio relative to the reference compound tetramethylstannane (Me4Sn). Regress ln(retention ratio) against 1/T (Kelvin) for each compound and convert the slope to an enthalpy of vaporization anchored to the Me4Sn reference value of ΔHv = 31.28 kJ/mol, yielding values in the 31.3–61.3 kJ/mol range (Table 4). Where boiling point data are available, divide each recomputed ΔHv by Tb to obtain the entropy of vaporization (ΔHv/Tb, reported as 89.1–93.4 J K-1 mol-1). Assemble per-compound values in the Table 4 order: Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Instrument used for isothermal measurement of retention times from which vapour pressures and ΔHv are derived)
- **FID detector** (Flame ionization detector coupled to the GC-15A for detecting organostannane peaks)
- **Chromatopac C-R6A integrator** (Integrator interfaced to the GC for recording raw retention times)
- **SE-30 capillary column** (2 m × 0.55 mm ID, 1.2 µm film column (Alltech) used because the C-87 stationary-phase column was unavailable; SE-30 has relatively low McReynold's constants)

## Examples

```
import numpy as np; slope = np.polyfit(1/T_K, np.log(t_comp/t_Me4Sn), 1)[0]; dHv_kJ = 31.28 - slope*8.314/1000
```

## Evaluation signals

- Me4Sn recomputed ΔHv equals the reference anchor ≈ 31.3 kJ/mol, confirming the anchoring step
- All recomputed ΔHv values fall within the reported 31.3–61.3 kJ/mol range of Table 4 (n-Bu4Sn = 61.3 kJ/mol at the top)
- ΔHv/Tb (89.1–93.4 J K-1 mol-1) is populated only for compounds with available boiling points and empty otherwise
- ln(retention ratio) vs 1/T regressions are linear over the 40–120°C series for each compound
- Output ordering matches Table 4: Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn

## Limitations

- Tetra-n-butylstannane was the largest compound whose vapour pressure could be determined under the experimental conditions — larger tetraorganostannanes cannot be handled
- A C-87 stationary phase capillary column (identified as most satisfactory by Bidleman) was unavailable, so SE-30 was substituted
- Lack of appropriate boiling point data prevents computing entropies of vaporization for all compounds and limits assessment of estimation-method adequacy for tetraalkenylstannanes
- Vapour pressures at 20°C were obtained by extrapolation rather than direct measurement

## Evidence
