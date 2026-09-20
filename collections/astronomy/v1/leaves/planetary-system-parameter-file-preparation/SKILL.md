---
name: planetary-system-parameter-file-preparation
description: Use when a new multi-planet system with published radii (e.g. from CHEOPS+TESS
  joint modelling) and RVs (e.g. ESPRESSO/HIRES) must be set up as input for plaNETic
  internal-structure posterior runs.
when_to_use_negative:
- The planets fall outside plaNETic's validated mass range of 0.5–15 Mearth
- Only TESS photometry with low per-transit SNR and large gaps is available for small
  planets — the paper shows planet d's radius deviated by >3σ from the discovery paper,
  so radii should come from CHEOPS+TESS joint modelling instead
- The goal is forward-modelling planet formation rather than Bayesian internal-structure
  inference from observed parameters
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: plaNETic
  role: Framework whose 'run_grid' folder structure and 'stellar_planetary_parameters.csv'
    format the new system's inputs must follow
  repo: github.com/joannegger/plaNETic
- name: Python 3.9
  role: Runtime for the virtual environment in which plaNETic is installed and the
    adapted 'run_grid_xxx.py' is executed
- name: CHEOPS
  role: Source of the planetary radii (9.6 days of observations, jointly modelled
    with TESS) recorded in the parameter file
- name: TESS
  role: Source of two photometric sectors jointly modelled with CHEOPS to derive the
    radii entered in the parameter file
- name: ESPRESSO
  role: Source of published RV masses for the planets listed in the parameter file
- name: HIRES
  role: Source of published RV masses for the planets listed in the parameter file
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202450472
    title: Unveiling the internal structure and formation history of the three planets
      transiting HIP 29442 (TOI-469) with CHEOPS
  source_content_hash: sha256:d2aa8fb82829f7bb23b59fe3d069be89da347bd42155b18e3be2bfd3b7bbb836
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BICEPS
  - CHEOPS
  - ESPRESSO
  - HIRES
  - Python
  - Python 3.9
  - TESS
  - TensorFlow
  - plaNETic
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# planetary-system-parameter-file-preparation

## Summary

Prepare a plaNETic 'run_grid' subfolder for a new planetary system (here HIP 29442 / TOI-469) by writing a 'stellar_planetary_parameters.csv' with observed host-star and planet properties from CHEOPS+TESS radius fits and ESPRESSO/HIRES RVs, plus an adapted 'run_grid_xxx.py' executable cloned from the 'TOI-469_Egger+' template.

## When to use

Use when a new multi-planet system with published radii (e.g. from CHEOPS+TESS joint modelling) and RVs (e.g. ESPRESSO/HIRES) must be set up as input for plaNETic internal-structure posterior runs.

## When NOT to use

- The planets fall outside plaNETic's validated mass range of 0.5–15 Mearth
- Only TESS photometry with low per-transit SNR and large gaps is available for small planets — the paper shows planet d's radius deviated by >3σ from the discovery paper, so radii should come from CHEOPS+TESS joint modelling instead
- The goal is forward-modelling planet formation rather than Bayesian internal-structure inference from observed parameters

## Inputs

- Observed host-star properties (e.g. HIP 29442 stellar parameters)
- Planetary radii from CHEOPS+TESS joint photometry fit (9.6 days CHEOPS, two TESS sectors)
- Published radial velocities from ESPRESSO and HIRES
- Template folder 'run_grid/TOI-469_Egger+' (or 'TOI-238_Egger+') containing the parameter-file structure and 'run_grid_xxx.py' executable

## Outputs

- New subfolder in 'run_grid' mirroring the template structure
- 'stellar_planetary_parameters.csv' with host-star and planetary observed properties
- Adapted executable 'run_grid_xxx.py' pointing at the new system

## How to apply

Create a new subfolder in 'run_grid' with the same structure as the 'TOI-469_Egger+' (or 'TOI-238_Egger+') template. Populate 'stellar_planetary_parameters.csv' with the observed properties of the host star and every planet to be modelled — for HIP 29442 this means the CHEOPS+TESS joint-fit radii (3.410 ± 0.046, 1.551 ± 0.045, 1.538 ± 0.049 R⊕ for planets b, c, d) and the published ESPRESSO/HIRES radial velocities. Copy and adapt the executable 'run_grid_xxx.py' from the template so it points at the new system's parameters, keeping the default 10,000 sampled stars and 10,000 planets per star and the six prior options (A1–A3, B1–B3). Run with Python 3.9 in a GPU-enabled environment; if the trained DNNs are not found, that is a GitHub Large File Storage issue, not a parameter-file error.

## Related tools

- **plaNETic** (Framework whose 'run_grid' folder structure and 'stellar_planetary_parameters.csv' format the new system's inputs must follow) — github.com/joannegger/plaNETic
- **Python 3.9** (Runtime for the virtual environment in which plaNETic is installed and the adapted 'run_grid_xxx.py' is executed)
- **CHEOPS** (Source of the planetary radii (9.6 days of observations, jointly modelled with TESS) recorded in the parameter file)
- **TESS** (Source of two photometric sectors jointly modelled with CHEOPS to derive the radii entered in the parameter file)
- **ESPRESSO** (Source of published RV masses for the planets listed in the parameter file)
- **HIRES** (Source of published RV masses for the planets listed in the parameter file)

## Examples

```
cd run_grid/HIP29442_new && cp ../TOI-469_Egger+/run_grid_TOI-469.py run_grid_HIP29442.py  # then edit stellar_planetary_parameters.csv and run: python run_grid_HIP29442.py
```

## Evaluation signals

- The new 'run_grid' subfolder matches the structure of the 'TOI-469_Egger+' or 'TOI-238_Egger+' template (parameter CSV + executable present)
- 'stellar_planetary_parameters.csv' includes observed properties of the host star and all three planets b, c, d with the CHEOPS+TESS radii and ESPRESSO/HIRES RV inputs
- The executable correctly points at the HIP 29442 system parameters rather than the template system's
- Running the executable produces 'xxx_posterior.npy' files in a 'posteriors' subfolder, one per prior option (A1–A3, B1–B3)
- Each output .npy loads as a numpy array with trailing dimension 28 (matching the reference TOI-238 b shape (1, 771909, 28) pattern)

## Limitations

- If plaNETic cannot find the trained DNNs, this is a GitHub Large File Storage issue — contact the authors rather than re-editing the parameter file
- Multiplanetary-system runs scale linearly in computation time with the number of planets; all planets are modelled simultaneously
- Reducing the default 10,000 sampled stars / 10,000 planets per star shortens runtime but yields smaller posterior distributions
- Radii from TESS photometry alone with low per-transit SNR and large gaps are unreliable (planet d deviated by >3σ from the discovery-paper value)

## Evidence

- [methods] run_grid subfolder structure: "create a new subfolder in 'run_grid' with the same structure as 'TOI-469_Egger+' or 'TOI-238_Egger+'"
- [methods] parameter file contents: "Parameter file 'stellar_planetary_parameters.csv' with the observed properties of the host star and all planets in the system that you would like to model"
- [readme] output generation: "Then simply adapt and run the executable, which will generate 'xxx_posterior.npy' files in a subfolder 'posteriors'."
- [abstract] radii inputs: "planetary radii of 3.410 ± 0.046, 1.551 ± 0.045, and 1.538 ± 0.049 R⊕ for planets b, c, and d"
- [methods] runtime environment: "To install the code, create a new virtual environment with Python 3.9, then git clone plaNETic and run 'pip install .'"
