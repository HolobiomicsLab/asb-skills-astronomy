---
name: posterior-uncertainty-table-generation
description: Use when an EMPEROR/reddemcee parallel-tempering run has finished (sim.autorun)
  and you need to extract median parameter values with asymmetric +/− uncertainties
  from datalogs into a publication-ready posterior parameter table.
when_to_use_negative:
- Input is raw radial-velocity time series that has not yet been run through an EMPEROR
  Simulation (load_data/set_engine/add_condition) — there are no posteriors to tabulate
  yet
- The analysis goal is a point-estimate orbit fit without Bayesian model comparison
  or uncertainty quantification (e.g. a quick chi-square fit)
- The MCMC chains have not converged (no autocorrelation/convergence check performed),
  so tabulated uncertainties would be meaningless
tools:
- name: EMPEROR (astroemperor)
  role: Produces the datalogs outputs (best_fit.dat, values.tex, stats.tex) whose
    posteriors are compiled into the parameter table
  repo: github.com/ReddTea/astroemperor
- name: reddemcee
  role: Adaptive parallel-tempering MCMC engine whose sampled chains provide the posterior
    distributions and ln Z evidence values tabulated
  repo: github.com/ReddTea/reddemcee
- name: emcee
  role: Underlying ensemble sampler used by reddemcee; also used for calculating autocorrelation
    times when judging chain convergence before tabulation
  repo: https://github.com/dfm/emcee
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:8e0631f8f9aa41c0f7babed85a121dc70fbba62b932b94b7730edc2d9e0f5bfd
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
yes_rate: 0.5
n_claims: 8
---

# posterior-uncertainty-table-generation

## Summary

Compiles MCMC posterior estimates and their asymmetric credible intervals (e.g. P_rot = 29.72 (+0.01/−0.02) d, P_mag = 2557.0 (+70.1/−36.7) d for HD 55693) into a final parameter table from EMPEROR's datalogs outputs (best_fit.dat, values.tex, stats.tex).

## When to use

Use when an EMPEROR/reddemcee parallel-tempering run has finished (sim.autorun) and you need to extract median parameter values with asymmetric +/− uncertainties from datalogs into a publication-ready posterior parameter table.

## When NOT to use

- Input is raw radial-velocity time series that has not yet been run through an EMPEROR Simulation (load_data/set_engine/add_condition) — there are no posteriors to tabulate yet
- The analysis goal is a point-estimate orbit fit without Bayesian model comparison or uncertainty quantification (e.g. a quick chi-square fit)
- The MCMC chains have not converged (no autocorrelation/convergence check performed), so tabulated uncertainties would be meaningless

## Inputs

- EMPEROR datalogs output folder (best_fit.dat, values.tex, stats.tex) from an autorun on an RV dataset (e.g. HD 55693 RV catalogue under datafiles/HD55693/RV)
- reddemcee parallel-tempering chain configuration engine_config['setup'] = [ntemps, nwalkers, nsweeps, nsteps]
- Prior/limit definitions entered via add_condition (period limits, priors, initial positions)

## Outputs

- Final parameter table of posterior estimates with asymmetric credible intervals (e.g. P_rot = 29.72 (+0.01/-0.02) d; P_mag = 2557.0 (+70.1/-36.7) d)
- LaTeX-ready values.tex / stats.tex entries for each model parameter
- Log-evidence (ln Z) per model for the accepted optimum model

## How to apply

After the EMPEROR Simulation completes autorun over increasing Keplerian/model complexity (with set_comparison_criteria('Evidence') and set_tolerance(10) so models are accepted only when log-evidence improves by more than the tolerance), read the datalogs output folder. Parse best_fit.dat for the best-fit parameters, and values.tex / stats.tex for posterior medians and credible-interval uncertainties, choosing asymmetric (+upper/−lower) error bars as reported for P_rot = 29.72 (+0.01/−0.02) d and P_mag = 2557.0 (+70.1/−36.7) d. Compile each recovered signal (Keplerian, sinusoid, magnetic cycle, stellar activity index terms, offsets and jitter) into a parameter table listing period, amplitude, and nuisance parameters with uncertainties. Cross-check that the recovered posteriors fall inside the priors set via add_condition (e.g. ~29.72 d rotation, ~2557 d magnetic cycle), since the paper shows ln Z is sensitive to prior selection — flag any posterior pinned at a prior boundary.

## Related tools

- **EMPEROR (astroemperor)** (Produces the datalogs outputs (best_fit.dat, values.tex, stats.tex) whose posteriors are compiled into the parameter table) — github.com/ReddTea/astroemperor
- **reddemcee** (Adaptive parallel-tempering MCMC engine whose sampled chains provide the posterior distributions and ln Z evidence values tabulated) — github.com/ReddTea/reddemcee
- **emcee** (Underlying ensemble sampler used by reddemcee; also used for calculating autocorrelation times when judging chain convergence before tabulation) — https://github.com/dfm/emcee

## Examples

```
import astroemperor; sim = astroemperor.Simulation(); sim.load_data('HD55693'); sim.set_engine('reddemcee'); sim.set_comparison_criteria('Evidence'); sim.set_tolerance(10); sim.autorun()
```

## Evaluation signals

- Recovered posterior periods must match the reported values within their credible intervals: P_rot = 29.72 (+0.01/-0.02) d and P_mag = 2557.0 (+70.1/-36.7) d for the HD 55693 validation case
- best_fit.dat, values.tex, and stats.tex files exist in the datalogs output subfolder after autorun completes
- Each successive accepted model improved ln Z by more than the set tolerance (set_tolerance(10))
- Tabulated uncertainties are asymmetric (+upper/−lower) and posteriors are not pinned at add_condition prior boundaries (given demonstrated ln Z sensitivity to prior selection)

## Limitations

- ln Z (and hence which model's posteriors to tabulate) is sensitive to prior selection, so tables built from a poorly chosen prior may reflect the prior, not the data
- Purely statistical inference might be insufficient on its own for robust exoplanet detection — the table should be interpreted alongside activity diagnostics
- reddemcee/kepler.py caveats: kepler.solve returns garbage for eccentricities outside 0–1, and the README warns its documentation may be more current than the README

## Evidence

- [methods] Posterior periods must match P_rot = 29.72 (+0.01/-0.02) d and P_mag = 2557.0 (+70.1/-36.7) d within their credible intervals.: [evidence span withheld — anchor status: not_found]
- [other] datalogs output folder contains best_fit.dat, values.tex, stats.tex: [evidence span withheld — anchor status: not_found]
- [abstract] Article abstract reports the recovered HD 55693 activity periods and ln Z prior sensitivity.: "the stellar rotation, P rot = 29.72 −0.02 + 0.01 , and magnetic cycle, P mag = 2557.0 −36.7 + 70.1 , were recovered, while demonstrating the sensitivity of ln 𝒵 to prior selection"
- [methods] Model comparison criteria and tolerance govern which run's posteriors are final.: "The comparison statistic can be set with the `set_comparison_criteria` method, and the tolerance changed with the `set_tolerance` method."
- [readme] Working-directory layout requirement for datalogs outputs.: "We need to set up our working directory with two subfolders, `datafiles` and `datalogs`, the former for data input, the later for output."
