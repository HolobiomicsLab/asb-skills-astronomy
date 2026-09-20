---
name: magnetic-cycle-period-recovery
description: Use when an RV timeseries shows long-period (multi-year) activity-related
  variability plus a stellar rotation signal, and you need posterior credible intervals
  for P_mag and P_rot with automatic model selection over ln Z.
when_to_use_negative:
- The goal is fitting short-period planetary Keplerians only (e.g. the 51 Peg or Barnard's
  star cases) rather than long-baseline activity cycles — the magnetic-cycle block
  and multi-year priors are unnecessary there
- The RV baseline is far shorter than the expected magnetic cycle timescale (here
  ~2557 d), so the long-period posterior cannot be meaningfully constrained and ln
  Z comparisons would be dominated by prior choice
- Data are astrometric (Hipparcos/Gaia) rather than radial velocity; use EMPEROR's
  joint RV+astrometry workflow instead of the activity-index setup
tools:
- name: EMPEROR (astroemperor)
  role: Framework that provides Simulation.load_data, add_condition, add_magnetic_cycle/add_sinusoid/add_sai
    blocks, set_comparison_criteria, set_tolerance and autorun for the activity-signal
    recovery
  repo: github.com/ReddTea/astroemperor
- name: reddemcee
  role: Adaptive Parallel Tempering MCMC sampler set as the engine (set_engine('reddemcee'))
    and configured via engine_config['setup'] = [ntemps, nwalkers, nsweeps, nsteps]
  repo: github.com/ReddTea/reddemcee
- name: emcee
  role: Underlying ensemble sampler on which reddemcee's parallel-tempering implementation
    is based; also used for autocorrelation times
  repo: github.com/dfm/emcee
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:a48d0eb1115f6fb08d265c5c613bafec6f5adb83dff3becc82dd7ee17b7055ca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMPEROR (astroemperor)
  - Python
  - emcee
  - reddemcee
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# magnetic-cycle-period-recovery

## Summary

Recover stellar magnetic cycle and rotation signals from radial velocity timeseries using EMPEROR's add_magnetic_cycle / add_sinusoid model blocks with parallel-tempering MCMC (reddemcee) and evidence-based model comparison, as demonstrated on the HD 55693 dataset.

## When to use

Use when an RV timeseries shows long-period (multi-year) activity-related variability plus a stellar rotation signal, and you need posterior credible intervals for P_mag and P_rot with automatic model selection over ln Z.

## When NOT to use

- The goal is fitting short-period planetary Keplerians only (e.g. the 51 Peg or Barnard's star cases) rather than long-baseline activity cycles — the magnetic-cycle block and multi-year priors are unnecessary there
- The RV baseline is far shorter than the expected magnetic cycle timescale (here ~2557 d), so the long-period posterior cannot be meaningfully constrained and ln Z comparisons would be dominated by prior choice
- Data are astrometric (Hipparcos/Gaia) rather than radial velocity; use EMPEROR's joint RV+astrometry workflow instead of the activity-index setup

## Inputs

- Radial velocity catalogue (time, RV, RV uncertainty, instrument) placed under datafiles/HD55693/RV
- Stellar activity index timeseries (for add_sai blocks)
- Prior specifications around expected rotation period (~29.72 d) and magnetic cycle period (~2557 d)

## Outputs

- Posterior estimates of P_rot and P_mag with credible intervals (e.g. P_rot = 29.72 (+0.01/−0.02) d, P_mag = 2557.0 (+70.1/−36.7) d)
- datalogs output files: best_fit.dat, values.tex, stats.tex
- Model-comparison log-evidence (ln Z) history across successive runs

## How to apply

Set up the working directory with 'datafiles' and 'datalogs' subfolders and place the RV catalogue under datafiles/HD55693/RV; instantiate astroemperor.Simulation and call load_data('HD55693'). Configure the parallel-tempering engine via sim.set_engine('reddemcee') and sim.engine_config['setup'] = [ntemps, nwalkers, nsweeps, nsteps] (paper setup: [10, 256, 2048, 1]). Add activity model blocks — add_magnetic_cycle / add_sinusoid for the long-term cycle, add_sai for stellar activity indices, plus add_offset and add_jitter — and set limits, priors and initial positions with add_condition, using priors centred near P_rot ~ 29.72 d and P_mag ~ 2557 d. Enable evidence-based model selection with set_comparison_criteria('Evidence') and set_tolerance(10) so a more complex model is only accepted when log-evidence improves by more than the tolerance, then run sim.autorun to search over increasing model complexity. Inspect datalogs (best_fit.dat, values.tex, stats.tex) and compile posterior periods with uncertainties; note the paper found ln Z is sensitive to prior selection, so keep priors defensible and report them.

## Related tools

- **EMPEROR (astroemperor)** (Framework that provides Simulation.load_data, add_condition, add_magnetic_cycle/add_sinusoid/add_sai blocks, set_comparison_criteria, set_tolerance and autorun for the activity-signal recovery) — github.com/ReddTea/astroemperor
- **reddemcee** (Adaptive Parallel Tempering MCMC sampler set as the engine (set_engine('reddemcee')) and configured via engine_config['setup'] = [ntemps, nwalkers, nsweeps, nsteps]) — github.com/ReddTea/reddemcee
- **emcee** (Underlying ensemble sampler on which reddemcee's parallel-tempering implementation is based; also used for autocorrelation times) — github.com/dfm/emcee

## Examples

```
import astroemperor; sim = astroemperor.Simulation(); sim.load_data('HD55693'); sim.set_engine('reddemcee'); sim.engine_config['setup'] = [10, 256, 2048, 1]; sim.autorun(1, 1)
```

## Evaluation signals

- Recovered posteriors match the reported values within credible intervals: P_rot = 29.72 (+0.01/−0.02) d and P_mag = 2557.0 (+70.1/−36.7) d
- Each accepted model increase improves ln Z by more than the set tolerance (set_tolerance(10)); check the evidence history in datalogs
- datalogs contains best_fit.dat, values.tex and stats.tex after autorun completes
- Engine configuration matches the paper setup [10, 256, 2048, 1] (ntemps, nwalkers, nsweeps, nsteps) or a justified variant
- Priors around 29.72 d and 2557 d are documented, since ln Z showed sensitivity to prior selection in the paper

## Limitations

- ln Z (log-evidence) is sensitive to prior selection, so model acceptance and hence the chosen model depend on prior choices for the activity periods
- Purely statistical inference might be insufficient on its own for robust exoplanet detection; activity signals should be cross-checked against activity indices
- reddemcee's README notes it 'works as a stand-alone program' but the README may lag the documentation ('this readme might not be updated')

## Evidence

- [abstract] the stellar rotation and magnetic cycle periods were recovered, while demonstrating the sensitivity of ln Z to prior selection: "the stellar rotation, P rot = 29.72 −0.02 + 0.01 , and magnetic cycle, P mag = 2557.0 −36.7 + 70.1 , were recovered, while demonstrating the sensitivity of ln 𝒵 to prior selection."
- [methods] working directory with datafiles and datalogs subfolders: "We need to set up our working directory with two subfolders, `datafiles` and `datalogs`, the former for data input, the later for output."
- [methods] engine configuration for reddemcee parallel tempering: "sim.set_engine('reddemcee')
sim.engine_config['setup'] = [10, 256, 2048, 1]"
- [methods] model comparison statistic and tolerance control: "The comparison statistic can be set with the `set_comparison_criteria` method, and the tolerance changed with the `set_tolerance` method."
- [readme] reddemcee is the adaptive parallel tempering sampler built for EMPEROR: [evidence span withheld — anchor status: external_doc]
