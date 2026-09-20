---
name: exoplanet-model-comparison-bic-evidence
description: Use when an RV time-series analysis must decide how many Keplerian (planet)
  signals to retain, i.e. successive EMPEROR autorun models (k = 1, 2, ...) need ranking
  by Bayesian evidence (log Z) or BIC under a stopping tolerance.
when_to_use_negative:
- Only a single fixed planet count is being fitted and no comparison among nested
  Keplerian models is needed — the autorun evidence loop adds runtime without decision
  value.
- 'The runs to be compared were configured with different priors: the paper shows
  ln Z is sensitive to prior selection (HD 55693), so BIC/ln Z comparisons across
  mismatched priors are invalid.'
- The data are not radial-velocity (or joint RV–astrometry) time series within EMPEROR's
  Keplerian-signal model space.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: EMPEROR (astroemperor)
  role: Hosts the autorun model-comparison loop; set_comparison_criteria('Evidence')
    and set_tolerance(10) control how successive Keplerian models are ranked and when
    the sweep stops.
  repo: https://github.com/ReddTea/astroemperor
- name: reddemcee
  role: Adaptive Parallel Tempering (APT) sampling engine whose per-model ln Z feeds
    the evidence comparison; exposes get_evidence_ti, get_evidence_ss, and get_evidence_hybrid
    for thermodynamic integration, stepping stones, and hybrid estimates.
  repo: https://github.com/ReddTea/reddemcee
- name: dynesty
  role: DNS (dynamic nested sampling) engine run on the identical model space to provide
    the baseline ln Z/BIC and effective-sampling reference against APT.
- name: kepler.py
  role: Fast Kepler's-equation solver (mean anomaly, eccentricity → true-anomaly cos/sin)
    underlying the Keplerian forward model whose per-model evidence is compared.
  repo: https://github.com/dfm/kepler.py
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202554336
    title: EMPEROR
  source_content_hash: sha256:c7fc7b0277d2713f88367c37269370af1ea9bd813ad8035edfa7afef35898ca8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ReddTea/astroemperor
  tools:
  - EMPEROR (astroemperor)
  - Python
  - dynesty
  - emcee
  - kepler.py
  - reddemcee
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# exoplanet-model-comparison-bic-evidence

## Summary

Automatic model-selection skill for radial-velocity exoplanet searches: EMPEROR's autorun loop compares successive k-Keplerian models via Bayesian evidence (ln Z, estimated by Curvature-aware Thermodynamic Integration TI+, Geometric-Bridge Stepping Stones SS+, or a Hybrid algorithm) or BIC, with a configurable tolerance deciding whether an additional planet signal is retained.

## When to use

Use when an RV time-series analysis must decide how many Keplerian (planet) signals to retain, i.e. successive EMPEROR autorun models (k = 1, 2, ...) need ranking by Bayesian evidence (log Z) or BIC under a stopping tolerance.

## When NOT to use

- Only a single fixed planet count is being fitted and no comparison among nested Keplerian models is needed — the autorun evidence loop adds runtime without decision value.
- The runs to be compared were configured with different priors: the paper shows ln Z is sensitive to prior selection (HD 55693), so BIC/ln Z comparisons across mismatched priors are invalid.
- The data are not radial-velocity (or joint RV–astrometry) time series within EMPEROR's Keplerian-signal model space.

## Inputs

- Radial-velocity time-series catalogue, e.g. 51peg.vels (LICK) placed under datafiles/51Peg/RV/
- EMPEROR Simulation object with sampling engine set (reddemcee APT or dynesty DNS) and engine_config['setup'] = [10, 256, 2048, 1]
- Per-parameter boundary conditions from add_condition (limits, init_pos), e.g. Period 1 limits [3, 5] d with init_pos [4.1, 4.3]; Amplitude 1 [45, 60] m/s with init_pos [50, 60]
- Comparison settings: set_comparison_criteria('Evidence' or 'BIC') and set_tolerance(10)

## Outputs

- stats.tex and values.tex tables per model k under datalogs/{system}/run_1/k_{k}/tables containing ln Z (Evidence), BIC, and posterior parameter estimates
- Automatically selected optimum Keplerian model count from the autorun loop
- Cross-engine comparison table (e.g. 51peg_apt_dns_comparison.csv) with effective sampling efficiency and the APT/DNS ratio (~3.76 for 51 Pegasi)

## How to apply

