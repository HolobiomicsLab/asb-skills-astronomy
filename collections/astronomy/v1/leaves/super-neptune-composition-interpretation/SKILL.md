---
name: super-neptune-composition-interpretation
description: 'Use when a confirmed transiting super-Neptune has published Rp and Mp
  (e.g. TOI-5005 b: 6.25 ± 0.24 R⊕, 32.7 ± 5.9 M⊕) and you must interpret its density,
  core/metal fraction, or compare it against the savanna vs ridge population regimes.'
when_to_use_negative:
- Planet radius falls outside the super-Neptune regime (not a Neptunian-ridge/savanna
  object), so the period-radius population comparison is meaningless
- No published dynamical or transit-derived mass exists, so rho = 3M/(4*pi*R^3) and
  the density-based regime assignment cannot be computed
- The goal is photometric vetting of a TESS candidate (e.g. flux contamination, CROWDSAP/FLFRCSAP
  metrics) rather than composition interpretation
edam_operation: http://edamontology.org/operation_2945
tools:
- name: Python
  role: Implements the density computation with uncertainty propagation and the Mann-Whitney
    U / rank-sum test on the savanna vs ridge density distributions; the paper's own
    TESS-cont package is also Python-based
  repo: github.com/castro-gzlz/TESS-cont
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:68ff79c8cdcfbf77ead03865f257847b20dc54c5d1d1e64ca7cd8727d38ba7e9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# super-neptune-composition-interpretation

## Summary

Interprets the bulk composition of a transiting super-Neptune by computing its mean density from published mass and radius, situating it in the Neptunian ridge/savanna population (dividing line ~1 g cm−3, p-value = 0.0092), and constraining metal mass fraction, formation pathway, and atmospheric mass loss.

## When to use

Use when a confirmed transiting super-Neptune has published Rp and Mp (e.g. TOI-5005 b: 6.25 ± 0.24 R⊕, 32.7 ± 5.9 M⊕) and you must interpret its density, core/metal fraction, or compare it against the savanna vs ridge population regimes.

## When NOT to use

- Planet radius falls outside the super-Neptune regime (not a Neptunian-ridge/savanna object), so the period-radius population comparison is meaningless
- No published dynamical or transit-derived mass exists, so rho = 3M/(4*pi*R^3) and the density-based regime assignment cannot be computed
- The goal is photometric vetting of a TESS candidate (e.g. flux contamination, CROWDSAP/FLFRCSAP metrics) rather than composition interpretation

## Inputs

- Published planet mass and radius with uncertainties (e.g. Mp = 32.7 ± 5.9 M⊕, Rp = 6.25 ± 0.24 R⊕)
- NASA Exoplanet Archive planetary parameters table for confirmed transiting super-Neptunes
- Period and radius values for ridge/savanna classification in the period-radius diagram
- Stellar metallicity for computing the Zplanet/Zstar ratio
- Photoevaporation model parameters for atmospheric mass-loss estimation

## Outputs

- Mean planetary density in g cm−3 with propagated uncertainty (TOI-5005 b: 0.74 ± 0.16 g cm−3)
- Population label (Neptunian ridge or savanna) per planet
- Mann-Whitney U p-value for savanna vs ridge density difference (target: 0.0092)
- Empirical density dividing line between regimes (~1 g cm−3)
- Core/metal mass fraction from internal structure modelling (Zplanet = 0.76)
- Atmospheric mass-loss rate with model-dependence range (0.013 ± 0.008 vs 0.17 ± 0.12 M⊕ Gyr−1)
- Single CSV listing each planet's population label, mass, radius, and computed mean density with test statistics

## How to apply

Compute the mean density as rho = 3M / (4*pi*R^3), propagating published mass and radius uncertainties (for TOI-5005 b this yields 0.74 ± 0.16 g cm−3). Classify the planet as Neptunian ridge or savanna from its position in the period-radius diagram, then run a nonparametric rank-based test (Mann-Whitney U / rank-sum) on savanna vs ridge density distributions to check whether the planet sits in the low-density savanna regime, whose dividing line from the ridge falls near 1 g cm−3 (reported p-value = 0.0092). Perform internal structure modelling to constrain the metal mass fraction (TOI-5005 b: Zplanet = 0.76, slightly below Neptune and Uranus) and compute the Zplanet/Zstar ratio to test consistency with the mass-metallicity relation, which indicates formation via core accretion. Estimate the present-day atmospheric mass-loss rate with photoevaporation models, explicitly comparing alternative models since predictions can differ by more than an order of magnitude. Write results (population label, mass, radius, density, p-value, dividing-line density) to a CSV for reproducibility.

## Related tools

- **Python** (Implements the density computation with uncertainty propagation and the Mann-Whitney U / rank-sum test on the savanna vs ridge density distributions; the paper's own TESS-cont package is also Python-based) — github.com/castro-gzlz/TESS-cont

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(savanna_densities, ridge_densities, alternative='two-sided')
```

## Evaluation signals

- Computed density of TOI-5005 b matches 0.74 ± 0.16 g cm−3 from the published Rp and Mp
- Reproduced Mann-Whitney U p-value matches the reported 0.0092 within the tolerance implied by sample reproducibility
- Empirical dividing line between savanna and ridge density distributions falls near 1 g cm−3
- Internal structure modelling returns an overall metal mass fraction slightly below Neptune and Uranus (Zplanet ≈ 0.76)
- Output CSV contains every planet's population label, mass, radius, mean density, the p-value, and the dividing-line density

## Limitations

- Atmospheric mass-loss predictions are strongly model-dependent: photoevaporation models gave contrasting rates (0.013 ± 0.008 vs 0.17 ± 0.12 M⊕ Gyr−1), so a single model's rate should not be quoted as definitive
- The savanna-vs-ridge p-value match is only reproducible within the tolerance implied by the population sample; differences in the retrieved planetary parameters table will shift the statistic
- Density-based regime assignment depends on the position in the period-radius diagram, which itself relies on the classification boundaries used to build the population sample

## Evidence

- [abstract] Population-level savanna vs ridge density comparison: [evidence span withheld — anchor status: not_found]
- [abstract] Metal mass fraction constraint: [evidence span withheld — anchor status: not_found]
- [abstract] Formation pathway interpretation: "The Z planet / Z star ratio is consistent with the well-known mass-metallicity relation, which suggests that TOI-5005 b was formed via core accretion"
- [abstract] Mass-loss model dependence: "found contrasting predictions depending on the choice of photoevaporation model (0.013 ± 0.008 M ⊕ Gyr −1 vs. 0.17 ± 0.12 M ⊕ Gyr −1 )"
- [abstract] Reference planet parameters: [evidence span withheld — anchor status: not_found]
