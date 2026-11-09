---
name: entropy-of-vaporization-derivation-from-boiling-points
description: Use when you have per-compound enthalpies of vaporization from GC vapour-pressure
  data plus literature boiling points (Tb), and need ΔSv = ΔHv/Tb for Table 4-style
  comparisons across organostannanes and hydrocarbons.
when_to_use_negative:
- The compound has no literature boiling point — the article derives entropies only
  'where appropriate boiling point information was available', so ΔHv/Tb cannot be
  populated and should be left blank rather than estimated.
- The analyte is a tetraalkenylstannane (e.g. Vi4Sn chemistry checks fail) and you
  intend to substitute hydrocarbon-based vapour-pressure estimation methods — the
  paper states these 'proved satisfactory for tetraalkylstannanes, but less successful
  for tetraalkenylstannanes'.
- The compound is solid at room temperature (MP > 25°C) and you need solid-state vapour
  pressure — Equation (9) data require conversion using entropy of fusion (ΔSf) and
  melting point, which is beyond this skill's scope.
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Source instrument for the isothermal retention-time measurements (40–120°C,
    SE-30 capillary column) whose temperature series feeds the van't Hoff regressions
    behind each ΔHv used in ΔHv/Tb.
- name: FID detector
  role: Detection of organostannane peaks in 1.0% w/v octane mixtures, enabling retention-time
    extraction for each run.
- name: Chromatopac C-R6A integrator
  role: Integrator recording raw retention times from the FID, which are then dead-time-corrected
    and averaged (5 runs; 3 for Allyl4Sn).
- name: SE-30 capillary column
  role: 2 m × 0.55 mm ID, 1.2 µm film Alltech SE-30 stationary phase used because
    C-87 was unavailable; provides the isothermal retention data set for the reference-compound
    calculation.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:f8b7702a5318e76fe6008fb7be87bcf3b3f4e6c3e3d8a0879f7b0408ea224c30
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Chromatopac C-R6A integrator
  - FID detector
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# entropy-of-vaporization-derivation-from-boiling-points

## Summary

Derives entropies of vaporization for tetraorganostannanes by dividing each compound's recomputed enthalpy of vaporization (from GC retention-ratio van't Hoff regressions anchored to Me4Sn at 31.28 kJ/mol) by its boiling point, yielding the ΔHv/Tb values (89.1–93.4 J K⁻¹ mol⁻¹) reported in Table 4.

## When to use

Use when you have per-compound enthalpies of vaporization from GC vapour-pressure data plus literature boiling points (Tb), and need ΔSv = ΔHv/Tb for Table 4-style comparisons across organostannanes and hydrocarbons.

## When NOT to use

- The compound has no literature boiling point — the article derives entropies only 'where appropriate boiling point information was available', so ΔHv/Tb cannot be populated and should be left blank rather than estimated.
- The analyte is a tetraalkenylstannane (e.g. Vi4Sn chemistry checks fail) and you intend to substitute hydrocarbon-based vapour-pressure estimation methods — the paper states these 'proved satisfactory for tetraalkylstannanes, but less successful for tetraalkenylstannanes'.
- The compound is solid at room temperature (MP > 25°C) and you need solid-state vapour pressure — Equation (9) data require conversion using entropy of fusion (ΔSf) and melting point, which is beyond this skill's scope.

## Inputs

- Corrected mean retention times for tetraorganostannanes at 40–120°C in 20°C increments (Table 1; means of 5 runs, 3 for Allyl4Sn, dead-time-corrected against natural gas)
- Me4Sn reference enthalpy of vaporization (ΔHv = 31.28 kJ/mol) and its vapour pressure–temperature data
- Literature boiling point (Tb) values for each tetraorganostannane
- Retention-ratio van't Hoff regression slopes (ln ratio vs 1/T) for each compound

## Outputs

- Per-compound enthalpies of vaporization ΔHv (kJ/mol, 31.3–61.3 range) as in Table 4
- Per-compound entropies of vaporization ΔHv/Tb (J K⁻¹ mol⁻¹, 89.1–93.4 range) populated only for compounds with available boiling points
- vaporization_enthalpies.csv ordered as in Table 4 (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn)

## How to apply

First compute each tetraorganostannane's enthalpy of vaporization: take the corrected mean retention times from the SE-30 capillary column isothermal runs (40–120°C in 20°C increments), form the retention ratio against Me4Sn, regress ln(retention ratio) versus 1/T (Kelvin), and convert each slope to ΔHv anchored to the Me4Sn reference value of ΔHv = 31.28 kJ/mol (Me4Sn must therefore come out at ≈31.3 kJ/mol). Then, for each compound, retrieve its boiling point Tb; entropies of vaporization are only derived 'where appropriate boiling point information was available' — the Me4Sn-to-n-Bu4Sn series yields ΔSv values of 89.1–93.4 J K⁻¹ mol⁻¹. Order the output as in Table 4 (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) and leave ΔHv/Tb empty for compounds lacking Tb data. The rationale is that ΔSv at the boiling point is the ratio ΔHv/Tb, and the paper tests whether this quantity follows trends shared with hydrocarbons versus molecular weight or intrinsic molar volume.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Source instrument for the isothermal retention-time measurements (40–120°C, SE-30 capillary column) whose temperature series feeds the van't Hoff regressions behind each ΔHv used in ΔHv/Tb.)
- **FID detector** (Detection of organostannane peaks in 1.0% w/v octane mixtures, enabling retention-time extraction for each run.)
- **Chromatopac C-R6A integrator** (Integrator recording raw retention times from the FID, which are then dead-time-corrected and averaged (5 runs; 3 for Allyl4Sn).)
- **SE-30 capillary column** (2 m × 0.55 mm ID, 1.2 µm film Alltech SE-30 stationary phase used because C-87 was unavailable; provides the isothermal retention data set for the reference-compound calculation.)

## Examples

```
from scipy.stats import linregress; s,_,_,_,_ = linregress(1/T_K, ln_ratio); dHv = -s*8.314/1000; dSv = dHv*1000/Tb if Tb else None
```

## Evaluation signals

- Me4Sn recomputed ΔHv must come out ≈31.3 kJ/mol, confirming the reference anchor of 31.28 kJ/mol was applied correctly.
- All recomputed ΔHv values must fall within the 31.3–61.3 kJ/mol Table 4 range (maximum 61.3 kJ/mol for n-Bu4Sn).
- Every populated ΔHv/Tb value must lie in the 89.1–93.4 J K⁻¹ mol⁻¹ range reported in Table 4.
- ΔHv/Tb cells must be empty for exactly those compounds lacking boiling point data; no imputed values are acceptable.
- Retention ratios must be computed from corrected mean retention times (natural-gas dead-time subtracted; means of 5 runs, 3 for Allyl4Sn), and regression must use 1/T in Kelvin over the 40–120°C series.

## Limitations

- Entropies of vaporization exist only for the subset of compounds with literature boiling points; missing Tb data also prevents confirming whether estimation methods are adequate for other tetraalkylstannanes and tetraalkenylstannanes.
- Hydrocarbon vapour-pressure estimation methods were less successful for tetraalkenylstannanes, limiting extension of the derived quantities to that class.
- Under the experimental conditions, tetra-n-butylstannane was the largest compound whose vapour pressure could be determined, bounding the usable ΔHv series at n-Bu4Sn.
- A C-87 stationary-phase capillary column (identified as most satisfactory by Bidleman) was unavailable, so SE-30 was substituted — retention data are specific to this column.

## Evidence
