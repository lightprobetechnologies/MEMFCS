# MEM-FCS Analyzer by LIGHTPROBE Technologies 


**Maximum Entropy Method (MEM) analysis of Fluorescence Correlation Spectroscopy (FCS) data, with a graphical interface.**

MEM-FCS Analyzer fits FCS autocorrelation curves with a maximum entropy approach that recovers *distributions* of diffusion times and reaction (triplet/blinking) times, rather than forcing the data into a fixed number of discrete components. A conventional discrete fit, including photon anti-bunching, is run alongside it, so both results can be compared directly in the same window.

---

## Features

- **Graphical interface** with data import, parameter entry, live plots, a results log and export.
- **Data import** of `.txt`, `.dat`, `.csv` and `.xlsx` / `.xls` files, single or multiple at once.
- **Discrete fit** with 4 diffusion components, 3 reaction components, an anti-bunching term and the confocal aspect-ratio (structure) parameter.
- **Automatic anti-bunching quality check** that decides whether anti-bunching is carried into the MEM fit.
- **MEM fit** returning a diffusion-time distribution and a reaction-time distribution simultaneously, with automatic selection of the regularization strength λ.
- **Flexible noise estimation** from the residuals of the discrete fit, or from an ensemble of replicate measurements.
- **Peak detection** on both distributions, with exact values reported in the log.
- **Optimization diagnostics** on demand: convergence plots and a table of fitting parameters.
- **Export** of all results to `.xlsx` or `.csv`.

---

## Installation

### Setup

Download the latest release from the release section; it currently works on Windows.

### Run

Run the executable downloaded from the release.

---

## Input Data Format

Each file must contain at least two columns:

| Column | Content | Unit |
|--------|---------|------|
| 1 | Lag time τ | **milliseconds** |
| 2 | Correlation function G(τ) | dimensionless |

- `.txt` and `.dat` files should be whitespace-delimited; `.csv` files comma-delimited.
- `.xlsx` / `.xls` files are read from the first sheet; rows with missing or non-numeric values are dropped.
- The curve should extend to at least 10 ms, because the long-time plateau is used to estimate the initial amplitude of the discrete fit.

Some example data is provided in the Sample Data folder in the repository, feel free to use them.
---

## Workflow

The interface is organized into three panels, followed by a log and three plots.

### 1 · Import Data

Load one or more files with **Load Data…**. The discrete fit is run immediately on the loaded range, so the data limits set here directly determine the discrete fit quality.

