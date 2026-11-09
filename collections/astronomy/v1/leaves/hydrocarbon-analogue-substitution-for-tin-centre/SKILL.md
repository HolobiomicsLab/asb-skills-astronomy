---
name: hydrocarbon-analogue-substitution-for-tin-centre
description: Use when estimating vapour pressures (or other hydrocarbon-derived physicochemical
  properties) of organostannanes at environmentally relevant temperatures (e.g. 20°C)
  where no direct hydrocarbon-based QSPR exists for the Sn-centred compound.
when_to_use_negative:
- 'When molecular-weight-based hydrocarbon relationships are needed: the article states
  ''any existing relationship involving MW for hydrocarbons can not be used with tetraorganostannanes'';
  only intrinsic-molar-volume relationships give a single unique relationship across
  organostannanes and hydrocarbons'
- 'When the target compound is solid at room temperature (MP > 25°C): Equation (9)
  data must be converted to solid vapour pressure using entropy of fusion (ΔSf) and
  melting point, an extra input this skill does not provide'
- 'When the compound is a triorganostannane: little vapour pressure data exists and
  published values vary considerably, so applicability of Equation (9) is uncertain'
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: Shimadzu GC-15A gas chromatograph
  role: Source of the experimental equilibrium vapour pressures (40–120°C, FID detector,
    SE-30 capillary column) against which the hydrocarbon-analogue estimates are validated
- name: SE-30 capillary column
  role: 2 m × 0.55 mm ID, 1.2 µm film column used for the relative GC retention-time
    vapour pressure determinations used as ground truth
- name: Antoine equation
  role: Hydrocarbon-based vapour pressure estimator applied to the carbon-substituted
    analogue, parameterized from the analogue's normal boiling point
- name: Modified Watson correlation
  role: Second hydrocarbon-based estimator applied with the same boiling point and
    molecular-size inputs to cross-check the Antoine baseline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/0045-6535(92)90276-w
    title: Equilibrium vapour pressures of tetraorganostannanes
  source_content_hash: sha256:d7f941e0683b05abe425dabbf9f44a24a4d3c981c07294cb43f9817879baba7c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Antoine equation
  - Modified Watson correlation
  - SE-30 capillary column
  - Shimadzu GC-15A gas chromatograph
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# hydrocarbon-analogue-substitution-for-tin-centre

## Summary

Builds hydrocarbon analogues of tetraorganostannanes by substituting a carbon atom for the central Sn atom, so that hydrocarbon-based vapour pressure estimators (Antoine equation, Modified Watson correlation) can be applied and their QSPR transferability to organometallics tested.

## When to use

Use when estimating vapour pressures (or other hydrocarbon-derived physicochemical properties) of organostannanes at environmentally relevant temperatures (e.g. 20°C) where no direct hydrocarbon-based QSPR exists for the Sn-centred compound.

## When NOT to use

- When molecular-weight-based hydrocarbon relationships are needed: the article states 'any existing relationship involving MW for hydrocarbons can not be used with tetraorganostannanes'; only intrinsic-molar-volume relationships give a single unique relationship across organostannanes and hydrocarbons
- When the target compound is solid at room temperature (MP > 25°C): Equation (9) data must be converted to solid vapour pressure using entropy of fusion (ΔSf) and melting point, an extra input this skill does not provide
- When the compound is a triorganostannane: little vapour pressure data exists and published values vary considerably, so applicability of Equation (9) is uncertain

## Inputs

- List of tetraorganostannanes (Me4Sn, Vi4Sn, Et4Sn, Allyl4Sn, n-Pr4Sn, i-Pr4Sn, n-Bu4Sn) with normal boiling points
- Molecular-size inputs (molecular weight, intrinsic molar volume) for each stannane/hydrocarbon analogue
- GC-measured vapour pressure data for tetraorganostannanes spanning 40–120°C in 20°C increments (means of 5 runs per compound, 3 for Allyl4Sn)
- Experimental 20°C extrapolated vapour pressures (Et4Sn 0.16 kPa; Vi4Sn 0.35 kPa)

## Outputs

- Antoine-equation vapour pressure estimates at 20°C for each hydrocarbon analogue (e.g. Et4Sn 0.15 kPa)
- Modified Watson correlation vapour pressure estimates at 20°C (e.g. Vi4Sn 0.25 kPa)
- vapour_pressure_estimates.csv containing estimated and experimental vapour pressures per compound
- Qualitative transferability verdict: satisfactory for tetraalkylstannanes vs. less successful for tetraalkenylstannanes

## How to apply

For each tetraorganostannane (e.g. Et4Sn, Vi4Sn), construct a hydrocarbon analogue by replacing the central tin atom with carbon, then apply hydrocarbon vapour pressure estimators to the analogue. Implement the Antoine equation baseline parameterized from the normal boiling point of each analogue, and the Modified Watson correlation using the same boiling point and molecular-size inputs. Extrapolate estimates down to 20°C, which lies below the experimental GC measurement range of 40–120°C used for the determinations. Compare estimates to experimental 20°C values (Et4Sn 0.16 kPa; Vi4Sn 0.35 kPa) to judge whether the hydrocarbon QSPR transfers: it proved satisfactory for tetraalkylstannanes but underestimated tetraalkenylstannanes. Write all estimated and experimental vapour pressures to vapour_pressure_estimates.csv.

## Related tools

- **Shimadzu GC-15A gas chromatograph** (Source of the experimental equilibrium vapour pressures (40–120°C, FID detector, SE-30 capillary column) against which the hydrocarbon-analogue estimates are validated)
- **SE-30 capillary column** (2 m × 0.55 mm ID, 1.2 µm film column used for the relative GC retention-time vapour pressure determinations used as ground truth)
- **Antoine equation** (Hydrocarbon-based vapour pressure estimator applied to the carbon-substituted analogue, parameterized from the analogue's normal boiling point)
- **Modified Watson correlation** (Second hydrocarbon-based estimator applied with the same boiling point and molecular-size inputs to cross-check the Antoine baseline)

## Examples

```
python estimate_vp.py --compounds Et4Sn,Vi4Sn --method antoine,modified_watson --T 20 --extrapolate --out vapour_pressure_estimates.csv
```

## Evaluation signals

- Antoine estimate for Et4Sn (0.15 kPa) matches its reported value within ~0.01 kPa of the experimental 0.16 kPa
- Estimates for Vi4Sn (0.26 and 0.25 kPa) reproduce the paper's qualitative finding of underestimation vs the experimental 0.35 kPa
- All 20°C estimates are extrapolations below the 40–120°C experimental measurement range and should be flagged as such
- Estimated values for each compound should be written to vapour_pressure_estimates.csv with both estimated and experimental columns comparable
- The alkyl/alkenyl split should emerge: errors small for tetraalkylstannanes, systematically low for tetraalkenylstannanes

## Limitations

- Hydrocarbon-based estimation methods are less successful for tetraalkenylstannanes (Vi4Sn estimated 0.26/0.25 kPa vs 0.35 kPa experimental)
- Lack of appropriate boiling point data prevents confirming whether the estimation methods are adequate for other tetraalkylstannanes and tetraalkenylstannanes
- The 20°C values are extrapolations below the 40–120°C GC measurement range, adding uncertainty
- No explicit alternative estimation method is named for tetraalkenylstannanes, for which hydrocarbon-based methods fail
- There is little data on molecular descriptors of organostannanes other than MW, limiting descriptor-based QSPR alternatives

## Evidence
