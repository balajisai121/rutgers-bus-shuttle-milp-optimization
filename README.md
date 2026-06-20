# Rutgers Bus Shuttle Network Optimization using MILP

A reproducible **Mixed-Integer Linear Programming (MILP)** project for selecting Rutgers–New Brunswick bus routes and assigning buses while maintaining required stop coverage.

![Weekday optimal allocation](assets/weekday_bus_allocation.png)

## Project overview

The campus shuttle system must serve many stops with a limited fleet. This model balances two competing goals:

1. reduce the passenger travel-time contribution of the selected routes; and
2. reduce the number of buses operated.

The optimization was first developed in Microsoft Excel Solver and is reproduced here in Python using **SciPy's HiGHS MILP solver**. The repository contains the cleaned route data, mathematical formulation, executable code, validation tests, generated results, and the original Excel workbook.

## Optimal results

| Period | Selected routes | Total buses | Passenger-time component | Bus-cost component | Objective |
|---|---|---:|---:|---:|---:|
| Weekday | EE, F, H, LX | 4 | 170 | 2,000 | **2,170** |
| Weekend | Weekend 1 | 1 | 64 | 500 | **564** |

The Python implementation reproduces the final Excel Solver results exactly.

## MILP formulation

### Decision variables

- `y_r = 1` when route `r` operates; otherwise `0`.
- `x_r` is the integer number of buses assigned to route `r`.

### Objective

Minimize:

```text
sum_r (P_r × T_r × y_r) + c × sum_r x_r
```

where `P_r` is passenger demand, `T_r` is route travel time, and `c = 500` is the bus-cost weight.

### Main constraints

```text
sum_r a_rs × y_r >= 1       for every required stop s
x_r >= y_r                  for every route r
x_r <= M × y_r              for every route r

y_r binary
x_r nonnegative integer
```

`a_rs` equals 1 when route `r` covers stop `s`, and `M = 10` is the maximum number of buses that may be assigned to one route.

A complete explanation is available in [`docs/model_formulation.md`](docs/model_formulation.md).

## Repository structure

```text
├── assets/                  Generated charts
├── config/                  Model parameters and required-stop definitions
├── data/                    Clean weekday and weekend route data
├── docs/                    Formulation, validation, and project explanation
├── project_files/           Original Excel Solver workbook
├── results/                 Optimal solutions and sensitivity outputs
├── src/                     Reusable optimization code
├── tests/                   Reproduction and feasibility tests
├── run.py                   Command-line entry point
└── requirements.txt         Python dependencies
```

## Run the project

### 1. Create a Python environment

```bash
python -m venv .venv
```

Activate it on Windows:

```bash
.venv\Scripts\activate
```

Activate it on macOS or Linux:

```bash
source .venv/bin/activate
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Solve both models

```bash
python run.py --period all --plots --sensitivity
```

Other examples:

```bash
python run.py --period weekday
python run.py --period weekend
python run.py --period weekday --bus-cost 750
```

### 4. Run validation tests

```bash
python -m unittest discover -s tests -v
```

## Important model limitation

The current Excel model charges passenger time when a route is activated, but **does not make waiting time decrease as more buses are assigned**. Therefore, the optimizer assigns exactly one bus to every selected route. This is mathematically consistent with the current formulation, but it does not yet represent the real frequency–waiting-time relationship.

A stronger future version should model route frequency or headway and use a piecewise-linear approximation of waiting time. See [`docs/model_validation_and_limitations.md`](docs/model_validation_and_limitations.md).

## Author

**Balaji Sai Tolapu**  
M.S. Industrial and Systems Engineering  
Rutgers University–New Brunswick
