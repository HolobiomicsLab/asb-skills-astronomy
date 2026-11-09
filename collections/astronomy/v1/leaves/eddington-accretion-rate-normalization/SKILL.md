---
name: eddington-accretion-rate-normalization
description: Use when a solved pAGN Thompson or Sirko disk yields Mdot(R) in physical
  units (MSun/yr) and you need a dimensionless, source-independent accretion-rate
  profile or inner-boundary value comparable across black-hole masses.
when_to_use_negative:
- The disk has not been solved yet — Mdot(R) only exists after tho.solve_disk(N=1e4);
  do not normalize unsolved or partially converged objects.
- You need absolute luminosities or thermal emission spectra rather than accretion-rate
  ratios; use pagn's calculate_lum_sum (Eq. 47 sums) instead.
- The configuration uses a custom Rout/Mdot_out/Rin triple rather than package defaults,
  in which case the reference normalization 26.29148 MSun/yr and the boundary values
  above do not apply without recomputation.
tools:
- name: pagn
  role: Provides Thompson.ThompsonAGN, solve_disk(N=1e4), plot_mdot(), and save()
    used to produce and normalize the Mdot(R) profile
  repo: https://gitlab.com/aatrani/pagn
- name: numpy
  role: Array handling for the radial Mdot(R) profile extracted from the solved disk
    object
- name: matplotlib
  role: Visualization of the normalized accretion-rate profile via tho.plot_mdot()
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stae1117
    title: '<tt>pAGN</tt> : the one-stop solution for AGN disc modelling'
  source_content_hash: sha256:1f1318095b32ea358045f97574cabffac2817c95dd80e984604b89e53d2b20b6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - matplotlib
  - numpy
  - pagn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# eddington-accretion-rate-normalization

## Summary

Normalizes the radially varying accretion-rate profile Mdot(R) of a solved pAGN AGN disk by the Eddington accretion rate (Mdot_Edd ≈ 26.29148 MSun/yr for the Thompson et al. 2005 configuration), yielding a dimensionless profile anchored to boundary values such as Mdot(Rin) = 1.831417e-01 Mdot_Edd.

## When to use

Use when a solved pAGN Thompson or Sirko disk yields Mdot(R) in physical units (MSun/yr) and you need a dimensionless, source-independent accretion-rate profile or inner-boundary value comparable across black-hole masses.

## When NOT to use

- The disk has not been solved yet — Mdot(R) only exists after tho.solve_disk(N=1e4); do not normalize unsolved or partially converged objects.
- You need absolute luminosities or thermal emission spectra rather than accretion-rate ratios; use pagn's calculate_lum_sum (Eq. 47 sums) instead.
- The configuration uses a custom Rout/Mdot_out/Rin triple rather than package defaults, in which case the reference normalization 26.29148 MSun/yr and the boundary values above do not apply without recomputation.

## Inputs

- pagn Thompson.ThompsonAGN configuration (sigma = 300 km/s, Mbh = 2e8 MSun * (sigma/200 km/s)^4, epsilon = 1e-3, m = 0.2, xi = 1.0, Mdot_out/Rout/Rin = None)
- Solved disk object from tho.solve_disk(N=1e4) with Mdot(R) array in MSun/yr
- Mdot_Edd ≈ 26.29148 MSun/yr for this configuration

## Outputs

- Dimensionless accretion-rate profile Mdot(R)/Mdot_Edd over the disk grid
- Boundary values: Mdot_c at Rout (≈9.606143e+03 MSun/yr), Mdot_out (≈8.311237e+03 MSun/yr), Mdot at Rin (≈4.815066 MSun/yr = 1.831417e-01 Mdot_Edd)
- Saved parameter file thompson_mdot.txt via tho.save()

## How to apply

Instantiate Thompson.ThompsonAGN with the documented example configuration (sigma = 300 km/s, Mbh from the M-sigma relation as 2e8 MSun * (sigma/200 km/s)^4, epsilon = 1e-3, m = 0.2, xi = 1.0, Mdot_out/Rout/Rin = None so package defaults/scalings apply), then solve the disk with tho.solve_disk(N=1e4), since 10^4 grid points is deemed sufficient resolution for most Mbh values. Extract Mdot(R) from the solved object and divide by the Eddington accretion rate (≈26.29148 MSun/yr for this configuration). Report the boundary values: Mdot_c at r = Rout (≈9.606143e+03 MSun/yr), Mdot_out (≈8.311237e+03 MSun/yr), and Mdot at r = Rin (≈4.815066 MSun/yr = 1.831417e-01 Mdot_Edd). Optionally visualize with tho.plot_mdot() and persist with the save() helper to a txt file such as thompson_mdot.txt. Rationale: normalizing by Mdot_Edd makes the inner-boundary accretion rate interpretable as a fraction of Eddington, here ~5 MSun/yr, deemed sufficient to fuel a bright AGN.

## Related tools

- **pagn** (Provides Thompson.ThompsonAGN, solve_disk(N=1e4), plot_mdot(), and save() used to produce and normalize the Mdot(R) profile) — https://gitlab.com/aatrani/pagn
- **numpy** (Array handling for the radial Mdot(R) profile extracted from the solved disk object)
- **matplotlib** (Visualization of the normalized accretion-rate profile via tho.plot_mdot())

## Examples

```
tho = Thompson.ThompsonAGN(Mbh=Mbh*ct.MSun, Rout=rout, Mdot_out=Mdot_out*ct.MSun/ct.yr); tho.solve_disk(N=1e4)
```

## Evaluation signals

- Computed Mdot at the inner boundary must match the documented reference value 4.815066 MSun/yr within numerical tolerance
- The normalized inner accretion rate must equal 1.831417e-01 Mdot_Edd, with Mdot_Edd ≈ 26.29148 MSun/yr for this configuration
- Mdot_out (≈8.311237e+03 MSun/yr) must fall below Mdot_c at the outer boundary (≈9.606143e+03 MSun/yr)
- solve_disk must run at N=1e4 grid points; 10^4 is documented as sufficient resolution for most Mbh values
- The saved thompson_mdot.txt must contain the profile and all three boundary values

## Limitations

- The pAGN GitHub repository (DariaGangardt/pAGN) is archived and unmaintained; the project moved to https://gitlab.com/aatrani/pagn.
- The Mdot_Edd normalization constant (≈26.29148 MSun/yr) is configuration-specific (sigma = 300 km/s, Mbh ≈ derived from M-sigma) and must be recomputed for other black-hole masses.
- The Thompson model instantiation relies on package defaults/scalings when Mdot_out/Rout/Rin are passed as None; overriding these changes the boundary normalization.

## Evidence

- [methods] solve_disk instantiation: "tho = Thompson.ThompsonAGN(Mbh = mbh, sigma = sigma, epsilon = epsilon, m = m, xi = xi,
                      Mdot_out= Mdot_out, Rout = Rout, Rin = Rin, opacity =opacity)"
- [methods] grid resolution: "sk.solve_disk(N=1e4) ; #10^4 tends to be a sufficient resolution for most Mbh values"
- [results] solved Thompson disk configuration: "Mdot_out = 1.5e-2
            obj = Thompson.ThompsonAGN(Mbh=Mbh*ct.MSun, Rout = rout, Mdot_out=Mdot_out*ct.MSun/ct.yr)"
- [methods] saving results: "One may also wish to save their results in a txt file."
- [readme] package scope: "`pagn` is a Python module to solve the equations from [Sirko and Goodman 2003]"
