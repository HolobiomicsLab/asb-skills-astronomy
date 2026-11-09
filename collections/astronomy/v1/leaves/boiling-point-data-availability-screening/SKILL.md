---
name: boiling-point-data-availability-screening
description: Use when a dataset contains derived enthalpies of vaporization (e.g.
  from GC-measured vapour pressures) but boiling point coverage is partial, and you
  must identify which compounds qualify for ΔSvap derivation versus which must be
  excluded.
when_to_use_negative:
- All compounds in the dataset have boiling points, so a simple ΔSvap computation
  needs no availability screen
- The analysis target is vapour pressure measurement or QSPR fitting (e.g. lnP vs
  intrinsic molar volume) and does not involve entropies of vaporization
- Compounds are solid at room temperature (MP > 25°C), where the paper notes Equation
  (9) data additionally require entropy of fusion (ΔSf) and melting point for solid
  vapour pressure conversion
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Source instrument whose isothermal capillary GC measurements (40–120°C) produced
    the vapour pressures from which ΔHvap, and conditionally ΔSvap, were derived
- name: SE-30 capillary column
  role: 2m x 0.55mm ID, 1.2µm film column fitted to the GC-15A for the retention-time
    measurements underlying the enthalpy/entropy derivations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:fc0553247041a0aa4147bc9c6838d9926dbe9d2280edc3960b05fa5d0b752a5e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# boiling-point-data-availability-screening

## Summary

A screening step that checks which compounds in a physicochemical dataset have usable normal boiling point (Tb) data before deriving entropies of vaporization (ΔSvap = ΔHvap/Tb), since ΔSvap can only be computed where Tb exists. In the Hawker 1992 tetraorganostannane study this gate determined that only Me4Sn, Vi4Sn and Et4Sn received ΔSvap values.

## When to use

Use when a dataset contains derived enthalpies of vaporization (e.g. from GC-measured vapour pressures) but boiling point coverage is partial, and you must identify which compounds qualify for ΔSvap derivation versus which must be excluded.

## When NOT to use

- All compounds in the dataset have boiling points, so a simple ΔSvap computation needs no availability screen
- The analysis target is vapour pressure measurement or QSPR fitting (e.g. lnP vs intrinsic molar volume) and does not involve entropies of vaporization
- Compounds are solid at room temperature (MP > 25°C), where the paper notes Equation (9) data additionally require entropy of fusion (ΔSf) and melting point for solid vapour pressure conversion

## Inputs

- Table of derived enthalpies of vaporization (ΔHvap, kJ/mol) for tetraorganostannanes (CSV/pandas DataFrame)
- Normal boiling point (Tb, K or °C) data compiled from literature for each compound

## Outputs

- Per-compound availability table listing ΔHvap, Tb availability flag, and ΔSvap (J/K/mol) for Tb-available compounds
- Count/list of excluded compounds lacking boiling point data

## How to apply

Load the derived ΔHvap table (e.g. reproduced Table 4 for the seven tetraorganostannanes Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) alongside available normal boiling points with pandas, and mark each compound as Tb-available or Tb-missing. Compute ΔSvap = ΔHvap/Tb only for Tb-available compounds, converting enthalpies from kJ/mol to J/K/mol consistently, and record missing-Tb compounds explicitly rather than silently dropping them. This gate matters because the paper notes that uncertainty over estimation-method adequacy persists 'because of lack of appropriate boiling point data'; here only three of seven compounds (Me4Sn, Vi4Sn, Et4Sn) had ΔSvap reported. The screened output then feeds downstream checks such as comparison against the ~90 J/K/mol Trouton's rule reference.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Source instrument whose isothermal capillary GC measurements (40–120°C) produced the vapour pressures from which ΔHvap, and conditionally ΔSvap, were derived)
- **SE-30 capillary column** (2m x 0.55mm ID, 1.2µm film column fitted to the GC-15A for the retention-time measurements underlying the enthalpy/entropy derivations)

## Examples

```
df = pd.read_csv('table4_dhvap.csv'); ok = df[df['Tb_K'].notna()].copy(); ok['dSvap'] = ok['dHvap_kJmol']*1000/ok['Tb_K']
```

## Evaluation signals

- Every compound flagged Tb-available has a computed ΔSvap; every Tb-missing compound has no ΔSvap value and appears in an explicit exclusion list
- ΔSvap units are J/K/mol, with ΔHvap converted from kJ/mol (factor 1000) and Tb in kelvin
- For the Hawker dataset the screen should pass exactly Me4Sn (89.1), Vi4Sn (93.0) and Et4Sn (93.4) J/K/mol and exclude Allyl4Sn, n-Pr4Sn, i-Pr4Sn and n-Bu4Sn
- The number of screened-in compounds matches the paper's statement that ΔSvap was derived only 'where appropriate boiling point information was available'

## Limitations

- Lack of appropriate boiling point data leaves it uncertain whether vapour-pressure estimation methods are adequate for other tetraalkylstannanes and tetraalkenylstannanes
- ΔSvap coverage is sparse: only 3 of 7 compounds (Me4Sn, Vi4Sn, Et4Sn) qualify, so group statistics (e.g. Trouton deviations, ≥5-flexible-link comparisons) rest on very few points
- Vapour pressures at 20°C were obtained by extrapolation from the 40–120°C measurement range, so derived quantities inherit extrapolation uncertainty

## Evidence

- [abstract] Conditional ΔSvap derivation is gated on boiling point availability: "Enthalpies of vaporization, and where appropriate boiling point information was available, entropies of vaporization for the tetraorganostannanes were also derived."
