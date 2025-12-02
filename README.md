# Multivariate Time Series to NetLogo Model Pipeline

This Jupyter notebook provides an end-to-end pipeline for transforming normalized multivariate longitudinal data into an executable NetLogo agent-based model. The pipeline uses advanced econometric techniques (VAR and VECM) to decompose complex interdependent relationships between variables and model their dynamics over time. Intent of this is to make ingesting real data easier for Complexity and Systems science classes, as well as enable modeling of Complex Adaptive Systems (CAS) more easily.

## Overview

The notebook analyzes how multiple variables influence each other over time, identifies both short-term causal relationships and long-term equilibrium relationships, and encodes these relationships into a NetLogo simulation model that can be used for forecasting and scenario analysis.

## Key Statistical Methods

### 1. Stationarity Testing (ADF & KPSS Tests)

**What it is:** Before analyzing time series data, we need to determine if variables are "stationary" - meaning their statistical properties (mean, variance) don't change over time.

**In this script:**
- **ADF Test (Augmented Dickey-Fuller)**: Tests if a variable has a unit root (non-stationary). A low p-value (< 0.05) indicates stationarity.
- **KPSS Test**: Tests the opposite - if a variable is stationary. A high p-value (> 0.05) indicates stationarity.
- Variables that are non-stationary are flagged for differencing (looking at changes rather than levels).

**Why it matters:** Non-stationary data can produce spurious correlations. VAR models require stationary data, while VECM can handle non-stationary data with cointegration.

### 2. Variable Selection (Correlation Centrality)

**What it is:** Reduces the dataset to the most influential variables to avoid over-parameterization.

**In this script:**
- Calculates each variable's "centrality" in the correlation network (sum of absolute correlations with all other variables)
- Combines correlation centrality with variance to identify the most important variables
- Selects the top N variables for analysis (default: 8)

**Why it matters:** VAR/VECM models grow exponentially in complexity with more variables. Selecting 6-8 key variables provides the best balance between completeness and statistical power.

### 3. Granger Causality Tests

**What it is:** Tests if past values of variable X help predict future values of variable Y, beyond what Y's own past values tell us.

**In this script:**
- Tests all pairwise combinations of selected variables
- For each pair, tests multiple lag lengths (1 to MAX_LAG periods)
- Identifies significant causal relationships (p < 0.05)
- Records the optimal lag length for each relationship

**Important:** "Granger causality" doesn't prove true causation - it means X provides predictive information about Y.

**Example:** If "temperature" Granger-causes "ice cream sales" with lag 1, it means yesterday's temperature helps predict today's ice cream sales.

### 4. VAR (Vector Autoregression)

**What it is:** A system of equations where each variable is predicted by its own past values AND the past values of all other variables.

**In this script:**
- Estimates a VAR model with optimal lag order (selected by AIC criterion)
- Each variable's equation includes coefficients for all lagged variables
- Captures short-run dynamics and mutual interdependencies
- Extracts coefficients that show how each variable influences others

**Example VAR equation for variable Y:**
```
Y(t) = α + β₁·Y(t-1) + β₂·X(t-1) + β₃·Z(t-1) + ε(t)
```

**Why it matters:** VAR reveals the network of short-term influences between variables, showing both direct effects and feedback loops.

### 5. Johansen Cointegration Test

**What it is:** Tests if non-stationary variables have stable long-run equilibrium relationships, even though they individually wander over time.

**In this script:**
- Tests for cointegration among the selected variables
- Determines the "cointegration rank" (number of long-run equilibrium relationships)
- Provides evidence for whether VECM is appropriate

**Example:** Stock prices of two competing companies might both trend upward (non-stationary), but their ratio stays bounded (cointegrated) because they compete in the same market.

**Why it matters:** Cointegration reveals long-term constraints that prevent variables from drifting arbitrarily far apart.

### 6. VECM (Vector Error Correction Model)

**What it is:** An extension of VAR for non-stationary cointegrated variables. It combines short-run VAR dynamics with error correction terms that pull the system back toward long-run equilibrium.

**In this script:**
- Estimates VECM if cointegration is detected
- Extracts two key components:
  - **Alpha (error correction coefficients)**: Speed of adjustment toward equilibrium
  - **Beta (cointegrating vectors)**: The long-run equilibrium relationships
- Combines with VAR-like short-run dynamics (gamma coefficients)

**VECM equation structure:**
```
ΔY(t) = α·(β'·X(t-1)) + Γ₁·ΔY(t-1) + ... + ε(t)
       └─────────┘     └──────────────┘
       Error correction  Short-run VAR
       (pulls to equilibrium) (immediate dynamics)
```

**Example:** If government spending and GDP are cointegrated at a 1:4 ratio long-term, but this period spending jumped while GDP lagged, the error correction pulls GDP up and spending down to restore the equilibrium relationship.

