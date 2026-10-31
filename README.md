# SIRD Endemic Projection Dashboard

An interactive [Streamlit](https://streamlit.io) dashboard for projecting the course of COVID-19 in Malaysia with an SIRD (Susceptible-Infected-Recovered-Deceased) model. Choose how strict non-pharmaceutical interventions (NPIs) are over future date ranges and see the projected infections, recoveries and deaths with uncertainty bands, alongside the estimated model parameters.

## Features

- **Scenario-based projection**: define up to eight date windows, each with an NPI strictness of *Strict*, *Mild* or *Loose*.
- **Stochastic forecasts**: 500 simulated paths per scenario using a stochastic Runge-Kutta (SRK4) solver, shown as the median with a 25th-75th percentile band.
- **Fit to data**: simulated past trajectories plotted against observed S, I, R and D.
- **Reproduction number**: R0 = beta / (r + d) from the estimated parameters, plus a weekly growth-based estimate that extends into the projection.
- **Parameter estimation results**:
  - histogram of the estimated infection rate (beta), colour-coded by NPI level,
  - recovery and fatality rate trends with logarithmic fits,
  - a slider to choose how many days of parameter history to display.

## How it works

1. The past 500 simulations (S, I, R, D) are produced in MATLAB with the SRK4 solver and loaded from CSV. Percentiles are computed for each day.
2. The projection starts from the median final state of those simulations and runs the SRK4 solver in Python with `sigma = 0.05`.
3. Projection parameters:
   - **Recovery (`r`) and fatality (`d`) rates** are extrapolated with a logarithmic fit, `a*ln(t) + b`, to the MCMC estimates.
   - **`gamma`** is the mean of the estimated values.
   - **Infection rate (`beta`)** is taken from the peaks of the histogram of historical beta values. The peaks are mapped to the NPI choices: Strict uses the lowest-beta peak, Mild the next, Loose the highest (tier lookup is `Strict=2, Mild=1, Loose=0` on the reversed peak list).

Model equations (S, I, R, D, population `P`):

```
dS = -S/P * beta * I + gamma * R
dI =  S/P * beta * I - (r + d) * I
dR =  r * I - gamma * R
dD =  d * I
```

with multiplicative noise `sigma * I * S / P` on S and I.

## Repository structure

```
.
├── main.py                        # Streamlit app
├── requirements.txt               # pinned dependencies
├── sird_data_Malaysia.csv         # observed data
├── mcmc_result_Malaysia.csv       # estimated parameters per day
└── 500simdata/
    ├── Malaysia_SIRD_S.csv        # 500 simulated paths per compartment
    ├── Malaysia_SIRD_I.csv
    ├── Malaysia_SIRD_R.csv
    └── Malaysia_SIRD_D.csv
```

## Input data

| File | Contents |
|---|---|
| `sird_data_Malaysia.csv` | `date`, `cases_active`, `cases_recovered`, `deaths_total`, `susceptible` |
| `mcmc_result_Malaysia.csv` | `t`, `beta`, `r`, `d`, `gamma`, `sigma` (one row per day; rows with missing values are dropped) |
| `500simdata/Malaysia_SIRD_*.csv` | Output of the MATLAB SRK4 simulation, 500 paths per compartment (read and transposed to days x paths) |

Population is set to 31,530,000 in `main.py`. Data source: *[add source here]*.

## Getting started

```bash
git clone <repo-url>
cd <repo-folder>
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
streamlit run main.py
```

The pinned versions (for example `numpy==1.24.2`, `pandas==1.5.3`) suit Python 3.8-3.11.

## Usage

Use the sidebar to set each NPI window: a start date, an end date and a strictness level. The first window defaults to the 30 days after the data ends. Leave a window's start and end at the same date to disable it. The chart updates automatically, and the slider on the right controls how much of the estimation history is shown.

## Notes

- Future days not covered by any NPI window have `beta = 0`, so no new infections occur on those days. Cover the full projection period with windows.
- Data paths in `main.py` use Windows separators (`500simdata\...`). On macOS or Linux, change them to `500simdata/...`.

## License

MIT (see `LICENSE`).

## Citation

If you use this project, please cite the associated paper: *[add reference / DOI here]*.