Set up the working directory with datafiles/ and datalogs/ subfolders, place the RV catalogue (e.g. 51peg.vels, LICK) under datafiles/51Peg/RV/, and load it with sim.load_data('51Peg'). Configure the sampler with sim.set_engine('reddemcee') and sim.engine_config['setup'] = [10, 256, 2048, 1], then constrain the parameter space via add_condition — for 51 Pegasi: 'Period 1' limits [3, 5] d with init_pos [4.1, 4.3] and 'Amplitude 1' limits [45, 60] m/s with init_pos [50, 60]. Run the automatic sweep with sim.autorun(1, 1) after set_comparison_criteria('Evidence') and set_tolerance(10), so each added Keplerian is accepted or rejected by its log Z (or BIC) gain against the tolerance. Evidence comes from reddemcee's thermodynamic integration, stepping-stones, or hybrid estimators (get_evidence_ti/ss/hybrid); extract ln Z, BIC, and posterior parameter estimates from each run's stats.tex and values.tex under datalogs/51Peg/run_1/k_1/tables. To benchmark samplers, repeat the identical model space with the DNS (dynesty) engine — for 51 Pegasi this reproduces APT's 3.76x effective sampling increase over DNS with tighter parameter posteriors. Keep priors identical across compared runs, since the paper demonstrates the sensitivity of ln Z to prior selection on HD 55693.

## Related tools

- **EMPEROR (astroemperor)** (Hosts the autorun model-comparison loop; set_comparison_criteria('Evidence') and set_tolerance(10) control how successive Keplerian models are ranked and when the sweep stops.) — https://github.com/ReddTea/astroemperor
- **reddemcee** (Adaptive Parallel Tempering (APT) sampling engine whose per-model ln Z feeds the evidence comparison; exposes get_evidence_ti, get_evidence_ss, and get_evidence_hybrid for thermodynamic integration, stepping stones, and hybrid estimates.) — https://github.com/ReddTea/reddemcee
- **dynesty** (DNS (dynamic nested sampling) engine run on the identical model space to provide the baseline ln Z/BIC and effective-sampling reference against APT.)
- **kepler.py** (Fast Kepler's-equation solver (mean anomaly, eccentricity → true-anomaly cos/sin) underlying the Keplerian forward model whose per-model evidence is compared.) — https://github.com/dfm/kepler.py

## Examples

```
sim.load_data('51Peg'); sim.set_engine('reddemcee'); sim.engine_config['setup'] = [10, 256, 2048, 1]; sim.set_comparison_criteria('Evidence'); sim.set_tolerance(10); sim.autorun(1, 1)
```

## Evaluation signals

- Every compared model k has stats.tex and values.tex under datalogs/{system}/run_1/k_{k}/tables reporting both ln Z and BIC.
- Acceptance of each added Keplerian is decided by the evidence gain against the configured tolerance (set_tolerance(10)); the stopping decision is traceable in datalogs/.
- Cross-engine check: recomputed effective sampling efficiency for 51 Pegasi yields an APT/DNS ratio close to the reported 3.76, with APT showing tighter posterior parameter intervals.
- For the same model, evidence estimates agree across thermodynamic integration, stepping stones, and hybrid estimators (reddemcee get_evidence_ti/ss/hybrid).
- Recovered Period 1 posterior for 51 Pegasi falls inside the imposed limits [3, 5] d near init_pos [4.1, 4.3].

## Limitations

- ln Z is sensitive to prior selection — demonstrated on HD 55693 (P_rot = 29.72 d, P_mag = 2557.0 d), so evidence comparisons require matched priors.
- Purely statistical inference might be insufficient on its own for robust exoplanet detection; BIC/ln Z verdicts still need physical vetting.
- Effective sampling efficiency is engine-dependent: DNS (dynesty) is ~3.76x less effective than APT on 51 Pegasi, affecting runtime and evidence precision.
- kepler.py's solve interface returns garbage for eccentricities outside the range zero to one, bounding usable eccentricity priors in the compared models.

## Evidence

- [abstract] APT yields a 3.76x effective sampling gain over DNS on 51 Pegasi with tighter posteriors: "For 51 Pegasi, we find APT achieves an effective sampling increase by a factor of 3.76 over DNS, while retrieving tighter parameter estimates."
- [abstract] ln Z is sensitive to prior selection: "while demonstrating the sensitivity of ln 𝒵 to prior selection."
- [intro] Autorun performs the model comparisons automatically to return the optimum model: "performs model comparisons automatically to return the optimum model that best describes the data"
- [methods] Comparison statistic and tolerance are configurable via set_comparison_criteria and set_tolerance: "The comparison statistic can be set with the `set_comparison_criteria` method, and the tolerance changed with the `set_tolerance` method."
- [methods] Evidence is estimated by TI+, SS+, or hybrid algorithms: "Curvature-aware Thermodynamic Integration (TI+), Geometric-Bridge Stepping Stones (SS+), and a Hybrid algorithm"
- [readme] EMPEROR offers multiple sampling routines aimed at multi-modal posteriors: "A number of posterior sampling routines are available, focused on efficiently searching for signals in highly multi-modal posteriors."
- [readme] kepler.py solve is invalid for eccentricities outside [0, 1]: [evidence span withheld — anchor status: external_doc]
