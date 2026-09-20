---
name: observational-constraint-overlay-plotting
description: Use when BBN abundance predictions have been computed over a grid of
  eta_B values and the goal is to compare them graphically against quasar absorption
  / deuterium observations and the CMB baryon-density band.
when_to_use_negative:
- When the goal is a New-Physics scan of a single BSM scenario rather than Standard
  Model abundances compared against the PDG/CMB observational constraints over eta_B.
- When the thermal background or weak rates have not yet been computed and stored
  — plot only after the BBN network solver output exists for each eta_B.
- When abundances are needed as input to a Monte Carlo likelihood analysis only (use
  the dedicated PRyM_Yp_DH_cosmoMC_2023.dat table instead of re-deriving plots).
edam_operation: http://edamontology.org/operation_0337
tools:
- name: PRyMordial
  role: Computes the SM BBN light-element abundances over the eta_B grid that populate
    the plotted curves
  repo: github.com/vallima/PRyMordial
- name: Python 3
  role: Language in which PRyMordial runs entirely; used to drive the eta_B sweep,
    matplotlib plotting and CSV export
- name: matplotlib
  role: Plots the four abundance curves versus eta_B with overlaid PDG observational
    data points and CMB constraint band
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:ea6df622a435ed51c81c436c8b21d48505b9cc17ee753b027751ced3b2ea19cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRyMordial
  - Python 3
  - matplotlib
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# observational-constraint-overlay-plotting

## Summary

Plots PRyMordial-predicted primordial mass fractions of helium-4, deuterium, helium-3 and lithium-7 as a function of the baryon-to-photon ratio eta_B, overlaying PDG observational data points and the CMB baryon-density constraint band to reproduce PDG Figure 24.1 style diagnostics.

## When to use

Use when BBN abundance predictions have been computed over a grid of eta_B values and the goal is to compare them graphically against quasar absorption / deuterium observations and the CMB baryon-density band.

## When NOT to use

- When the goal is a New-Physics scan of a single BSM scenario rather than Standard Model abundances compared against the PDG/CMB observational constraints over eta_B.
- When the thermal background or weak rates have not yet been computed and stored — plot only after the BBN network solver output exists for each eta_B.
- When abundances are needed as input to a Monte Carlo likelihood analysis only (use the dedicated PRyM_Yp_DH_cosmoMC_2023.dat table instead of re-deriving plots).

## Inputs

- Solver output: final mass fractions of helium-4, deuterium, helium-3 and lithium-7 for each eta_B value on the grid
- Grid of baryon-to-photon ratio eta_B values spanning the observationally allowed region
- PDG observational data points for primordial abundances (as in PDG Figure 24.1)
- CMB baryon-density constraint band
- Stored thermal background evolution and weak neutron-to-proton conversion rates shipped with PRyMordial

## Outputs

- fig2_abundances_vs_etaB.png (four abundance curves vs eta_B with PDG points and CMB band overlaid)
- abundance_curves.csv (abundance-versus-eta_B table for helium-4, deuterium, helium-3, lithium-7)

## How to apply

First run the PRyMordial Standard Model BBN network (Python 3, e.g. via runPRyM_julia.py or the PRyMdemoSM notebook) over a grid of baryon-to-photon ratio eta_B values spanning the observationally allowed region, extracting final mass fractions of helium-4, deuterium, helium-3 and lithium-7 for each eta_B from the solver output. Then plot the four abundance curves versus eta_B with matplotlib, overlaying the PDG observational data points (e.g. deuterium from quasar absorption) and the CMB baryon-density constraint band as shown in PDG Figure 24.1. The rationale is that BBN observables constrain cosmology: agreement between the eta_B-dependence of the curves, the deuterium observation, and the CMB band validates both the BBN computation and the cosmological inputs. Save the figure as fig2_abundances_vs_etaB.png and write the abundance-versus-eta_B table to abundance_curves.csv for downstream use (e.g. with the shipped PRyM_Yp_DH_cosmoMC_2023.dat BBN likelihood table).

## Related tools

- **PRyMordial** (Computes the SM BBN light-element abundances over the eta_B grid that populate the plotted curves) — github.com/vallima/PRyMordial
- **Python 3** (Language in which PRyMordial runs entirely; used to drive the eta_B sweep, matplotlib plotting and CSV export)
- **matplotlib** (Plots the four abundance curves versus eta_B with overlaid PDG observational data points and CMB constraint band)

## Examples

```
python runPRyM_julia.py
```

## Evaluation signals

- Curves reproduce the PDG Figure 24.1 behaviour: helium-4 mass fraction high and weakly dependent on eta_B, deuterium/helium-3/lithium-7 decreasing with increasing eta_B.
- The deuterium prediction matches quasar absorption observations at the eta_B preferred by the CMB baryon-density constraint.
- The overlaid CMB baryon-density band intersects the eta_B region where predicted abundances agree with the observational data points.
- abundance_curves.csv contains final mass fractions of all four species (Y_P, D/H, He-3, Li-7) for every eta_B value on the grid.
- Figure file fig2_abundances_vs_etaB.png renders all four curves plus both overlay layers (PDG points, CMB band).

## Limitations

- The plot validation applies to the Standard Model run; BSM scenarios active during BBN shift abundances and will not reproduce the PDG Figure 24.1 baseline.
- PRyMordial is written in Python with optional Julia/SciML routines; skipping recommended dependencies (Numba, Numdifftools) changes performance, not the plotted results, but the Julia path (PyJulia, diffeqpy) must be installed to use that backend.
- Discussion-section changelog is absent from the source, so reproducibility caveats for the figure could not be extracted from the paper.

## Evidence

- [results] PRyMordial predicts the primordial abundances of helium-4, deuterium, helium-3, and lithium-7 as a function of the cosmic baryon density, reproducing the PDG Figure 24.1 behaviour: [evidence span withheld — anchor status: not_found]
- [methods] Plot the four abundance curves versus eta_B with matplotlib and overlay the PDG observational data points and the CMB baryon-density constraint band as shown in PDG Figure 24.1.: [evidence span withheld — anchor status: not_found]
- [readme] A dedicated numerical table for standard cosmological analyses involving BBN likelihoods is also in the repo.: "A dedicated numerical table for standard cosmological analyses involving BBN likelihoods is also in the repo."
- [abstract] The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
