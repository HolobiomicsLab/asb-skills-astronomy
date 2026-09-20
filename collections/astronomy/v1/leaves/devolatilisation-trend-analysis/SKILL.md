---
name: devolatilisation-trend-analysis
description: Use when you have stellar elemental abundance patterns for F/G/K stars
  and need to compute per-element condensation temperatures and planet bulk compositions/devolatilisation
  trends, validating synthetic abundance-pattern effects against real solar-neighbourhood
  data.
when_to_use_negative:
- Stellar abundance data is very incomplete — the README warns to consider 'the sensibility
  of running simulations on very incomplete abundance data sets (e.g. if Al, Ca, Mg,
  Si, or O data is not reported)'
- You need solid-solution mineralogy or stellar types beyond F/G/K — the first release
  only includes pure minerals and Brewer+2016 F, G, K star abundances
- The goal is gas-phase-only chemistry — the TEA code is noted as the tool limited
  to gas-phase simulations, whereas this skill targets condensation into minerals
  and planet compositions
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0621
tools:
- name: ECCOplanets
  role: Open-source Python equilibrium condensation code providing the simulation,
    molecule, and planet classes used to compute condensation temperatures and planet
    compositions
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Language in which the code is built, together with pandas, numpy and matplotlib
    for data handling and plotting
- name: pandas
  role: Common package used by the code for reading the .pkl thermochemical and stellar
    abundance databases
- name: numpy
  role: Common package used for the numerical Gibbs-free-energy-minimisation computation
- name: matplotlib
  role: Plotting of molecule-vs-temperature curves and planet composition results
    (figure style via pltStyle.py)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:6396da1aead40909e11de1539d282ed877b15e2462a5c39da02e00c91530022a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AninaTimmermann/ECCOplanets
  tools:
  - ECCOplanets
  - Python
  - matplotlib
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# devolatilisation-trend-analysis

## Summary

Derives rocky planet bulk compositions and devolatilisation patterns relative to host stars by extracting 50% condensation temperatures (Tc) of planet-building elements from equilibrium condensation simulations, then verifying abundance-pattern trends (Tc rising with metallicity, ~-100 K shift for C/O from 0.1 to 0.7) against real stellar data from the Brewer+2016 solar-neighbourhood sample.

## When to use

Use when you have stellar elemental abundance patterns for F/G/K stars and need to compute per-element condensation temperatures and planet bulk compositions/devolatilisation trends, validating synthetic abundance-pattern effects against real solar-neighbourhood data.

## When NOT to use

- Stellar abundance data is very incomplete — the README warns to consider 'the sensibility of running simulations on very incomplete abundance data sets (e.g. if Al, Ca, Mg, Si, or O data is not reported)'
- You need solid-solution mineralogy or stellar types beyond F/G/K — the first release only includes pure minerals and Brewer+2016 F, G, K star abundances
- The goal is gas-phase-only chemistry — the TEA code is noted as the tool limited to gas-phase simulations, whereas this skill targets condensation into minerals and planet compositions

## Inputs

- Brewer+2016 stellar abundance database (.pkl, quality-selected F/G/K subsample with S/N > 100, log g > 3.5)
- ECCOplanets thermochemical databases (.pkl)
- Condensation simulation parameters: temperature range and resolution, molecule selection, star-specific elemental abundance pattern

## Outputs

- Condensation temperatures (50% condensation points) of planet-building elements per star
- Molecule-vs-temperature condensation curves (improved: cropped, recomputed, smoothed)
- Rocky planet bulk compositions as a function of temperature and devolatilisation relative to each host star

## How to apply

Load the Brewer+2016 stellar abundance patterns for the quality-selected subsample (S/N > 100, log g > 3.5, ~100 F, G and K stars) via data.py, which reads the packaged .pkl thermochemical and stellar abundance databases. For each star, run a condensation simulation with simulation.py (setting temperature range and resolution, molecule selection, and the star-specific elemental abundance pattern), then improve the result using the classes.py simulation class: crop the gas phase, recompute the affected section to remove numerical glitches, and smooth the curve. Extract the 50% condensation point of each planet-building element from the molecule-vs-temperature curves, and compute rocky planet bulk compositions as a function of temperature and devolatilisation relative to each host star with the planet class (sample s3planet_composition.py). Validate by checking that derived Tc values for common planet-building elements agree with literature within ±5 K and that the trends found with synthetic patterns (rising Tc with metallicity; ∆Tc ≈ -100 K for C/O increase from 0.1 to 0.7) are reproduced by the real stellar data.

## Related tools

- **ECCOplanets** (Open-source Python equilibrium condensation code providing the simulation, molecule, and planet classes used to compute condensation temperatures and planet compositions) — https://github.com/AninaTimmermann/ECCOplanets
- **Python** (Language in which the code is built, together with pandas, numpy and matplotlib for data handling and plotting)
- **pandas** (Common package used by the code for reading the .pkl thermochemical and stellar abundance databases)
- **numpy** (Common package used for the numerical Gibbs-free-energy-minimisation computation)
- **matplotlib** (Plotting of molecule-vs-temperature curves and planet composition results (figure style via pltStyle.py))

## Examples

```
run s1simulation.py with a star-specific abundance pattern, then in s3planet_composition.py use the planet class to compute bulk composition and devolatilisation relative to the host star (see samples s1simulation.py → s2analyses.py → s3planet_composition.py in the ECCOplanets repo)
```

## Evaluation signals

- Derived condensation temperatures for common planet-building elements agree with literature values within ±5 K
- Real stellar data reproduce the synthetic-pattern trend that Tc rises with metallicity (~∆Tc ≈ +100 K for -0.3 ≤ [M/H] ≤ 0.4)
- Real stellar data reproduce the C/O sensitivity: increase in C/O from 0.1 to 0.7 gives ∆Tc ≈ -100 K
- Improved simulation curves no longer show numerical glitches after cropping the gas phase, recomputing the affected section, and smoothing
- Extracted 50% condensation points are consistent across the ~100-star quality-selected Brewer+2016 subsample (S/N > 100, log g > 3.5)

## Limitations

- Only pure minerals are considered at the moment; solid solutions are planned for a later release
- First release includes only F, G, and K star abundance data from the Brewer+2016 database; other stellar types require added data and consideration of uncertainties
- Simulations on very incomplete abundance sets (missing Al, Ca, Mg, Si, or O) may not be sensible
- Condensation temperatures depend on assumed disk pressure; the tested 10^-6 - 0.1 bar range alone corresponds to ∆Tc ≈ +350 K, so pressure assumptions strongly affect results

## Evidence

- [abstract] To analyse the influence of the abundance pattern, key element ratios were varied, and the results were validated using solar neighbourhood stars.: "To analyse the influence of the abundance pattern, key element ratios were varied, and the results were validated using solar neighbourhood stars."
- [abstract] Tc within ±5 K of literature values: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [abstract] C/O increase reduces Tc by ~100 K: "An increase in C/O from 0.1 to 0.7 results in a decrease of ∆Tc ≈ −100 K. Other element ratios are less influential."
- [readme] computing the composition of rocky planets as a function of temperature and analysing these planets, with regard to the condensation temperature of elements, bulk composition and devolatilisation: "computing the composition of rocky planets as a function of temperature and analysing these planets, with regard to the condensation temperature of elements, bulk composition and devolatilisation"
- [readme] improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [readme] incomplete abundance data caveat: "Also you might need to think about the sensibility of running simulations on very incomplete abundance data sets (e.g. if Al, Ca, Mg, Si, or O data is not reported)."
