---
name: exclusion-limit-overlay-visualization-vs-mass
description: Use when a BBN parameter scan over the mass of a thermalized new species
  (scalar, vector, Majorana, or Dirac X) yields abundance predictions that must be
  compared against observational exclusion limits on a shared mass axis.
when_to_use_negative:
- When the analysis is a Standard-Model-only BBN computation with no new-physics flags
  (NP_nu_flag/NP_e_flag) and no mass scan to overlay exclusion limits on.
- When the observables of interest are not BBN light-element abundances or N_eff (e.g.
  CMB-only thermal-history fits), since the skill is built around the Fig. 4 abundance-band
  comparison.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3450
tools:
- name: PRyMordial
  role: Evolves the coupled thermal history (N_eff, n-to-p conversion) per mass point
    with NP flags enabled, computes BBN abundances, and provides the Fig. 4 style
    abundance-band data for the overlay plot.
  repo: https://github.com/vallima/PRyMordial
- name: Python 3
  role: Runtime for PRyMordial and for rendering the abundance-band figure with the
    overlaid exclusion region.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1140/epjc/s10052-024-12442-0
    title: 'PRyMordial: the first three minutes, within and beyond the standard model'
  source_content_hash: sha256:d783d6f1dc49337f7954152de355741c763395c7a1b42d59ff2b054083af6ff7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vallima/PRyMordial
  tools:
  - PRyMordial
  - Python 3
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3450
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# exclusion-limit-overlay-visualization-vs-mass

## Summary

Assemble PRyMordial-predicted light-element abundance bands across a new-physics species mass grid (Fig. 4 style: electrophilic/neutrinophilic X scan) and overlay the observational exclusion region on the same mass axis, producing fig4_np_abundance_bands.png.

## When to use

Use when a BBN parameter scan over the mass of a thermalized new species (scalar, vector, Majorana, or Dirac X) yields abundance predictions that must be compared against observational exclusion limits on a shared mass axis.

## When NOT to use

- When the analysis is a Standard-Model-only BBN computation with no new-physics flags (NP_nu_flag/NP_e_flag) and no mass scan to overlay exclusion limits on.
- When the observables of interest are not BBN light-element abundances or N_eff (e.g. CMB-only thermal-history fits), since the skill is built around the Fig. 4 abundance-band comparison.

## Inputs

- PRyMordial run configuration with NP_nu_flag and NP_e_flag enabled
- Mass grid over which species X thermalizes with the plasma or neutrino bath
- Species X spin/quantum structure selection (real/complex scalar, massive vector, Majorana or Dirac fermion)
- Observational exclusion region data on the species mass axis

## Outputs

- fig4_np_abundance_bands.png — abundance-band plot versus X mass with overlaid observational exclusion region
- Predicted light-element abundance bands (e.g. Yp, D/H) as a function of species mass across the scan

## How to apply

Run PRyMordial with NP_nu_flag and NP_e_flag enabled to couple species X to the neutrino bath and electromagnetic plasma, and evolve the thermal history with the ODE solver retaining non-instantaneous decoupling effects to obtain N_eff and the neutron-to-proton conversion history per mass point. Compute BBN light-element yields for each point on the defined mass grid, then assemble the predicted abundance bands versus species mass. Render the Fig. 4 style abundance-band plot and overlay the observational exclusion region on the same mass axis, saving the figure as fig4_np_abundance_bands.png. The choice of spin/quantum structure (scalar, vector, Majorana, or Dirac) and whether X thermalizes with the plasma or the neutrino bath must be fixed before scanning, since these determine whether the primary effects are the expansion rate (time-temperature relation) or the neutrino-to-photon temperature ratio evolution.

## Related tools

- **PRyMordial** (Evolves the coupled thermal history (N_eff, n-to-p conversion) per mass point with NP flags enabled, computes BBN abundances, and provides the Fig. 4 style abundance-band data for the overlay plot.) — https://github.com/vallima/PRyMordial
- **Python 3** (Runtime for PRyMordial and for rendering the abundance-band figure with the overlaid exclusion region.)

## Examples

```
python runPRyM_julia.py  # or run the PRyMdemoNP Jupyter notebook after setting NP_nu_flag/NP_e_flag and the X mass grid
```

## Evaluation signals

- Reproduced abundance bands and overlaid exclusion boundary show visual agreement with the corresponding Fig. 4 panel of the published PRyMordial paper (band positions and exclusion boundary match).
- N_eff and abundance yields are computed with non-instantaneous decoupling effects retained, not with an instantaneous-decoupling approximation.
- The mass axis of the predicted bands and of the observational exclusion region coincide, so the exclusion comparison is a same-axis overlay.
- Each mass point on the grid has a complete coupled thermal-history solution and abundance yield — no missing points in the band.
- The output figure is saved under the expected filename fig4_np_abundance_bands.png.

## Limitations

- PRyMordial's optional dependencies (Numba, Numdifftools, Vegas, PyJulia/diffeqpy with Julia/SciML) affect performance; the code 'does not make use of Vegas by default', so omitting recommended installs may slow the mass-grid scan.
- Only species thermalized with the photon-electron plasma or the neutrino bath are covered by the NP_e_flag/NP_nu_flag machinery; decoupled or non-thermal species require a different treatment.
- Validation is visual against the published Fig. 4 panel; the article's discussion section content is absent from the extracted document, so no additional limitations or reproducibility caveats were extractable.

## Evidence

- [other] workflow card: NP flags and scan design: [evidence span withheld — anchor status: not_found]
- [other] workflow card: figure output and validation: [evidence span withheld — anchor status: not_found]
- [other] workflow card: ODE evolution detail: [evidence span withheld — anchor status: not_found]
- [abstract] abstract: code capability: "The code offers fast and precise evaluation of BBN light-element abundances together with the effective number of relativistic degrees of freedom, including non-instantaneous decoupling effects."
- [readme] README: runtime and invocation: "To run a quick example, open the terminal and once in PRyMordial folder just type: 

python runPRyM_julia.py"
- [readme] README: NP demo notebook: "Examples on the usage of PRyMordial are provided in the Jupyter notebooks PRyMdemoSM and PRyMdemoNP."
