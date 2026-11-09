---
name: linear-regression-of-vant-hoff-plots
description: Use when you have corrected GC retention times for a homologous series
  measured isothermally at multiple temperatures, a reference compound with known
  vaporization enthalpy, and need ΔHv/ΔSv per compound.
when_to_use_negative:
- Input is vapour pressure data lacking isothermal, temperature-resolved retention
  times — the regression of ln(retention ratio) vs 1/T cannot be constructed
- No reference compound with known vapour pressure–temperature data (here Me4Sn) is
  available to anchor the slopes
- Estimating vapour pressures of tetraalkenylstannanes, for which current hydrocarbon-based
  estimation methods were less successful
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Generates the isothermal retention-time data (40–120°C) that feed the ln(retention
    ratio) vs 1/T regressions
- name: FID detector
  role: Flame ionization detector used with the GC-15A to acquire organostannane retention
    peaks
- name: Chromatopac C-R6A integrator
  role: Integrates the FID signal to yield raw retention times used in the vapour
    pressure determinations
- name: SE-30 capillary column
  role: 2 m × 0.55 mm ID, 1.2 µm film column (Alltech) on which all isothermal retention
    measurements are made
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:2540f8dd3847b6b0801bc8b133c0c1b8f1854bab1a463a6a74475b01d376b802
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chromatopac C-R6A integrator
  - FID detector
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# linear-regression-of-vant-hoff-plots

## Summary

Derives enthalpies (and entropies) of vaporization for tetraorganostannanes by linear regression of ln(retention ratio vs Me4Sn) against 1/T from isothermal GC retention times, anchoring slopes to the Me4Sn reference ΔHv of 31.28 kJ/mol.

## When to use

Use when you have corrected GC retention times for a homologous series measured isothermally at multiple temperatures, a reference compound with known vaporization enthalpy, and need ΔHv/ΔSv per compound.

## When NOT to use

- Input is vapour pressure data lacking isothermal, temperature-resolved retention times — the regression of ln(retention ratio) vs 1/T cannot be constructed
- No reference compound with known vapour pressure–temperature data (here Me4Sn) is available to anchor the slopes
- Estimating vapour pressures of tetraalkenylstannanes, for which current hydrocarbon-based estimation methods were less successful

## Inputs

- Corrected mean GC retention times per compound per temperature (40–120°C, 20°C increments) from the SE-30 capillary column (Table 1)
- Retention times of the Me4Sn reference compound at the same isothermal temperatures
- Literature boiling points (Tb) for compounds where entropy of vaporization is desired
- Reference anchor value: ΔHv(Me4Sn) = 31.28 kJ/mol

## Outputs

- Per-compound slopes of ln(retention ratio) vs 1/T regressions
- Enthalpies of vaporization (ΔHv, kJ/mol) per tetraorganostannane, expected in the 31.3–61.3 kJ/mol range
- Entropies of vaporization (ΔHv/Tb, J K-1 mol-1) only for compounds with available boiling points, expected in the 89.1–93.4 J K-1 mol-1 range
- vaporization_enthalpies.csv ordered as in Table 4: Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn

## How to apply

Load corrected mean retention times (natural-gas dead-time subtracted, averaged over 5 runs; 3 for Allyl4Sn) measured isothermally from 40°C to 120°C in 20°C increments on the SE-30 capillary column (Table 1). For each compound and each temperature, compute the retention ratio relative to Me4Sn from the corrected retention times. Regress ln(retention ratio) against 1/T in Kelvin for each tetraorganostannane (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) to obtain the slope of the van't Hoff-type relationship. Convert each slope to ΔHv by anchoring to the Me4Sn reference value of 31.28 kJ/mol, which should reproduce the 31.3–61.3 kJ/mol range of Table 4. Where boiling point (Tb) data exist, compute the entropy of vaporization as ΔHv/Tb and leave ΔHv/Tb empty otherwise.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Generates the isothermal retention-time data (40–120°C) that feed the ln(retention ratio) vs 1/T regressions)
- **FID detector** (Flame ionization detector used with the GC-15A to acquire organostannane retention peaks)
- **Chromatopac C-R6A integrator** (Integrates the FID signal to yield raw retention times used in the vapour pressure determinations)
- **SE-30 capillary column** (2 m × 0.55 mm ID, 1.2 µm film column (Alltech) on which all isothermal retention measurements are made)

## Examples

```
slope, _ = np.polyfit(1.0/(T_C + 273.15), np.log(rt_compound/rt_Me4Sn), 1)
```

## Evaluation signals

- Me4Sn recomputed ΔHv ≈ 31.3 kJ/mol, confirming the 31.28 kJ/mol reference anchor is applied correctly
- All recomputed ΔHv values fall within the Table 4 range of 31.3–61.3 kJ/mol (n-Bu4Sn highest at 61.3 kJ/mol)
- ΔHv/Tb is populated only for compounds with available boiling points and lies in the 89.1–93.4 J K-1 mol-1 range
- ln(retention ratio) vs 1/T plots are linear across the 40–120°C series (40, 60, 80, 100, 120°C) for each compound
- Retention ratios are computed from corrected (dead-time-subtracted, 5-run mean; 3-run for Allyl4Sn) retention times, not raw integrator values

## Limitations

- Allyl4Sn retention times are means of only 3 runs (vs 5 for other compounds), reducing precision of its regression
- ΔHv/Tb cannot be reported for compounds lacking boiling point data, and lack of appropriate boiling point data prevents confirming whether estimation methods are adequate for other tetraalkylstannanes
- Tetra-n-butylstannane was the largest compound whose vapour pressure could be determined under the experimental conditions; heavier homologues are out of scope

## Evidence
