---
name: complex-valued-output-parsing
description: Use when a solver's step output (e.g. oscode's 'output.txt') stores numerical
  solution steps as complex-valued pairs on a time grid and downstream plotting or
  comparison needs real arrays of times and solution values.
when_to_use_negative:
- Input is plain numeric (real-valued) text output that genfromtxt can read without
  converters — no complex pair decoding is needed
- Solution values come from pyoscode.solve(...) in Python, which returns solution
  arrays directly rather than writing 'output.txt'
- You need dense-output values at t_eval rather than the solver's internal RK/WKB
  steps
tools:
- name: numpy
  role: parses 'output.txt' via np.genfromtxt with dtype=complex and per-column parse_pair
    converters, delimiter=';'
  repo: https://pypi.org/project/numpy/
- name: oscode
  role: C++ burst example whose compiled run ('./burst') writes the complex step pairs
    to 'output.txt' that this skill parses
  repo: https://github.com/fruzsinaagocs/oscode
- name: matplotlib
  role: consumes the parsed RK and WKB step arrays to reproduce the burst comparison
    figure
  repo: https://pypi.org/project/matplotlib/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1103/physrevresearch.2.013030
    title: Efficient method for solving highly oscillatory ordinary differential equations
      with applications to physical systems
  source_content_hash: sha256:26f04d0f883bec23d597d68010bdd90518bf5c9227bfd3c2f7238e6107faffde
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - Python
  - matplotlib
  - numpy
  - oscode
  - pyoscode
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# complex-valued-output-parsing

## Summary

Parses the semicolon-delimited 'output.txt' produced by the C++ oscode burst example, using numpy's genfromtxt with per-column converters to decode the complex-valued (t, x) step pairs so RK and WKB steps become usable numeric arrays. This is required because oscode writes steps as complex pairs that numpy cannot read natively.

## When to use

Use when a solver's step output (e.g. oscode's 'output.txt') stores numerical solution steps as complex-valued pairs on a time grid and downstream plotting or comparison needs real arrays of times and solution values.

## When NOT to use

- Input is plain numeric (real-valued) text output that genfromtxt can read without converters — no complex pair decoding is needed
- Solution values come from pyoscode.solve(...) in Python, which returns solution arrays directly rather than writing 'output.txt'
- You need dense-output values at t_eval rather than the solver's internal RK/WKB steps

## Inputs

- 'output.txt' from the C++ oscode burst example, containing semicolon-delimited lines with complex-valued (t, x) step pairs for RK and WKB steps
- custom parse_pair converter function for decoding the pair fields into complex numbers

## Outputs

- numpy complex-dtype array of parsed step pairs (times and complex solution values)
- separated RK-step and WKB-step series on the time grid, ready for plotting against the analytic burst solution

## How to apply

After running the burst example ('g++ -I../include/ ... ./burst'), which writes RK and WKB steps to 'output.txt', load the file with np.genfromtxt specifying dtype=complex and per-column converters (parse_pair) with delimiter=';'. The converters decode each semicolon-separated field into its complex representation, so the two columns become the time and complex solution value of every step taken by the solver (Runge-Kutta and WKB steps alike). Then split or mask the parsed rows by step type if the RK/WKB distinction matters, and feed the resulting ts/xs arrays into matplotlib alongside the analytic reference 100*(1+t^2)^0.5/100*cos(100*arctan(t)) for the burst-equation comparison.

## Related tools

- **numpy** (parses 'output.txt' via np.genfromtxt with dtype=complex and per-column parse_pair converters, delimiter=';') — https://pypi.org/project/numpy/
- **oscode** (C++ burst example whose compiled run ('./burst') writes the complex step pairs to 'output.txt' that this skill parses) — https://github.com/fruzsinaagocs/oscode
- **matplotlib** (consumes the parsed RK and WKB step arrays to reproduce the burst comparison figure) — https://pypi.org/project/matplotlib/

## Examples

```
data = np.genfromtxt('output.txt', dtype=complex, converters={1: parse_pair, 2: parse_pair}, delimiter=';')
```

## Evaluation signals

- np.genfromtxt returns an array with dtype complex (no parsing errors or NaNs from undecoded fields)
- Number of parsed rows matches the number of non-empty lines in 'output.txt'
- Real parts of column 1 form a monotonically (integration direction aware) ordered time grid spanning the integration range
- Plotted numerical steps visually and numerically track the analytic envelope cos(100*arctan(t)) (scaled by 100*(1+t^2)^0.5/100) across the rapid oscillations
- RK and WKB step subsets both remain consistent with the analytic curve, confirming no steps were dropped or misdecoded during conversion

## Limitations

- The parsing recipe is specific to the semicolon-delimited complex-pair format written by the burst C++ example; other output formats (e.g. pyoscode's in-memory solution or 'example.txt') need different handling
- Requires a user-defined parse_pair converter — stock genfromtxt cannot decode these fields, so changing the file layout silently breaks parsing
- Legacy solver versions had output bugs (dense-output fixes in 1.0.5/1.0.6 and changelog entries for correct file handling), so very old releases may produce 'output.txt' content that parses but misrepresents the solution

## Evidence

- [results] parse step pairs with converters: "data = np.genfromtxt(f1,dtype=complex,converters={1:parse_pair,2:parse_pair},delimiter=";"
- [abstract] why both step types appear in the output file: "dynamically switching between using a Runge---Kutta estimate and the Wentzel---Kramers---Brillouin approximation in areas of slowly varying frequency"
- [results] comparison of parsed steps against analytic burst solution: [evidence span withheld — anchor status: not_found]
- [readme] origin of the output file: "To plot results from `burst.cpp`: `examples/plot_burst.py`"
