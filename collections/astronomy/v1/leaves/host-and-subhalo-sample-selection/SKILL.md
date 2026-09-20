---
name: host-and-subhalo-sample-selection
description: 'Use when building the per-halo fitting sample for diffmah MAH fits:
  selecting both host halos and subhalos with present-day mass greater than 10^11
  Msun from merger tree catalogs such as TNG.'
when_to_use_negative:
- The input is a population-level or mean MAH calibration dataset rather than individual
  halo merger trees — the diffmah population-assembly model is the appropriate tool
  there.
- The halos of interest have present-day mass below 10^11 Msun, outside the regime
  where the paper demonstrates 0.1 dex fit accuracy.
- The analysis requires resolved growth at t < 1 Gyr, where the paper does not quantify
  model accuracy.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: diffmah
  role: Provides the differentiable 3-parameter MAH model and fitting entry points
    (diffmah_fitter_demo.ipynb, history_fitting_script.py with mpi4py) applied to
    each selected halo
  repo: https://github.com/ArgonneCPAC/diffmah/
- name: JAX
  role: Autodiff backend for the JAX-based gradient optimizer that minimizes log-mass
    residuals when fitting each selected halo's history
- name: numpy
  role: Array handling for loading merger tree catalogs and extracting/filtering per-halo
    mass accretion histories
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.21105/astro.2105.05859
    title: A Differentiable Model of the Assembly of Individual and Populations of
      Dark Matter Halos
  source_content_hash: sha256:dc54903e12731ca350338b672dcede14a803f14baab0359f8f448a76d349fcba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArgonneCPAC/diffmah
  tools:
  - JAX
  - Python
  - diffmah
  - numpy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# host-and-subhalo-sample-selection

## Summary

Constructs the individual-halo fitting sample for diffmah mass accretion history (MAH) work: selecting both host halos and subhalos with present-day mass greater than 10^11 Msun from halo merger tree catalogs (gravity-only and TNG hydrodynamical) and extracting each object's log halo mass vs. cosmic time history for 3-parameter diffmah fitting.

## When to use

Use when building the per-halo fitting sample for diffmah MAH fits: selecting both host halos and subhalos with present-day mass greater than 10^11 Msun from merger tree catalogs such as TNG.

## When NOT to use

- The input is a population-level or mean MAH calibration dataset rather than individual halo merger trees — the diffmah population-assembly model is the appropriate tool there.
- The halos of interest have present-day mass below 10^11 Msun, outside the regime where the paper demonstrates 0.1 dex fit accuracy.
- The analysis requires resolved growth at t < 1 Gyr, where the paper does not quantify model accuracy.

## Inputs

- Halo merger tree catalog from a cosmological simulation (e.g., TNG hydrodynamical or gravity-only) with per-snapshot halo masses
- Host-halo vs. subhalo classification (tree-node labels) for present-day (z=0) nodes
- Cosmic time array mapping merger tree snapshots to cosmic time

## Outputs

- Selected fitting sample of host halos and subhalos with present-day mass greater than 10^11 Msun
- Per-halo mass accretion histories (log halo mass vs. cosmic time) as numpy arrays ready for 3-parameter diffmah fitting

## How to apply

Load the halo merger tree catalog (TNG hydrodynamical or gravity-only) and classify each present-day tree node as a host halo or subhalo, since the paper's accuracy claim explicitly covers both populations. Apply the mass cut on present-day mass, retaining every halo with M0 > 10^11 Msun — the threshold above which the 3-parameter diffmah model achieves ~0.1 dex typical accuracy. For each retained halo, extract its mass accretion history (log halo mass vs. cosmic time) from the merger tree using numpy for array handling, keeping snapshot cosmic times so residuals can be evaluated per epoch. Restrict fit-quality acceptance to t > 1 Gyr, the range over which the paper quantifies the 0.1 dex criterion; do not gate sample membership on residuals at earlier times. Feed the selected MAHs to the diffmah fitting machinery — diffmah_fitter_demo.ipynb for a single simulated halo, or history_fitting_script.py with mpi4py to fit large numbers of halos in parallel — which minimizes log-mass residuals using JAX-based gradients.

## Related tools

- **diffmah** (Provides the differentiable 3-parameter MAH model and fitting entry points (diffmah_fitter_demo.ipynb, history_fitting_script.py with mpi4py) applied to each selected halo) — https://github.com/ArgonneCPAC/diffmah/
- **JAX** (Autodiff backend for the JAX-based gradient optimizer that minimizes log-mass residuals when fitting each selected halo's history)
- **numpy** (Array handling for loading merger tree catalogs and extracting/filtering per-halo mass accretion histories)

## Examples

```
conda create -c conda-forge -n diffit python=3.9 numpy jax pytest ipython jupyter matplotlib scipy h5py diffmah
```

## Evaluation signals

- After selection, every retained host halo and subhalo has present-day mass greater than 10^11 Msun (min of M0 over the sample exceeds the cut).
- Both populations are present: the sample includes subhalos as well as host halos, matching the paper's scope.
- Each selected halo has MAH coverage at t > 1 Gyr so the 0.1 dex acceptance criterion is evaluable.
- Downstream diffmah fits on the selected sample reproduce typical log-mass residuals of ~0.1 dex for t > 1 Gyr.
- Applying the same cut to gravity-only and TNG trees yields consistent host/subhalo stratification and comparable fit diagnostics.

## Limitations

- The mass cut applies to present-day (z=0) mass and does not guarantee well-resolved early-time growth; accuracy was quantified only for t > 1 Gyr.
- The paper relies on large samples of merger trees from high-resolution cosmological simulations; lower-resolution trees may degrade MAH extraction and fit quality.
- No changelog is available for the diffmah repository, so version-dependent behavior of the fitting scripts is not documented.

## Evidence

- [abstract] Mass threshold and accuracy target for the selected sample: "we demonstrate that our 3-parameter model can approximate halo growth with a typical accuracy of 0.1 dex for $t > 1$ Gyr for all halos of present-day mass greater than $10^{11} M_{\odot}$"
- [abstract] TNG hydrodynamical halos are in scope for the sample: "as well as in the TNG hydrodynamical simulation"
- [abstract] Input data type: merger trees from high-resolution cosmological simulations: "Using large samples of halo merger trees taken from high-resolution cosmological simulations"
- [methods] diffmah is the model applied to each selected halo's mass assembly history: "Diffmah is a differentiable model for the mass assembly history of individual and populations of dark matter halos."
- [readme] Parallel fitting entry point for large selected samples: [evidence span withheld — anchor status: external_doc]
