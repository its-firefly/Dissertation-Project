# Passive Cold-Storage Insulation Modelling (Transient Heat Conduction + Parameter Fitting)

## Overview

This project investigates the thermal performance of a **multi-layer passive cooling container** — used for off-grid cold-chain storage (e.g. food/vaccine transport) — by combining experimental temperature-logging data with a **1-D transient heat conduction model**.

The container wall is modelled as a stack of layers: external insulation, an outer structural shell, a gas-filled insulation gap (originally CO₂, with comparisons against air and noble gases), an inner structural shell, and a phase-change material (PCM) core that provides the cooling effect.

```
       [Ambient Air, h_amb = 5 W/m²K]
                      ||
       +------------------------------+
       |   External Insulation (50mm) |
       +------------------------------+
       |   Outer Shell (3mm)          |
       +------------------------------+
       |   Insulation Gas Gap (30mm)  |  <-- CO2 / Air / Argon / Krypton / Xenon
       +------------------------------+
       |   Inner Shell (3mm)          |
       +------------------------------+
       |   PCM Core (30mm)            |
       +------------------------------+
                      ||
            [Refrigerated Chamber]
```

## Methodology

### 1. Transient Conduction Model
Each layer is modelled with the 1-D transient heat conduction equation:

$$\frac{\partial^2 T}{\partial x^2} = \frac{1}{\alpha}\frac{\partial T}{\partial t}, \qquad \alpha = \frac{k}{\rho C_p}$$

Solved analytically via separation of variables, giving a 100-term series solution in terms of dimensionless space, temperature, and Fourier number:

$$\theta^* = \sum_{n=1}^{100} C_n \exp(-\zeta_n^2 Fo)\cos(\zeta_n x^*)$$

where the eigenvalues $\zeta_n$ satisfy the transcendental equation $\zeta_n \tan\zeta_n = Bi$ (solved numerically with `scipy.optimize.fsolve`), and $C_n$ are the corresponding series coefficients.

### 2. Layer Coupling
Layers are solved sequentially, with the surface temperature of one layer feeding in as the boundary condition for the next — propagating the temperature signal from the ambient environment, through the insulation and gas gap, into the PCM core.

### 3. Parameter Fitting (`optim.py`)
Several physical properties of the PCM and gap convective coefficients were not known a priori. These are recovered by **minimising the mean-squared error** between the model's predicted PCM temperature history and experimental thermocouple logs, using `scipy.optimize.minimize` (L-BFGS-B with bounds).

### 4. Comparative & Sensitivity Analysis (`correlationAnalysis.ipynb`)
Once the model is fitted, the insulation gas is swapped for alternatives (air, argon, krypton, xenon) to compare cold-storage duration. A large-sample Pearson correlation analysis is then run to identify which physical parameters (gas conductivity, density, specific heat, layer thickness, etc.) most strongly affect PCM temperature evolution.

## Repository Structure

```
Dissertation-Project/
├── main.ipynb                  # Main driver: loads data, runs fitting, simulates scenarios, plots results
├── optim.py                    # Objective function + L-BFGS-B parameter-fitting routine
├── utils.py                    # Core transient conduction solver (eigenvalue roots, series expansion)
├── correlationAnalysis.ipynb   # Monte Carlo sampling + Pearson correlation / sensitivity analysis
├── experimentalData.txt        # Logged experimental temperature data
├── extras/                     # Auxiliary optimisation outputs and reference images
├── SpVarData@timeT/            # Spatial temperature distribution outputs at various times
├── .idea/tvsTData/             # Temperature-vs-time datasets for each gas (CO2, air, argon, krypton, xenon)
├── correlation_Data_Plots/      # Correlation heatmaps and tables
└── graphs/                      # Analytical vs. experimental comparison plots
```

## Requirements

```bash
pip install numpy scipy matplotlib pandas
```

## Running

1. Run `main.ipynb` to fit the unknown PCM/gas parameters against `experimentalData.txt` and simulate the temperature evolution for each insulation gas scenario.
2. Run `correlationAnalysis.ipynb` to perform the sensitivity/correlation analysis and generate the correlation plots.

## Key Findings

- A well-applied external insulation layer substantially extended the time the PCM core remained below freezing.
- Replacing the air gap with CO₂ (or other gases) produced **no meaningful improvement** in cold-storage duration in either the experimental or analytical results — the gas layer reaches thermal steady-state too quickly for its conductivity to matter much.
- Sensitivity analysis showed that PCM **specific heat capacity and density** had a much larger effect on cooling performance than the choice of insulation gas.

## Notes

- The fitted PCM properties are empirical regression results (back-calculated to match experimental data), not independently measured material properties — they should be treated as model-fitting parameters rather than as published material data.