| Setting | Description |
|---------|-------------|
| Minimum Time (s) | Lower limit of the lag-time range kept for analysis (default `1e-9`) |
| Maximum Time (s) | Upper limit of the lag-time range kept for analysis (default `1`) |
| Ensemble noise determination | Use the noise estimated from replicate measurements (see [Noise Estimation](#noise-estimation)) |

> **Tip:** Adjust the minimum and maximum time until the discrete fit follows the experimental curve accurately. The MEM fit takes its fixed parameters from this discrete fit.

### 2 · MEM Components Configuration

Choose the search ranges and resolution for both distributions.

| Setting | Description | Default |
|---------|-------------|---------|
| Minimum / Maximum Diffusion Time (s) | Range of the diffusion-time grid | `1e-5` / `0.1` |
| Diffusion Components | Number of grid points for the diffusion distribution | `100` |
| Minimum / Maximum Reaction Time (s) | Range of the reaction-time grid | `10e-9` / `2e-5` |
| Reaction Components | Number of grid points for the reaction distribution | `100` |

Grids are logarithmically spaced.

### 3 · Analysis

| Button | Action |
|--------|--------|
| **Run MEM Fit** | Runs the maximum entropy optimization wait for the optimisation, till the plots and the log is updated|
| **Optimization History & Fitting Parameters** | Opens convergence plots and a parameter table (enabled after a fit) |
| **Export Results (.xlsx / .csv)** | Saves the spectra, fit curves and residuals |

### Log

All results are written to the log with exact values: parametric fit parameters, the optimal λ, the reduced χ², and the positions and amplitudes of every detected peak. Use **Clear Log** to empty it.

---

## Methods

### Discrete fit

The experimental curve is first fitted with a parametric model composed of four 3D-diffusion components, three reaction (triplet) components and an anti-bunching factor:

$$
G(\tau) = \left[1 - C_{ab}\ e^{-\tau/\tau_{ab}}\right]\
\frac{1 - T + \sum_{j=1}^{3} T_j e^{-\tau/\tau_{T_j}}}{1 - T}\
\sum_{i=1}^{4}\frac{A_i}{\left(1+\tau/\tau_{D_i}\right)\sqrt{1+\tau/\left(S^2\tau_{D_i}\right)}}
$$

with $T = T_1 + T_2 + T_3$ and $S$ the aspect ratio of the confocal volume. The fit returns the amplitudes and times of all components together with $S$, $C_{ab}$ and $\tau_{ab}$.

### Anti-bunching check

Anti-bunching occurs at nanosecond lag times, where photon-counting noise is often large. The program therefore tests the discrete-fit residuals below 5 ns for:

- a statistically significant offset of the fit from the data (Z-score),
- an imbalance between positive and negative residuals,
- too few crossings of the fit through the data scatter,
- a residual exceeding 40 % of the local experimental value.

If the data pass, anti-bunching is included in the MEM fit. If any test fails, the user is warned in the log, the discrete fit is repeated on τ ≥ 10 ns without anti-bunching, and **the anti-bunching regime is excluded from the MEM fit**. Adjusting the minimum and maximum time bounds can change this outcome.

### Maximum entropy fit

The anti-bunching parameters ($C_{ab}$, $\tau_{ab}$) and the aspect ratio $S$ from the discrete fit are treated as **constants** in the MEM fit. The solver then determines two non-negative distributions: a diffusion spectrum $a_D$ over the diffusion-time grid and a reaction spectrum $a_T$ over the reaction-time grid, by minimizing

$$
Q = \chi^2 - \lambda\S_{\text{entropy}}
$$

using   [L-BFGS-B](https://en.wikipedia.org/wiki/Limited-memory_BFGS) with an exact analytical gradient. The regularization strength λ is scanned over 50 logarithmically spaced values between $10^{-2}$ and $10^{4}$, and the value giving a reduced χ² closest to 1 (subject to a regularization-dominance condition which restricts over dominance of χ² ) is selected automatically.

### Noise estimation

| Mode | Source of σ |
|------|-------------|
| **Single data set** (default) | Deviations of the experimental curve from the discrete fit, with a small noise floor |
| **Ensemble** | Standard deviation across a set of replicate measurements |

To use ensemble noise:

1. Load all replicate files together with **Load Data…**. Their curves are interpolated onto a common lag-time grid and the point-wise standard deviation is stored.
2. Tick **Ensemble noise determination**.
3. Load the single data set to be analysed. The stored ensemble noise is used as σ for the MEM fit.

---

## Outputs

### Plots

1. **Correlation curve**: experimental data with the discrete and MEM fits.
2. **Residuals**: MEM fit minus experiment.
3. **Combined distributions**: diffusion distribution on the left y-axis and reaction distribution on the right y-axis, sharing one time axis. Detected peaks are marked, and the discrete-fit components are overlaid as stems for direct comparison.

### Optimization window

Opened with **Optimization History & Fitting Parameters** (not shown automatically):

- **Convergence History**: objective function, χ² and entropy versus iteration.
- **Optimization Parameters**: optimal λ, reduced χ², entropy, grid settings, parametric-fit inputs and optimizer settings.

### Export

| Format | Files written |
|--------|---------------|
| `.xlsx` | One workbook with sheets `Diffusion_Spectrum`, `Triplet_Spectrum` and `FCS_Fit_Data` |
| `.csv` | `<name>.csv` (diffusion spectrum), `<name>_triplet.csv` and `<name>_fit.csv` |

---

## Notes and Limitations

- The convergence plots show the optimizer history of the last λ in the scan, not the history at the selected optimal λ.
- The MEM calculation scans 50 values of λ, so run time grows with the number of grid points and data points. Reduce the grid size for quick exploration.
- Result quality depends on how well the discrete fit describes the data. Review the discrete fit and the anti-bunching message in the log before interpreting the MEM distributions.

---


## License

Free to use for academic purposes only.

## Citation

If you use this software in published work, please cite it as:

```
Parijat Sengupta, K. Garai, J. Balaji, N. Periasamy, S. Maiti, Measuring Size Distribution in Highly Heterogeneous Systems with Fluorescence Correlation Spectroscopy,
Biophysical Journal, Volume 84, Issue 3, 2003, Pages 1977-1984, ISSN 0006-3495, https://doi.org/10.1016/S0006-3495(03)75006-1.
```

## Contact

Kaustav Pal · kaustavpal1612@gmail.com · Birla Institute of Technology and Science Hyderabad Campus
