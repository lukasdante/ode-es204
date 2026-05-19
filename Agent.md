# ES 204 Memristor ODE Project Notes

## Project Goal

This repository implements the mini-project proposed in
`Lagramada-Villareal-Proposal.pdf`: compare Explicit Euler, classical RK4,
and Implicit Euler on a bounded nonlinear HP TiO2 memristor state equation.

The implementation is intentionally course-level and transparent. It uses:

- NumPy for arrays and numerical computation
- Matplotlib for plots
- Pandas for summary tables
- Jupyter Notebook as the executable script format

No SciPy ODE solvers are used.

## Main Notebook

The main implementation is:

`src/memristor_methods.ipynb`

The notebook uses the Jupyter kernel:

- kernelspec name: `es204`
- display name: `ES204 Project`

It contains the full workflow:

1. Define physical and numerical parameters.
2. Implement the memristor model.
3. Implement Explicit Euler, RK4, and Implicit Euler.
4. Simulate several step sizes.
5. Store raw and clipped trajectories.
6. Build overshoot and final-state tables.
7. Build runtime tables using repeated `time.perf_counter()` calls.
8. Generate state trajectory, V-I hysteresis, boundary, and runtime figures.

## Model

The implemented model follows the proposal:

```text
M(w) = Ron (w/D) + Roff (1 - w/D)
v(t) = V0 sin(omega t)
i(t, w) = v(t) / M(w)
dw/dt = mu_v Ron v(t) / (D^2 M(w))
```

The physical state constraint is `0 <= w <= D`.

The notebook default uses `frequency_hz = 1e8` as a compact stress case. This
keeps the simulation short while making boundary overshoot visible in the raw
trajectories.

## Bound Handling

Each solver produces a raw trajectory. Raw trajectories are not clipped during
time stepping, so overshoot is visible and measurable.

For physically meaningful current-voltage plots, the notebook also stores a
clipped state trajectory:

```text
w_clipped = min(max(w_raw, 0), D)
```

Summary tables report bound violations from the raw states.

## Numerical Methods

Explicit Euler uses:

```text
w_{n+1} = w_n + h f(t_n, w_n)
```

RK4 uses the classical four-stage formula:

```text
k1 = f(t_n, w_n)
k2 = f(t_n + h/2, w_n + h k1/2)
k3 = f(t_n + h/2, w_n + h k2/2)
k4 = f(t_n + h, w_n + h k3)
w_{n+1} = w_n + h (k1 + 2k2 + 2k3 + k4)/6
```

Implicit Euler solves the nonlinear scalar equation:

```text
F(w_{n+1}) = w_{n+1} - w_n - h f(t_{n+1}, w_{n+1}) = 0
```

The notebook uses an Explicit Euler predictor followed by Newton iteration.
The derivative is analytic for the scalar memristor model. If Newton reaches
the iteration limit near the raw memristance singularity, the notebook falls
back to the equivalent scalar quadratic form of the same Implicit Euler update
and selects the root closest to the Newton predictor.

## Outputs

Executing the notebook writes:

- `latex/figures/state_trajectories.png`
- `latex/figures/bound_violations.png`
- `latex/figures/hysteresis_loops.png`
- `latex/figures/runtime_per_step.png`
- `latex/tables/overshoot_summary.csv`
- `latex/tables/final_state_summary.csv`
- `latex/tables/runtime_summary.csv`

The LaTeX report source is:

`latex/report/report.tex`

The longer presentation-oriented report source is:

`latex/exhaustive_report/exhaustive_report.tex`

## Execution

Run the notebook end-to-end with:

```bash
.venv/bin/jupyter nbconvert --to notebook --execute --inplace src/memristor_methods.ipynb --ExecutePreprocessor.kernel_name=es204 --ExecutePreprocessor.timeout=3600
```