**Why it matters:** VECM captures both how variables respond to short-term shocks AND how they revert to long-term equilibrium relationships.

## Pipeline Workflow

### Input Requirements
- **Data format:** Excel (.xlsx) or CSV (.csv) file
- **Data preparation:**
  - Normalized time series (z-scores for normally distributed data, log transformations for exponential growth)
  - One time column (e.g., "Year", "Date", "FY")
  - Multiple numeric variable columns

### Processing Steps

1. **Data Loading & Exploration**
   - Loads data and identifies numeric variables
   - Displays summary statistics and data structure

2. **Stationarity Testing**
   - Runs ADF and KPSS tests on all variables
   - Outputs: `stationarity_tests.xlsx`

3. **Variable Selection**
   - Calculates correlation centrality and variance
   - Selects top N most important variables
   - Outputs: `variable_importance.xlsx`, `variable_importance.png`

4. **Granger Causality Analysis**
   - Tests all pairwise relationships with multiple lags
   - Identifies significant causal connections
   - Outputs: `granger_causality.xlsx`

5. **VAR Estimation**
   - Selects optimal lag order using AIC
   - Estimates VAR model
   - Extracts coefficients for each variable
   - Outputs: `var_summary.txt`, `var_coefficients.xlsx`

6. **Cointegration Testing**
   - Johansen test determines cointegration rank
   - Outputs: `cointegration_test.txt`

7. **VECM Estimation** (if cointegration exists)
   - Estimates error correction model
   - Extracts alpha (adjustment speeds) and beta (equilibrium relationships)
   - Outputs: `vecm_summary.txt`, `vecm_alpha.xlsx`, `vecm_beta.xlsx`

8. **NetLogo Model Generation**
   - Converts statistical model into NetLogo code
   - Implements:
     - Error correction mechanisms (from VECM)
     - VAR short-run dynamics (from Granger causality)
     - Polynomial interactions (2nd and 3rd order)
     - Numerical stability controls
   - Outputs: `{MODEL_NAME}.nlogo`

### Output Files

**Statistical Analysis:**
- `stationarity_tests.xlsx` - ADF and KPSS test results
- `variable_importance.xlsx` - Variable ranking by importance
- `variable_importance.png` - Visualization of top variables
- `granger_causality.xlsx` - Significant causal relationships
- `var_summary.txt` - Complete VAR model output
- `var_coefficients.xlsx` - VAR coefficient matrix
- `cointegration_test.txt` - Johansen test results
- `vecm_summary.txt` - Complete VECM model output
- `vecm_alpha.xlsx` - Error correction coefficients
- `vecm_beta.xlsx` - Cointegrating vectors

**NetLogo Model:**
- `{MODEL_NAME}.nlogo` - Executable NetLogo model (requires NetLogo 6.4)

## NetLogo Model Features

The generated NetLogo model includes:

1. **Global Variables:** All selected variables with lag histories (up to MAX_LAG periods)

2. **Error Correction Mechanism:**
   - Calculates equilibrium deviations using beta coefficients
   - Applies corrections using alpha coefficients
   - Adjustable strength via `error-correction-strength` slider

3. **VAR Dynamics:**
   - Implements all significant Granger causal relationships
   - Applies lagged influences between variables
   - Adjustable strength via `var-strength` slider

4. **Polynomial Interactions:**
   - 2nd order (X × Y) interactions
   - 3rd order (X × Y × Z) interactions (optional)
   - Adjustable via `interaction-strength` slider
   - Can be toggled with `use-interactions?` switch

5. **Noise & Stability:**
   - Gaussian noise addition (adjustable via `noise-level`)
   - Variable clipping to prevent numerical overflow

6. **Visualization:**
   - Real-time monitors for all variables
   - Multi-line plot showing all variables over time
   - Year counter

### Using the NetLogo Model

1. Open the `.nlogo` file in NetLogo 6.4
2. Click **Setup** to initialize
3. Adjust sliders to control model behavior:
   - `error-correction-strength`: How strongly variables return to equilibrium (0-2)
   - `var-strength`: Strength of short-run VAR dynamics (0-2)
   - `noise-level`: Random fluctuation magnitude (0-5)
   - `interaction-strength`: Polynomial interaction effects (0-1)
   - `max-year`: Simulation stopping point (0 = run indefinitely)
4. Click **Go** to run the simulation
5. Observe variable trajectories in the plot and monitors

## Configuration Parameters

Edit these in the notebook's Section 0:

