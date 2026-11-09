---
name: gaussian-process-noise-modelling
description: Use when fitting RV timeseries (e.g. Barnard's star/GJ 699, HARPS-type
  instruments) where stellar activity or correlated noise may mimic planetary signals
  and competing noise models (jitter, moving-average, GP) must be compared by Evidence
  (ln Z).
when_to_use_negative:
- The data show no evidence of correlated/stellar-activity noise and a simple jitter
  term suffices — adding a GP risks absorbing genuine Keplerian signals.
- The dataset is a single-planet, high-SNR signal with clean white noise (e.g. the
  51 Peg test case), where noise-model competition adds complexity without benefit.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: EMPEROR (astroemperor)
  role: Framework orchestrating the noise-model construction (sim.switch_celerite,
    sim.moav) and Bayesian model comparison over Evidence
  repo: github.com/ReddTea/astroemperor
- name: celerite2
  role: Computes the Gaussian Process likelihood with a RotationTerm kernel for correlated
    stellar-activity noise
  repo: https://celerite2.readthedocs.io/en/latest/
- name: reddemcee
  role: Adaptive parallel-tempering MCMC sampler whose thermodynamic-integration /
    stepping-stones evidence (ln Z) decides between noise models
  repo: https://github.com/ReddTea/reddemcee
- name: kepler.py
  role: Fast Kepler's-equation solver used for the Keplerian component fitted alongside
    the GP noise model
  repo: https://github.com/dfm/kepler.py
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:bd99b8c42c7d3566046a6ea3e6a649e6d72021645abf2748578cc8477e706244
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPEROR (astroemperor)
  - Python
  - celerite2
  - kepler.py
  - reddemcee
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gaussian-process-noise-modelling

## Summary

Model stellar-activity and instrumental correlated noise in radial velocity timeseries with a Gaussian Process (RotationTerm kernel via celerite2) alongside jitter and moving-average alternatives, so EMPEROR's Bayesian model comparison can select the optimal noise treatment.

## When to use

Use when fitting RV timeseries (e.g. Barnard's star/GJ 699, HARPS-type instruments) where stellar activity or correlated noise may mimic planetary signals and competing noise models (jitter, moving-average, GP) must be compared by Evidence (ln Z).

## When NOT to use

- The data show no evidence of correlated/stellar-activity noise and a simple jitter term suffices — adding a GP risks absorbing genuine Keplerian signals.
- The dataset is a single-planet, high-SNR signal with clean white noise (e.g. the 51 Peg test case), where noise-model competition adds complexity without benefit.

## Inputs

- Radial velocity timeseries catalogue (e.g. datafiles/GJ699/RV, multi-instrument RV files)
- EMPEROR Simulation object with starmass set (to derive minimum mass and semi-major axis)
- celerite2 RotationTerm kernel configuration for the GP noise model
- reddemcee engine setup [ntemps, nwalkers, nsweeps, nsteps] with adaptive ladder parameters (adapt_tau, adapt_nu, adapt_mode)

## Outputs

- Per-model posterior distributions (period, amplitude, eccentricity, longitude, offsets, GP hyperparameters) in the datalogs tables and plots
- Evidence (ln Z) values per noise model, compared under the set tolerance
- Selected best noise model with derived planet parameters (period, minimum mass, semi-major axis)

## How to apply

Within an EMPEROR simulation, enable the GP path with sim.switch_celerite = True, which builds the noise model as a celerite2 Gaussian Process using a RotationTerm kernel that captures stellar rotation (cf. P_rot = 29.72 d recovered for HD 55693). Competing noise treatments are constructed in parallel runs: jitter via add_jitter, moving-average blocks via sim.moav order, and the GP; each is attached to the same Keplerian search. Execute sim.autorun over increasing Keplerian counts with sim.set_comparison_criteria('Evidence') and a set_tolerance threshold so a more complex noise model is accepted only when ln Z (estimated via thermodynamic integration or stepping stones in reddemcee) improves beyond tolerance. For Barnard's star the winning configuration must still retrieve the 3.1536±0.0003 d, 0.38±0.03 M_earth planet rather than absorbing it into the noise, confirming the GP does not over-fit the Doppler signal.

## Related tools

- **EMPEROR (astroemperor)** (Framework orchestrating the noise-model construction (sim.switch_celerite, sim.moav) and Bayesian model comparison over Evidence) — github.com/ReddTea/astroemperor
- **celerite2** (Computes the Gaussian Process likelihood with a RotationTerm kernel for correlated stellar-activity noise) — https://celerite2.readthedocs.io/en/latest/
- **reddemcee** (Adaptive parallel-tempering MCMC sampler whose thermodynamic-integration / stepping-stones evidence (ln Z) decides between noise models) — https://github.com/ReddTea/reddemcee
- **kepler.py** (Fast Kepler's-equation solver used for the Keplerian component fitted alongside the GP noise model) — https://github.com/dfm/kepler.py

## Examples

```
sim.switch_celerite = True; sim.set_engine('reddemcee'); sim.set_comparison_criteria('Evidence'); sim.autorun(1, 1)
```

## Evaluation signals

- Retrieved Keplerian for Barnard's star matches P = 3.1536±0.0003 d, m sin i = 0.38±0.03 M_earth, a = 0.02315±0.00039 AU within uncertainties
- The GP model is retained only if its ln Z exceeds the competing jitter/moav models by more than the value set with set_tolerance
- Evidence values are computed via reddemcee's thermodynamic integration (TI+), stepping stones (SS+), or the hybrid estimator
- GP hyperparameters (e.g. rotation period of the RotationTerm) converge to physically plausible stellar-rotation timescales, not the planet period

## Limitations

- Purely statistical inference might be insufficient on its own for robust exoplanet detection — ln Z model selection alone can mis-assign signals between GP and Keplerian components.
- Evidence (ln Z) is sensitive to prior selection (demonstrated for HD 55693), so noise-model comparisons must fix priors across competing models.
- kepler.solve returns garbage for eccentricities outside 0–1; GP-amplified eccentricity posteriors can drift into invalid ranges.

## Evidence

- [methods] Build candidate models with different noise treatments: [evidence span withheld — anchor status: not_found]
- [methods] Evidence estimation: "Curvature-aware Thermodynamic Integration (TI+), Geometric-Bridge Stepping Stones (SS+), and a Hybrid algorithm"
- [abstract] ln Z prior sensitivity: "the stellar rotation, P rot = 29.72 −0.02 + 0.01 , and magnetic cycle, P mag = 2557.0 −36.7 + 70.1 , were recovered, while demonstrating the sensitivity of ln 𝒵 to prior selection."
- [abstract] Purely statistical inference caveat: "Purely statistical inference might be insufficient on its own for robust exoplanet detection."
- [readme] kepler validity range: [evidence span withheld — anchor status: external_doc]
