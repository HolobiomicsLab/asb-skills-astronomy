---
name: cmb-baryon-density-constraint-comparison
description: Use when predicted BBN light-element abundances are available across
  a grid of eta_B values and you need to check consistency with the CMB-inferred baryon
  density and quasar absorption deuterium observations.
when_to_use_negative:
- The analysis targets New Physics scenarios active during BBN rather than the Standard
  Model baseline — use the BSM-capable workflow instead, since this skill reproduces
  the SM PDG benchmark.
- You need a single-point abundance prediction at one fixed cosmology rather than
  abundance curves as a function of eta_B with observational overlays.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: PRyMordial
  role: Runs the Standard Model BBN network over the eta_B grid using the stored thermal
    background and weak rates, yielding final mass fractions of helium-4, deuterium,
    helium-3, and lithium-7.
  repo: https://github.com/vallima/PRyMordial
- name: Python 3
  role: Runtime for PRyMordial and for matplotlib plotting of the abundance curves
    with observational overlays and CSV export.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:83b7e38bdd5cde1ef5ae6c96fa1a51a349dfaf53898543a86360c33acecb8553
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallima/PRyMordial
  tools:
  - PRyMordial
  - Python 3
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cmb-baryon-density-constraint-comparison

## Summary

Overlays PRyMordial-predicted primordial abundance curves (helium-4, deuterium, helium-3, lithium-7) versus baryon-to-photon ratio eta_B with the CMB baryon-density constraint band, testing whether BBN predictions agree with the independently measured cosmic baryon density.

## When to use

Use when predicted BBN light-element abundances are available across a grid of eta_B values and you need to check consistency with the CMB-inferred baryon density and quasar absorption deuterium observations.

## When NOT to use

- The analysis targets New Physics scenarios active during BBN rather than the Standard Model baseline — use the BSM-capable workflow instead, since this skill reproduces the SM PDG benchmark.
- You need a single-point abundance prediction at one fixed cosmology rather than abundance curves as a function of eta_B with observational overlays.

## Inputs

- Stored thermal background evolution shipped with the PRyMordial package
- Weak neutron-to-proton conversion rates shipped with the PRyMordial package
- Grid of baryon-to-photon ratio eta_B values spanning the observationally allowed region
- PDG observational data points for primordial abundances
- CMB baryon-density constraint band

## Outputs

- Plot of helium-4, deuterium, helium-3, and lithium-7 mass fractions versus eta_B overlaid with PDG data points and the CMB constraint band (fig2_abundances_vs_etaB.png)
- Table of abundance-versus-eta_B curves (abundance_curves.csv)

## How to apply

Run the PRyMordial Standard Model BBN network over a grid of baryon-to-photon ratio eta_B values spanning the observationally allowed region, loading the stored thermal background evolution and weak neutron-to-proton conversion rates shipped with the package. Extract final mass fractions of helium-4, deuterium, helium-3, and lithium-7 from the solver output for each eta_B value. Plot the four abundance curves versus eta_B and overlay the PDG observational data points and the CMB baryon-density constraint band, following the layout of PDG Figure 24.1. Judge success when the deuterium curve passes through the quasar absorption measurement at the eta_B value favored by the CMB constraint band, reproducing PDG Figure 24.1 behaviour as in PRyMordial Fig. 2.

## Related tools

- **PRyMordial** (Runs the Standard Model BBN network over the eta_B grid using the stored thermal background and weak rates, yielding final mass fractions of helium-4, deuterium, helium-3, and lithium-7.) — https://github.com/vallima/PRyMordial
- **Python 3** (Runtime for PRyMordial and for matplotlib plotting of the abundance curves with observational overlays and CSV export.)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- The predicted deuterium abundance matches quasar absorption observations at the eta_B value selected by the CMB baryon-density constraint band.
- The resulting plot reproduces the qualitative behaviour of PDG Figure 24.1 (PRyMordial Fig. 2): helium-4 weakly dependent on eta_B, deuterium and lithium-7 strongly varying.
- abundance_curves.csv contains mass fractions for all four light elements at every eta_B grid point from the solver output.
- The four abundance curves span the observationally allowed region of eta_B rather than a single fiducial value.

## Limitations

- Agreement between BBN and the CMB band is a consistency check within the Standard Model; discrepancies should be interpreted as model mismatch, not numerical error.
- PRyMordial includes non-instantaneous decoupling effects, so curves may differ from instantaneous-decoupling textbook treatments.

## Evidence

- [results] PRyMordial predicts the primordial abundances of helium-4, deuterium, helium-3, and lithium-7 as a function of the cosmic baryon density, reproducing the PDG Figure 24.1 behaviour: [evidence span withheld — anchor status: not_found]
- [abstract] The code offers fast and precise evaluation of BBN light-element abundances: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [abstract] PRyMordial is suitable for state-of-the-art analyses in the Standard Model: "PRyMordial is suitable for state-of-the-art analyses in the Standard Model as well as for general investigations into New Physics active during BBN."
- [readme] A dedicated numerical table for standard cosmological analyses involving BBN likelihoods: "A dedicated numerical table for standard cosmological analyses involving BBN likelihoods is also in the repo."
- [intro] the study of the Early Universe around the BBN epoch offers unique insight on New Physics (NP): "the study of the Early Universe around the BBN epoch offers unique insight on New Physics (NP)"
