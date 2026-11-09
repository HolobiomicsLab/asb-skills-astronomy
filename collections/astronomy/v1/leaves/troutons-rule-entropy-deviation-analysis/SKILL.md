---
name: troutons-rule-entropy-deviation-analysis
description: Use when ΔHvap values and normal boiling points are available for a homologous
  compound series and you need to test whether ΔSvap conforms to Trouton's rule (~90
  J/K/mol) or shows substituent-driven excess entropy.
when_to_use_negative:
- ΔHvap is available but no boiling point exists (e.g. Allyl4Sn, n-Pr4Sn, i-Pr4Sn,
  n-Bu4Sn) — ΔSvap cannot be computed without Tb
- The compounds are solid at room temperature, where vapour pressure requires entropy
  of fusion and melting point conversion rather than simple ΔSvap = ΔHvap/Tb
- The goal is predicting vapour pressures themselves (QSPR vs. molecular weight or
  intrinsic molar volume), not testing entropy-of-vaporization conformity
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Instrument whose isothermal capillary GC vapour-pressure measurements (40–120°C)
    produced the ΔHvap values the Trouton analysis consumes
- name: SE-30 capillary column
  role: 2 m × 0.55 mm ID SE-30 column (1.2 µm film) used to separate tetraorganostannanes
    for vapour-pressure determination underlying the derived entropies
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:e4d7e03c99af0ead59bf5d647f9fceaa810bbcf8788f31f0a5de9f22a4b8908a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# troutons-rule-entropy-deviation-analysis

## Summary

Derives entropies of vaporization (ΔSvap = ΔHvap/Tb) for tetraorganostannanes from measured vapour-pressure enthalpies and boiling points, then tests clustering around the ~90 J/K/mol Trouton's rule reference value.

## When to use

Use when ΔHvap values and normal boiling points are available for a homologous compound series and you need to test whether ΔSvap conforms to Trouton's rule (~90 J/K/mol) or shows substituent-driven excess entropy.

## When NOT to use

- ΔHvap is available but no boiling point exists (e.g. Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) — ΔSvap cannot be computed without Tb
- The compounds are solid at room temperature, where vapour pressure requires entropy of fusion and melting point conversion rather than simple ΔSvap = ΔHvap/Tb
- The goal is predicting vapour pressures themselves (QSPR vs. molecular weight or intrinsic molar volume), not testing entropy-of-vaporization conformity

## Inputs

- Table 4 enthalpies of vaporization (kJ/mol) for seven tetraorganostannanes
- Normal boiling point data (K or °C) for subset of compounds (Me4Sn, Vi4Sn, Et4Sn)
- Compound metadata: number of flexible methylene-like links per tetraorganostannane

## Outputs

- trouton_analysis.csv with per-compound ΔSvap (J/K/mol), deviation from ~90 J/K/mol, and group statistics
- Summary statistics: mean absolute deviation and range of ΔSvap deviations

## How to apply

Load the reproduced Table 4 enthalpies of vaporization (kJ/mol) and available normal boiling points for Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn and n-Bu4Sn with pandas. Convert ΔHvap from kJ/mol to J/mol and compute ΔSvap = ΔHvap/Tb with Tb in Kelvin, using only the three compounds with boiling point data (Me4Sn 89.1, Vi4Sn 93.0, Et4Sn 93.4 J/K/mol). Calculate each compound's deviation from the ~90 J/K/mol Trouton reference plus summary statistics (mean absolute deviation, range). Group compounds by number of flexible methylene-like links and test whether compounds with ≥5 such links show increased ΔSvap, since the underlying ΔHvap data come from vapour pressures measured by capillary GC over 40–120°C. Write the per-compound ΔSvap table and group summary to trouton_analysis.csv.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Instrument whose isothermal capillary GC vapour-pressure measurements (40–120°C) produced the ΔHvap values the Trouton analysis consumes)
- **SE-30 capillary column** (2 m × 0.55 mm ID SE-30 column (1.2 µm film) used to separate tetraorganostannanes for vapour-pressure determination underlying the derived entropies)

## Examples

```
import pandas as pd; df = pd.read_csv('table4_dhvap.csv'); df['dSvap'] = df['dHvap']*1000 / (df['Tb_C']+273.15); df['dev_trouton'] = df['dSvap'] - 90; df.to_csv('trouton_analysis.csv', index=False)
```

## Evaluation signals

- Every compound with an available boiling point (Me4Sn, Vi4Sn, Et4Sn) has a computed ΔSvap in the output table
- Computed ΔSvap values match the reported 89.1, 93.0 and 93.4 J/K/mol within unit-conversion rounding
- Deviations from the ~90 J/K/mol Trouton reference are small (mean absolute deviation of a few J/K/mol), confirming clustering
- Unit consistency check: ΔHvap converted from kJ/mol to J/mol and Tb in Kelvin before division
- Group summary distinguishes compounds with ≥5 flexible methylene-like links and reports their ΔSvap shift

## Limitations

- Only three of the seven tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn) have boiling point data, so ΔSvap and Trouton conformity can only be assessed for that subset
- Lack of appropriate boiling point data prevents confirming whether estimation methods are adequate for other tetraalkylstannanes and tetraalkenylstannanes
- For solid tetraorganostannanes (MP > 25°C), ΔSvap-based treatment requires conversion using entropy of fusion (ΔSf) and melting point
- Vapour pressures at 20°C were obtained by extrapolation from the 40–120°C measurement range, adding uncertainty to derived thermodynamic quantities

## Evidence
