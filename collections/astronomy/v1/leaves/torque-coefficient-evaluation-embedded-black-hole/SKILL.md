---
name: torque-coefficient-evaluation-embedded-black-hole
description: Use when you need migration torque magnitudes or torque coefficients
  versus radius for an embedded object (e.g. a 50 solar-mass BH) in a solved Sirko
  & Goodman or Thompson et al. AGN disk profile from pAGN.
when_to_use_negative:
- You only need a solved AGN disk structure (temperature, density, Q profiles) without
  migration torques — a plain solve_disk/plot of key parameters suffices
- Your disk model is not one of the pAGN-supported models (Sirko & Goodman 2003 or
  Thompson et al. 2005) or lacks the gradients needed by the torque prescriptions
tools:
- name: pagn
  role: Solves the Sirko & Goodman / Thompson et al. AGN disk equations to supply
    the radial profiles used by the torque prescriptions
  repo: https://gitlab.com/aatrani/pagn
- name: SciPy
  role: UnivariateSpline interpolation of log-radius surface-density and thermal gradients
    for the coefficient prescriptions
- name: numpy
  role: Writes the coefficient-vs-radius table with numpy.savetxt
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:7bf03c9cbfbf3de18ca34598da47c506cf9ce80302d80158e6267b73671a9a72
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SciPy
  - numpy
  - pagn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# torque-coefficient-evaluation-embedded-black-hole

## Summary

Computes migration torques and the dimensionless torque coefficients (CI_jm17_iso, CI_jm17_tot, CI_p10, CL) for a stellar-mass black hole embedded in an AGN disk solved with pAGN, following the Grishin et al. 2023 equations. The total torque is assembled as Gamma_tot = Gamma_therm (Masset 2017) + Gamma_I_jm_tot (Jiménez & Masset 2017).

## When to use

Use when you need migration torque magnitudes or torque coefficients versus radius for an embedded object (e.g. a 50 solar-mass BH) in a solved Sirko & Goodman or Thompson et al. AGN disk profile from pAGN.

## When NOT to use

- You only need a solved AGN disk structure (temperature, density, Q profiles) without migration torques — a plain solve_disk/plot of key parameters suffices
- Your disk model is not one of the pAGN-supported models (Sirko & Goodman 2003 or Thompson et al. 2005) or lacks the gradients needed by the torque prescriptions

## Inputs

- Solved pAGN SirkoAGN or ThompsonAGN disk object with radial profiles from solve_disk(N=1e4) (radius, Omega, temperature, density, scale height, sound speed, optical depth, Q)
- Embedded BH mass and mass ratio q relative to the central BH (e.g. 50 Msun migrator, q = 5e-6 around a 1e6 Msun central BH)
- Log-radius surface-density and thermal gradients to be spline-interpolated (UnivariateSpline k=3, s=0.005, ext=0)

## Outputs

- Radius-indexed table of the four torque coefficients (CI_jm17_iso, CI_jm17_tot, CI_p10, CL) with radius in R/ct.pc, written via numpy.savetxt (e.g. torque_coefficients.csv)
- Total migration torque per radius: Gamma_tot = Gamma_therm + Gamma_I_jm_tot

## How to apply

First instantiate and solve the disk model (e.g. Sirko.SirkoAGN with le=0.5, alpha=0.01, b=0 and sk.solve_disk(N=1e4)) to obtain radial profiles of radius, Omega, temperature, density, scale height, sound speed, optical depth, and Q. Then build UnivariateSpline interpolants (k=3, s=0.005, ext=0) of the log-radius surface-density and thermal gradients — this requires scipy version >= 1.10.0. Evaluate the coefficient prescriptions CI_jm17_iso and CI_jm17_tot (Jiménez & Masset 2017), CI_p10 (Paardekooper et al. 2010), and CL (Masset 2017) using the equations of Grishin et al. 2023, and assemble Gamma_tot = Gamma_therm + Gamma_I_jm_tot for the embedded BH mass (e.g. 50 Msun with mass ratio q = 5e-6). Collect coefficient values against radius in units of R/ct.pc and write the table with numpy.savetxt (e.g. to torque_coefficients.csv).

## Related tools

- **pagn** (Solves the Sirko & Goodman / Thompson et al. AGN disk equations to supply the radial profiles used by the torque prescriptions) — https://gitlab.com/aatrani/pagn
- **SciPy** (UnivariateSpline interpolation of log-radius surface-density and thermal gradients for the coefficient prescriptions)
- **numpy** (Writes the coefficient-vs-radius table with numpy.savetxt)

## Examples

```
sk = Sirko.SirkoAGN(Mbh=Mbh*ct.MSun, le=0.5, alpha=0.01, b=0); sk.solve_disk(N=1e4); np.savetxt('torque_coefficients.csv', coeffs, header='radius,CI_jm17_iso,CI_jm17_tot,CI_p10,CL')
```

## Evaluation signals

- The solved Sirko & Goodman disk shows temperature decreasing outward from ~1e5–1e6 K
- h/r stays below one in the inner disk regions of the solved model
- Output table contains a radius column plus all four coefficient columns (CI_jm17_iso, CI_jm17_tot, CI_p10, CL)
- Coefficients are tabulated against radius expressed in R/ct.pc units
- Disk solution was obtained with N=1e4 grid points (stated as sufficient resolution for most Mbh values)

## Limitations

- UnivariateSpline gradients require scipy version 1.10.0 or newer
- The GitHub repository DariaGangardt/pAGN is archived and unmaintained; use the GitLab repo (gitlab.com/aatrani/pagn) for the latest version
- Custom opacities must be provided in a specific format for SirkoAGN/ThompsonAGN to accept them

## Evidence

- [results] Total torque as sum of thermal and Jiménez & Masset torques: [evidence span withheld — anchor status: not_found]
- [results] Spline interpolation with version constraint: "UnivariateSpline(rlog10, Sigmalog10, k=3, s=0.005, ext=0)  # need scipy ver 1.10.0"
- [methods] Disk solve step: "sk = Sirko.SirkoAGN(Mbh=Mbh, le=le, Mdot=Mdot, alpha=alpha, X=X, 
                    b=b, opacity = Opacity)
sk.solve_disk(N=1e4)"
- [results] Grishin equations used for torque calculation: "We use the equations from
`Grishin et al. 2023 <https://arxiv.org/pdf/2307.07546.pdf>`__"
- [readme] Package availability and scope: "`pagn` is a Python module to solve the equations from [Sirko and Goodman 2003](https://academic.oup.com/mnras/article/341/2/501/1105444) and [Thompson et al."