```python
DATA_FILE = 'path/to/your/data.csv'          # Input data file
TIME_COLUMN = 'time'                          # Name of time column
TOP_N_VARIABLES = 8                           # Number of variables to select
MAX_LAG = 5                                   # Maximum lag order to test
GRANGER_ALPHA = 0.05                          # Significance level for Granger tests
OUTPUT_DIR = 'NetLogo_Model_Output'           # Output directory
MODEL_NAME = 'Time_Series_VECM_Model'         # Output model name
INCLUDE_SECOND_ORDER = True                   # Include X*Y interactions
INCLUDE_THIRD_ORDER = True                    # Include X*Y*Z interactions
```

## Requirements

```python
pandas
numpy
matplotlib
seaborn
statsmodels
```

Install with: `pip install pandas numpy matplotlib seaborn statsmodels`

## Interpretation Guide

### Understanding the Outputs

**Granger Causality Results:**
- Low p-values (< 0.05) indicate strong predictive relationships
- `best_lag` shows the optimal time delay for the relationship
- Example: If X → Y with lag 2 and p=0.001, then X from 2 periods ago strongly predicts current Y

**VAR Coefficients:**
- Positive coefficients: variables move together
- Negative coefficients: variables move in opposite directions
- Large coefficients (> 0.5): strong influence
- Coefficients are multiplied by lagged values to predict current values

**VECM Alpha (Error Correction):**
- Negative values: variable decreases when above equilibrium
- Positive values: variable increases when below equilibrium
- Magnitude shows adjustment speed (larger = faster correction)

**VECM Beta (Cointegrating Vectors):**
- Define the long-run equilibrium relationships
- Example: β₁=1.0, β₂=-0.25 means "Variable 1 = 0.25 × Variable 2" in equilibrium

### Common Patterns

1. **No Cointegration (rank = 0):** Variables don't have stable long-term relationships - only VAR dynamics apply
2. **Low Cointegration Rank (1-2):** One or two dominant long-term equilibria constrain the system
3. **High Cointegration Rank:** Many equilibrium constraints - system is tightly coupled
4. **Many Granger Relationships:** Dense network of interdependencies
5. **Few Granger Relationships:** Variables are relatively independent

## Theoretical Background

This pipeline implements the **Engle-Granger two-step approach** to time series analysis:

1. **Step 1:** Test for cointegration (Johansen test)
2. **Step 2:** Estimate VECM if cointegrated, or VAR if not

The resulting model decomposes variable dynamics into:
- **Trend components:** Long-run equilibrium relationships (VECM beta)
- **Adjustment components:** Error correction speeds (VECM alpha)
- **Short-run components:** Immediate responses to lagged changes (VAR gamma)
- **Noise components:** Random shocks (residuals)

## Applications

This pipeline can be used for:

- **Economic forecasting:** GDP, inflation, interest rates
- **Business analytics:** Sales, inventory, prices across product lines
- **Environmental modeling:** Climate variables, pollution indicators
- **Health metrics:** Disease prevalence, treatment outcomes over time
- **Engineering systems:** Sensor networks, multivariate control systems
- **Social dynamics:** Population demographics, policy impacts

Any domain with multiple interrelated variables measured over time can benefit from this analysis.

## Limitations & Assumptions

1. **Data must be normalized:** The model assumes z-scored or log-transformed data
2. **Sufficient observations:** Need at least 50-100 time points for reliable estimates
3. **Linear relationships:** VAR/VECM assume linear dynamics (polynomial interactions help but don't fully address nonlinearity)
4. **Stationarity/Cointegration:** Assumes variables are either stationary or cointegrated
5. **No structural breaks:** Assumes relationships are stable over the sample period
6. **Lag selection:** Limited by MAX_LAG parameter - may miss longer-term effects

## Troubleshooting

**"Insufficient data" warnings:**
- Increase sample size or reduce TOP_N_VARIABLES

**VAR/VECM doesn't converge:**
- Reduce MAX_LAG
- Remove highly collinear variables
- Check for missing values or outliers

**NetLogo model behaves erratically:**
- Reduce `var-strength` and `interaction-strength` sliders
- Increase `error-correction-strength` to stabilize
- Check that input data was properly normalized

**No cointegration found:**
- This is normal - not all systems have long-run equilibria
- Model will use VAR dynamics only (still valid)

## Citation

If you use this pipeline in research, please cite:

```
Engle, R.F. and Granger, C.W.J. (1987). "Co-integration and Error Correction:
Representation, Estimation, and Testing". Econometrica, 55(2), 251-276.

Lütkepohl, H. (2005). "New Introduction to Multiple Time Series Analysis".
Springer-Verlag Berlin Heidelberg.
```

## License

This pipeline is provided as-is for educational and research purposes.

## Author

Auto-generated pipeline with explanatory documentation.
NetLogo code generation assisted by Claude AI (Anthropic).

---

**Version:** 1.0
**Last Updated:** 2025
**Compatible with:** NetLogo 6.4, Python 3.7+
